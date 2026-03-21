## 1. 概述

这段代码是一个基于 MySQL C API 开发的图片存储与管理程序。它通过命令行交互，支持在 MySQL 数据库中创建 `images` 表，插入用户信息（姓名、性别），以及插入和读取图片文件（BLOB 类型）。程序采用预处理语句（prepared statements）来防止 SQL 注入，并正确处理二进制数据。对于实习生来说，这是一个理解数据库编程、BLOB 数据处理、预处理语句使用以及 C 语言与 MySQL 交互的实用范例。

---

## 2. 头文件解释

```c
#include <stdio.h>      // 标准输入输出，如 printf、scanf
#include <stdlib.h>     // 内存分配（malloc、free）、程序退出等
#include <string.h>     // 字符串操作（strlen、strcmp）
#include <mysql/mysql.h> // MySQL C API 函数和类型
```

- **`stdio.h`**：用于控制台输入输出和文件操作（`fopen`、`fread`、`fwrite`）。
- **`stdlib.h`**：提供动态内存分配和程序退出功能。
- **`string.h`**：用于字符串长度计算、比较等。
- **`mysql/mysql.h`**：MySQL C 客户端库头文件，包含连接、语句执行、预处理语句等 API。

---

## 3. 核心数据结构

代码中主要使用了 MySQL C API 提供的结构体：

- **`MYSQL`**：表示一个 MySQL 连接句柄，通过 `mysql_init` 和 `mysql_real_connect` 获取。
- **`MYSQL_STMT`**：预处理语句句柄，通过 `mysql_stmt_init` 创建，用于执行参数化 SQL。
- **`MYSQL_BIND`**：绑定参数或结果的结构体，用于在 C 变量和 SQL 语句之间传递数据。
- **`MYSQL_RES`**：结果集元数据，用于获取字段信息。
- **`MYSQL_FIELD`**：字段信息结构体，包含字段名、类型、长度等。

---

## 4. 宏定义

本代码未定义自定义宏，直接使用常量字符串和数组。

---

## 5. 主要函数思路详解

### 5.1 `read_file`

```c
unsigned char* read_file(const char *filename, size_t *size);
```

**设计思路**：以二进制方式读取整个文件到内存，返回缓冲区指针，同时通过 `size` 参数返回文件大小。该函数用于准备插入到数据库的 BLOB 数据。

**实现步骤**：
1. 以 `"rb"` 模式打开文件，确保二进制读取。
2. 使用 `fseek` 和 `ftell` 获取文件大小。
3. 动态分配 `*size` 字节的缓冲区。
4. 读取整个文件内容到缓冲区。
5. 关闭文件，返回缓冲区指针（调用者负责释放）。

**关键点**：
- 必须检查 `fopen` 和 `malloc` 的返回值。
- 读取后需要 `fclose`，避免资源泄漏。

---

### 5.2 `insert_image`

```c
void insert_image(MYSQL *conn);
```

**设计思路**：通过预处理语句向 `images` 表插入一条记录，包含图片名称和图片二进制数据。

**实现步骤**：
1. 提示用户输入文件路径和图片名称。
2. 调用 `read_file` 读取图片数据到内存。
3. 初始化预处理语句：`mysql_stmt_init`。
4. 准备 SQL：`INSERT INTO images (name, image_data) VALUES (?, ?)`，调用 `mysql_stmt_prepare`。
5. 绑定参数：
   - 第一个参数为字符串类型（`MYSQL_TYPE_STRING`），指向图片名称，`buffer_length` 设为实际长度。
   - 第二个参数为 BLOB 类型（`MYSQL_TYPE_BLOB`），指向图片数据缓冲区，`buffer_length` 设为文件大小。
6. 执行语句：`mysql_stmt_execute`。
7. 打印影响行数。
8. 释放预处理语句和图片数据缓冲区。

**关键点**：
- 使用 `MYSQL_BIND` 结构体传递二进制数据。
- 执行后需要关闭语句（`mysql_stmt_close`）和释放文件数据（`free`）。

---

### 5.3 `read_image`

```c
void read_image(MYSQL *conn);
```

**设计思路**：根据用户输入的 ID 从数据库中查询图片的 BLOB 数据，并保存到本地文件。

**实现步骤**：
1. 提示用户输入图片 ID 和保存路径。
2. 准备查询 SQL：`SELECT image_data FROM images WHERE id = ?`。
3. 绑定输入参数（ID）为 `MYSQL_TYPE_LONG`。
4. 执行查询，调用 `mysql_stmt_store_result` 缓存结果集。
5. 检查是否有结果行（`mysql_stmt_num_rows`）。
6. 获取结果集元数据（`mysql_stmt_result_metadata`），用于后续绑定。
7. 绑定结果集：先绑定一个空缓冲区，`length` 指针指向 `data_len`。
8. 第一次 `mysql_stmt_fetch` 获取实际数据长度（`data_len` 被填充）。
9. 根据 `data_len` 分配缓冲区，重新绑定结果集，再次 `mysql_stmt_fetch` 获取数据。
10. 将数据写入本地文件（`fopen` + `fwrite`）。
11. 释放相关资源（元数据、结果集、语句、数据缓冲区）。

**关键点**：
- 需要先获取数据长度再分配内存，因为 BLOB 大小未知。
- `mysql_stmt_store_result` 必须调用，以便使用 `mysql_stmt_num_rows` 和多次 `mysql_stmt_fetch`。
- 注意资源释放顺序，避免内存泄漏。

---

### 5.4 `create_images_table`

```c
void create_images_table(MYSQL *conn);
```

**设计思路**：创建 `images` 表（如果不存在）。

**实现**：
- 执行 SQL：`CREATE TABLE IF NOT EXISTS images (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, image_data LONGBLOB NOT NULL, upload_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP)`。
- 使用 `mysql_query` 执行，失败时打印错误。

---

### 5.5 `insert_user`

```c
void insert_user(MYSQL *conn);
```

**设计思路**：向 `tbl_user` 表插入多条用户记录，使用预处理语句和循环输入，直到用户输入 `quit` 退出。

**实现步骤**：
1. 准备 SQL：`INSERT INTO tbl_user (U_NAME, U_GENDER) VALUES (?, ?)`。
2. 绑定两个字符串参数，并指定 `length` 指针以传递实际长度。
3. 循环提示用户输入姓名和性别，每次输入后执行语句。
4. 当输入 `quit` 时退出循环。
5. 关闭预处理语句。

**关键点**：
- 使用 `length` 指针是因为 `buffer` 是固定大小的数组，但实际输入长度可能小于缓冲区大小。
- 每次执行前更新 `name_len` 和 `gender_len`。

---

### 5.6 `main`

**设计思路**：建立数据库连接，显示菜单，根据用户选择调用相应功能函数。

**实现步骤**：
1. 初始化 MySQL 连接句柄（`mysql_init`）。
2. 连接数据库（`mysql_real_connect`），指定主机、用户名、密码、数据库名。
3. 设置字符集为 `utf8mb4`。
4. 调用 `create_images_table` 确保表存在。
5. 进入菜单循环，处理用户选择：
   - 1：调用 `insert_user`。
   - 2：调用 `insert_image`。
   - 3：调用 `read_image`。
   - 4：退出。
6. 关闭连接。

---

## 6. 完整代码注释（带详细解释）

```c
#include <stdio.h>      // 标准输入输出
#include <stdlib.h>     // 内存分配、退出
#include <string.h>     // 字符串操作
#include <mysql/mysql.h> // MySQL C API

/**
 * 以二进制方式读取整个文件到内存
 * @param filename 文件路径
 * @param size 输出参数，返回文件大小
 * @return 动态分配的缓冲区指针，失败返回 NULL，调用者需 free
 */
unsigned char* read_file(const char *filename, size_t *size) {
    FILE *fp = fopen(filename, "rb");        // 二进制读模式
    if (!fp) {
        perror("fopen failed");
        return NULL;
    }
    fseek(fp, 0, SEEK_END);                  // 定位到文件尾
    *size = ftell(fp);                       // 获取文件大小
    rewind(fp);                              // 回到文件头

    unsigned char *buffer = (unsigned char*)malloc(*size);
    if (!buffer) {
        fclose(fp);
        return NULL;
    }
    fread(buffer, 1, *size, fp);             // 读取全部内容
    fclose(fp);
    return buffer;
}

/**
 * 向 images 表插入一张图片
 * @param conn MySQL 连接句柄
 */
void insert_image(MYSQL *conn) {
    char filename[256];
    char img_name[256];
    size_t file_size;
    unsigned char *file_data;

    printf("请输入图片文件路径：");
    scanf("%s", filename);
    printf("请输入图片名称/描述：");
    scanf("%s", img_name);

    // 读取图片文件
    file_data = read_file(filename, &file_size);
    if (!file_data) {
        fprintf(stderr, "读取文件失败\n");
        return;
    }

    // 准备预处理语句
    const char *sql = "INSERT INTO images (name, image_data) VALUES (?, ?)";
    MYSQL_STMT *stmt = mysql_stmt_init(conn);
    if (!stmt) {
        fprintf(stderr, "mysql_stmt_init failed\n");
        free(file_data);
        return;
    }

    if (mysql_stmt_prepare(stmt, sql, strlen(sql))) {
        fprintf(stderr, "mysql_stmt_prepare failed: %s\n", mysql_stmt_error(stmt));
        mysql_stmt_close(stmt);
        free(file_data);
        return;
    }

    // 绑定参数
    MYSQL_BIND bind[2];
    memset(bind, 0, sizeof(bind));

    // 第一个参数：图片名称（字符串）
    bind[0].buffer_type = MYSQL_TYPE_STRING;
    bind[0].buffer = img_name;
    bind[0].buffer_length = strlen(img_name);
    bind[0].length = NULL;      // 字符串长度由 buffer_length 决定

    // 第二个参数：图片数据（二进制 BLOB）
    bind[1].buffer_type = MYSQL_TYPE_BLOB;
    bind[1].buffer = file_data;
    bind[1].buffer_length = file_size;
    bind[1].length = NULL;

    if (mysql_stmt_bind_param(stmt, bind)) {
        fprintf(stderr, "mysql_stmt_bind_param failed: %s\n", mysql_stmt_error(stmt));
        mysql_stmt_close(stmt);
        free(file_data);
        return;
    }

    // 执行插入
    if (mysql_stmt_execute(stmt)) {
        fprintf(stderr, "mysql_stmt_execute failed: %s\n", mysql_stmt_error(stmt));
    } else {
        printf("图片插入成功，影响行数：%llu\n", mysql_stmt_affected_rows(stmt));
    }

    mysql_stmt_close(stmt);
    free(file_data);
}

/**
 * 读取图片并保存到文件
 * @param conn MySQL 连接句柄
 */
void read_image(MYSQL *conn) {
    int id;
    char output_path[256];

    printf("请输入要读取的图片 ID：");
    scanf("%d", &id);
    printf("请输入保存路径：");
    scanf("%s", output_path);

    // 准备查询语句
    const char *sql = "SELECT image_data FROM images WHERE id = ?";
    MYSQL_STMT *stmt = mysql_stmt_init(conn);
    if (!stmt) {
        fprintf(stderr, "mysql_stmt_init failed\n");
        return;
    }

    if (mysql_stmt_prepare(stmt, sql, strlen(sql))) {
        fprintf(stderr, "mysql_stmt_prepare failed: %s\n", mysql_stmt_error(stmt));
        mysql_stmt_close(stmt);
        return;
    }

    // 绑定输入参数（ID）
    MYSQL_BIND in_bind;
    memset(&in_bind, 0, sizeof(in_bind));
    in_bind.buffer_type = MYSQL_TYPE_LONG;
    in_bind.buffer = &id;
    in_bind.buffer_length = sizeof(id);
    in_bind.length = NULL;

    if (mysql_stmt_bind_param(stmt, &in_bind)) {
        fprintf(stderr, "mysql_stmt_bind_param failed: %s\n", mysql_stmt_error(stmt));
        mysql_stmt_close(stmt);
        return;
    }

    // 执行查询
    if (mysql_stmt_execute(stmt)) {
        fprintf(stderr, "mysql_stmt_execute failed: %s\n", mysql_stmt_error(stmt));
        mysql_stmt_close(stmt);
        return;
    }

    // 存储结果集到客户端
    if (mysql_stmt_store_result(stmt)) {
        fprintf(stderr, "mysql_stmt_store_result failed: %s\n", mysql_stmt_error(stmt));
        mysql_stmt_close(stmt);
        return;
    }

    // 检查是否有结果行
    my_ulonglong row_count = mysql_stmt_num_rows(stmt);
    if (row_count == 0) {
        printf("未找到 ID 为 %d 的记录\n", id);
        mysql_stmt_free_result(stmt);
        mysql_stmt_close(stmt);
        return;
    }

    // 获取结果集元数据（用于后续绑定）
    MYSQL_RES *res_meta = mysql_stmt_result_metadata(stmt);
    if (!res_meta) {
        fprintf(stderr, "mysql_stmt_result_metadata failed: %s\n", mysql_stmt_error(stmt));
        mysql_stmt_free_result(stmt);
        mysql_stmt_close(stmt);
        return;
    }

    // 绑定结果集：先绑定空缓冲区以获取数据长度
    MYSQL_BIND result_bind;
    unsigned long data_len = 0;
    memset(&result_bind, 0, sizeof(result_bind));
    result_bind.buffer_type = MYSQL_TYPE_BLOB;
    result_bind.buffer = NULL;
    result_bind.buffer_length = 0;
    result_bind.length = &data_len;

    if (mysql_stmt_bind_result(stmt, &result_bind)) {
        fprintf(stderr, "mysql_stmt_bind_result failed: %s\n", mysql_stmt_error(stmt));
        mysql_free_result(res_meta);
        mysql_stmt_free_result(stmt);
        mysql_stmt_close(stmt);
        return;
    }

    // 第一次 fetch：获取数据长度
    int ret = mysql_stmt_fetch(stmt);
    if (ret == 0) {
        if (data_len > 0) {
            // 分配缓冲区
            unsigned char *data = (unsigned char*)malloc(data_len);
            if (!data) {
                fprintf(stderr, "内存分配失败\n");
                mysql_free_result(res_meta);
                mysql_stmt_free_result(stmt);
                mysql_stmt_close(stmt);
                return;
            }

            // 重新绑定结果集，指定缓冲区
            result_bind.buffer = data;
            result_bind.buffer_length = data_len;
            if (mysql_stmt_bind_result(stmt, &result_bind)) {
                fprintf(stderr, "mysql_stmt_bind_result failed\n");
                free(data);
                mysql_free_result(res_meta);
                mysql_stmt_free_result(stmt);
                mysql_stmt_close(stmt);
                return;
            }

            // 第二次 fetch：获取实际数据
            ret = mysql_stmt_fetch(stmt);
            if (ret == 0) {
                // 将数据写入文件
                FILE *fp = fopen(output_path, "wb");
                if (fp) {
                    fwrite(data, 1, data_len, fp);
                    fclose(fp);
                    printf("图片已保存到 %s，大小：%lu 字节\n", output_path, data_len);
                } else {
                    perror("fopen failed");
                }
            } else {
                fprintf(stderr, "mysql_stmt_fetch failed after binding buffer\n");
            }
            free(data);
        } else {
            printf("图片数据为空\n");
        }
    } else if (ret == MYSQL_NO_DATA) {
        printf("没有找到数据\n");
    } else {
        fprintf(stderr, "mysql_stmt_fetch error: %s\n", mysql_stmt_error(stmt));
    }

    // 释放资源
    mysql_free_result(res_meta);
    mysql_stmt_free_result(stmt);
    mysql_stmt_close(stmt);
}

/**
 * 创建 images 表（如果不存在）
 * @param conn MySQL 连接句柄
 */
void create_images_table(MYSQL *conn) {
    const char *sql = "CREATE TABLE IF NOT EXISTS images ("
                      "id INT AUTO_INCREMENT PRIMARY KEY,"
                      "name VARCHAR(255) NOT NULL,"
                      "image_data LONGBLOB NOT NULL,"
                      "upload_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP)";
    if (mysql_query(conn, sql)) {
        fprintf(stderr, "创建images表失败：%s\n", mysql_error(conn));
    } else {
        printf("images表已存在或创建成功\n");
    }
}

/**
 * 向 tbl_user 表插入多条用户记录（使用预处理语句循环输入）
 * @param conn MySQL 连接句柄
 */
void insert_user(MYSQL *conn) {
    char name[32];
    char gender[8];
    unsigned long name_len, gender_len;

    // 准备预处理语句
    const char *sql = "INSERT INTO tbl_user (U_NAME, U_GENDER) VALUES (?, ?)";
    MYSQL_STMT *stmt = mysql_stmt_init(conn);
    if (!stmt) {
        fprintf(stderr, "mysql_stmt_init failed\n");
        return;
    }

    if (mysql_stmt_prepare(stmt, sql, strlen(sql))) {
        fprintf(stderr, "mysql_stmt_prepare failed: %s\n", mysql_stmt_error(stmt));
        mysql_stmt_close(stmt);
        return;
    }

    // 绑定参数，注意使用 length 指针传递实际长度
    MYSQL_BIND bind[2];
    memset(bind, 0, sizeof(bind));

    bind[0].buffer_type = MYSQL_TYPE_STRING;
    bind[0].buffer = name;
    bind[0].buffer_length = sizeof(name);
    bind[0].length = &name_len;

    bind[1].buffer_type = MYSQL_TYPE_STRING;
    bind[1].buffer = gender;
    bind[1].buffer_length = sizeof(gender);
    bind[1].length = &gender_len;

    if (mysql_stmt_bind_param(stmt, bind)) {
        fprintf(stderr, "mysql_stmt_bind_param failed: %s\n", mysql_stmt_error(stmt));
        mysql_stmt_close(stmt);
        return;
    }

    while (1) {
        printf("\n请输入姓名（输入 'quit' 退出）：");
        scanf("%s", name);
        if (strcmp(name, "quit") == 0) break;

        printf("请输入性别（Male/Female）：");
        scanf("%s", gender);

        name_len = strlen(name);
        gender_len = strlen(gender);

        if (mysql_stmt_execute(stmt)) {
            fprintf(stderr, "mysql_stmt_execute failed: %s\n", mysql_stmt_error(stmt));
        } else {
            printf("插入成功，影响行数：%llu\n", mysql_stmt_affected_rows(stmt));
        }
    }

    mysql_stmt_close(stmt);
}

int main() {
    // 初始化 MySQL 连接句柄
    MYSQL *conn = mysql_init(NULL);
    if (!conn) {
        fprintf(stderr, "mysql_init failed\n");
        return EXIT_FAILURE;
    }

    // 连接到 MySQL 服务器（请根据实际情况修改密码和数据库名）
    if (!mysql_real_connect(conn, "localhost", "root", "123",
                            "testdb", 0, NULL, 0)) {
        fprintf(stderr, "连接失败：%s\n", mysql_error(conn));
        mysql_close(conn);
        return EXIT_FAILURE;
    }

    // 设置字符集为 utf8mb4，以支持中文等字符
    if (mysql_set_character_set(conn, "utf8mb4")) {
        fprintf(stderr, "设置字符集失败：%s\n", mysql_error(conn));
        mysql_close(conn);
        return EXIT_FAILURE;
    }
    printf("Connected to MySQL successfully!\n");

    // 确保 images 表存在
    create_images_table(conn);

    int choice;
    do {
        printf("\n===== 请选择操作 =====\n");
        printf("1. 插入用户数据（姓名、性别）\n");
        printf("2. 插入图片\n");
        printf("3. 读取图片\n");
        printf("4. 退出\n");
        printf("请输入选项（1-4）：");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                insert_user(conn);
                break;
            case 2:
                insert_image(conn);
                break;
            case 3:
                read_image(conn);
                break;
            case 4:
                printf("退出程序。\n");
                break;
            default:
                printf("无效选项，请重新输入。\n");
        }
    } while (choice != 4);

    mysql_close(conn);
    printf("Connection closed.\n");
    return 0;
}
```

---

## 7. 实习面试常见问题及解答思路

### Q1：为什么使用预处理语句（prepared statements）而不是直接拼接 SQL 字符串？

**答**：预处理语句有以下优点：
- **防止 SQL 注入**：参数与 SQL 语句分开，数据库引擎会对参数进行转义，避免恶意输入破坏 SQL 结构。
- **处理二进制数据**：BLOB 数据可能包含特殊字符（如 `\0`），直接拼接会导致截断或错误，预处理语句通过绑定缓冲区正确处理二进制数据。
- **性能提升**：对于重复执行的 SQL，预处理语句只需编译一次，多次执行可减少解析开销。

### Q2：在读取图片时，为什么需要先绑定空缓冲区获取长度，再分配内存？

**答**：因为 BLOB 字段的实际数据长度在查询前未知。先绑定空缓冲区执行 `mysql_stmt_fetch` 时，MySQL 会通过 `length` 参数返回实际数据长度。根据此长度动态分配足够的内存，然后重新绑定缓冲区再次 fetch 获取数据。这样可以避免因预先分配过大内存造成的浪费，或分配过小导致数据截断。

### Q3：代码中 `insert_user` 函数的参数绑定为什么使用了 `length` 指针，而 `insert_image` 没有？

**答**：在 `insert_user` 中，绑定的缓冲区是固定大小的数组（`name[32]` 和 `gender[8]`），但实际输入的字符串长度可能小于缓冲区大小。通过 `length` 指针传递实际长度，可以确保 MySQL 只写入有效数据，而不是整个缓冲区。在 `insert_image` 中，图片数据长度由 `file_size` 直接给出，且缓冲区大小就是实际数据大小，所以无需额外 `length`，直接使用 `buffer_length` 即可。

### Q4：`read_image` 函数中调用了 `mysql_stmt_store_result`，它的作用是什么？可以省略吗？

**答**：`mysql_stmt_store_result` 用于将结果集从服务器端缓存到客户端。它允许后续调用 `mysql_stmt_num_rows` 获取行数，并且在调用 `mysql_stmt_fetch` 时从本地缓存读取数据，减少与服务器的交互。如果省略，仍然可以逐行 fetch，但无法预先知道行数，且 fetch 时可能涉及多次网络通信。对于 BLOB 数据，通常需要缓存以方便多次获取（如先获取长度再获取数据）。

### Q5：如何处理可能的内存泄漏？

**答**：代码中每个动态分配的内存（`file_data`、`data`）都在使用后调用 `free` 释放。预处理语句、结果集元数据、结果集本身也在使用后调用相应的关闭/释放函数（`mysql_stmt_close`、`mysql_free_result`、`mysql_stmt_free_result`）。此外，主函数结束时关闭连接。确保每个资源都有对应的释放操作是防止内存泄漏的关键。

### Q6：如果图片文件非常大（例如超过内存可用大小），这段代码会有什么问题？如何改进？

**答**：`read_file` 将整个文件读入内存，对于超大文件可能导致内存耗尽。改进方法包括：
- 分块读取和插入：使用 `mysql_stmt_send_long_data` 分块发送数据。
- 使用 MySQL 的 `LOAD_FILE` 函数直接导入文件（需配置文件权限）。
- 避免一次性加载整个文件，而是通过多次读取和写入数据库。

### Q7：代码中 `read_image` 在绑定结果集时使用了两次 `mysql_stmt_fetch`，是否可以优化？

**答**：可以使用 `mysql_stmt_fetch_column` 直接获取指定列的数据，避免第二次绑定整个结果集。但当前方法（先获取长度再分配缓冲区后 fetch）也是常见做法，简单易懂。优化时还可以使用 `mysql_stmt_attr_set` 设置 `STMT_ATTR_UPDATE_MAX_LENGTH` 为 1，让 MySQL 自动分配足够大的缓冲区，但需注意内存管理。

### Q8：如何确保数据库表 `tbl_user` 存在？代码中直接插入，如果表不存在会怎样？

**答**：代码假设 `tbl_user` 已存在，未做检查。实际应用中应添加创建表的逻辑，或先查询表是否存在。示例中仅用于演示预处理语句的循环输入，生产环境需完善。

---

## 8. 实习建议

- **动手实践**：安装 MySQL 和 MySQL C 开发库（`libmysqlclient-dev`），编译运行代码（`gcc -o mysql_image mysql_image.c -lmysqlclient`），测试插入和读取图片功能。
- **深入学习**：
  - 阅读 MySQL C API 官方文档，理解 `MYSQL_STMT`、`MYSQL_BIND` 等结构体的详细用法。
  - 学习如何处理大文件的流式插入（`mysql_stmt_send_long_data`）。
  - 了解字符集设置对中文等非 ASCII 字符的影响。
- **扩展练习**：
  - 增加错误处理，确保每个步骤的返回值都得到检查。
  - 实现图片更新和删除功能。
  - 支持从数据库读取图片列表（显示 ID 和名称）。
  - 将图片数据以 Base64 编码形式输出，便于 Web 展示。
- **代码审查**：注意资源释放、缓冲区溢出（如 `scanf` 输入未限制长度），可改用 `fgets` 安全读取。
- **简历项目经验建议**：
  - **项目名称**：基于 MySQL C API 的图片存储管理系统
  - **项目描述**：使用 C 语言和 MySQL C API 开发一个命令行工具，支持将图片文件存入数据库（BLOB 类型），并根据 ID 读取图片保存到本地。核心采用预处理语句防止 SQL 注入，正确处理二进制数据，并通过动态内存管理确保大文件的正确读取。项目中还实现了用户信息的批量插入，展示了预处理语句在循环输入中的应用。此项目深入实践了数据库编程、二进制数据处理和资源管理。
  - **技术栈**：C 语言、MySQL C API、预处理语句、BLOB 数据类型、文件 I/O
  - **收获**：掌握了 MySQL 数据库的 C 语言接口使用方法，理解了预处理语句的优势和 BLOB 数据的处理技巧，学会了如何编写安全、高效的数据库操作代码。
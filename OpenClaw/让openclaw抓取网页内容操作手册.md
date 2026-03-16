# OpenClow 浏览器控制功能完全操作手册（通过 Chrome 扩展爬取网页）

本手册将引导你从零开始，让 OpenClaw 能够通过 Chrome 扩展控制你的本地浏览器，从而实现网页访问、数据抓取等功能。整个过程无需在服务器上运行无头浏览器，资源消耗低，配置简单。

---

## 📌 前提条件
- 你已经拥有一台运行 OpenClaw 的服务器（IP 地址 `47.253.154.240`），且 OpenClaw 网关服务正常运行。
- 你本地有一台电脑（Windows / macOS / Linux），安装有 Chrome 浏览器。
- 你能够通过 SSH 连接到服务器（知道 root 密码或使用密钥）。

---

## 第一步：获取网关 Token

OpenClaw 的浏览器扩展需要与网关进行身份验证，你需要先获取网关的 token。

1. **通过 SSH 登录服务器**（如果你已经在服务器终端，可直接执行）：
   ```bash
   ssh root@你的服务器IP
   ```
2. 运行以下命令查看 token：
   ```bash
   openclaw config get gateway.auth.token
   ```
   或者直接查看配置文件：
   ```bash
   cat ~/.openclaw/openclaw.json | grep token
   ```
   输出类似：
   ```
   "token": "83aadd7147403cda99bb906af40c9dd4fd03c5e84dbe1a8b"
   ```
3. **复制这个 token**（不要带引号），保存到记事本备用。

---

## 第二步：建立 SSH 隧道（关键步骤）

Chrome 扩展默认连接本地的 `127.0.0.1:18792`，但你的 OpenClaw 运行在远程服务器上。因此需要将服务器的 `18792` 端口通过 SSH 隧道转发到你的本地电脑。

### 2.1 在本地电脑上打开终端

- **Windows**：可以打开 PowerShell 或 CMD（命令提示符）。
- **macOS / Linux**：打开“终端”应用。

### 2.2 执行隧道命令

根据你的操作系统，输入以下命令（**注意替换 `你的服务器IP` 为实际 IP**）：

```bash
ssh -L 18792:127.0.0.1:18792 root@你的服务器IP -N
```

例如：
```bash
ssh -L 18792:127.0.0.1:18792 root@47.253.154.240 -N
```

**首次连接可能会提示确认主机密钥**，输入 `yes` 回车即可。之后会要求输入服务器密码（输入时不会显示，直接输完回车）。

**成功后终端会卡住**（没有新的输出），这是正常的！**这个窗口必须一直保持打开**，隧道才有效。不要关闭它，最小化即可。

---

## 第三步：安装并加载 Chrome 扩展

### 3.1 获取扩展文件

在服务器终端（另一个窗口）执行：
```bash
openclaw browser extension path
```
输出类似：
```
/root/.openclaw/browser/chrome-extension
```
这是扩展文件在服务器上的路径。

**你需要将这个文件夹复制到本地电脑**。可以使用 `scp` 命令（在本地终端新开一个窗口）：

```bash
scp -r root@你的服务器IP:/root/.openclaw/browser/chrome-extension 本地存放路径
```
例如（Windows 用户可以在 PowerShell 中执行）：
```bash
scp -r root@47.253.154.240:/root/.openclaw/browser/chrome-extension C:\Users\你的用户名\Desktop\openclaw-extension
```
如果使用 `scp` 不方便，也可以用 FTP 工具（如 FileZilla）下载。

### 3.2 在 Chrome 中加载扩展

1. 打开 Chrome 浏览器，在地址栏输入 `chrome://extensions/` 并回车。
2. **开启右上角的“开发者模式”**（开关点亮）。
3. 点击左上角 **“加载已解压的扩展程序”**。
4. 选择你刚才下载到本地的 `chrome-extension` 文件夹。
5. 加载成功后，工具栏会出现一个拼图形状的 OpenClaw 图标。点击拼图图标，找到 OpenClaw Browser Relay，点击**图钉图标**将其固定到工具栏。

---

## 第四步：配置扩展

1. **点击工具栏上的 OpenClaw 扩展图标**，会打开一个配置页面（类似下图）。
   ![配置页面示例](https://docs.openclaw.ai/assets/chrome-extension-config.png)
2. 在页面中：
   - **Port** 保持默认 `18792`。
   - **Gateway token** 输入框粘贴你第一步复制的 token。
3. 配置后稍等片刻，页面底部会显示 `Relay reachable and authenticated at http://127.0.0.1:18792/`，表示连接成功。

---

## 第五步：附加标签页

1. 在 Chrome 中打开你想要 AI 控制的网页（例如某个招聘网站）。
2. **点击工具栏上的 OpenClaw 扩展图标**。
   - 如果图标显示红色 `!`，说明隧道未建立或 token 错误，请检查。
   - 正常连接后，图标会短暂显示 `…`，然后变为 **`ON`**。
3. 图标显示 `ON` 表示当前标签页已附加，OpenClaw 现在可以控制这个页面了。

**注意**：目前建议一次只附加一个标签页，避免多个标签页同时附加时可能出现的冲突。

---

## 第六步：验证连接

在服务器终端（或新开一个 SSH 窗口）运行：
```bash
openclaw browser --browser-profile chrome tabs
```
如果看到你附加的标签页信息（如标题、URL），说明一切正常，现在你可以开始让 AI 操作浏览器了。

---

## 第七步：让 AI 控制浏览器抓取数据

### 7.1 基础命令示例

在 OpenClaw 的聊天界面（Web 或 Telegram 等），你可以直接输入以下命令（部分命令可能需要技能支持，但基础操作通常内置）：

- **获取当前页面快照（给元素自动编号）**：
  ```bash
  openclaw browser --browser-profile chrome snapshot
  ```
- **点击编号为 `e12` 的元素**：
  ```bash
  openclaw browser --browser-profile chrome click e12
  ```
- **在输入框 `e23` 中输入文本**：
  ```bash
  openclaw browser --browser-profile chrome type e23 "我要搜索的内容"
  ```
- **输入后按回车**：
  ```bash
  openclaw browser --browser-profile chrome type e23 "我要搜索的内容" --submit
  ```
- **截取当前页面截图**：
  ```bash
  openclaw browser --browser-profile chrome screenshot
  ```

### 7.2 实战：爬取招聘岗位信息

假设你想从某招聘网站抓取岗位列表：

1. **手动导航**：在附加的 Chrome 标签页中打开目标招聘页面。
2. **让 AI 分析页面结构**：
   > “请获取当前页面的快照。”
3. **提取信息**：
   > “根据快照，找到所有岗位名称、公司名称和薪资，整理成表格发给我。”
4. **翻页继续抓取**：
   > “点击下一页按钮，然后重复刚才的抓取操作。”

AI 会通过扩展控制你已打开的标签页，利用你已登录的会话状态，直接提取页面内容。

---

## ⚠️ 安全与注意事项

- **不要附加敏感页面**：如网银、邮箱、私人社交账号等，因为 AI 可以读取当前页面的所有内容。
- **隧道窗口必须保持打开**：一旦关闭，扩展就会断开连接，需要重新建立隧道。
- **重启电脑后**：需要重新运行 SSH 隧道命令。
- **专用 Chrome 配置文件**：建议为 OpenClaw 单独创建一个 Chrome 用户，与个人浏览数据隔离。
- **用完分离**：操作完成后，再次点击扩展图标即可分离（徽章消失），释放连接。

---

## ❓ 常见问题

### Q：扩展图标一直显示红色 `!`
- 检查 SSH 隧道是否正常运行（隧道窗口是否卡住）。
- 确认端口 `18792` 没有被其他程序占用。
- 检查 token 是否正确，可以在服务器上用 `openclaw config get gateway.auth.token` 重新确认。

### Q：执行 `openclaw browser --browser-profile chrome tabs` 返回空列表
- 确认 Chrome 中是否已附加标签页（图标显示 `ON`）。
- 如果已附加但仍无输出，尝试刷新标签页后重新附加。

### Q：AI 操作时提示“元素不存在”或“无法点击”
- 可能是页面结构变化，重新运行 `snapshot` 获取最新编号。
- 确保页面已完全加载，没有弹出遮挡层。

---

恭喜！你现在已经成功让 OpenClaw 具备了浏览器控制能力。无论是自动搜集招聘信息、定时访问网页，还是执行任何重复性操作，都可以通过 AI 指令轻松完成。

如果在操作中遇到其他问题，欢迎随时查阅官方文档 [docs.openclaw.ai/tools/chrome-extension](https://docs.openclaw.ai/tools/chrome-extension) 或联系技术支持。
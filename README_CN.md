# JSOS

> 一个基于浏览器的桌面环境。无后端、无数据库、无 Docker。打开 URL，你就拥有了一个桌面。

[English](README.md)

---

![JSOS 截图](screenshot.jpg)

---

## 这是什么？

JSOS（JavaScript OS）在浏览器中运行完整的桌面环境——窗口、任务栏、壁纸、应用管理、终端——全部由 [WebContainer](https://webcontainers.io)（浏览器内的 Node.js）驱动。你可以把它理解为一个轻量级的、可自托管的云桌面，整个系统就是一组静态文件。

应用以 ZIP 包形式分发，在 WebContainer 中作为独立 Node.js 进程运行。安装、运行、卸载——全在浏览器内完成。

---

## 部署

JSOS 是一组静态文件。部署到任何支持自定义 HTTP 头的平台即可。

### 1. Cloudflare Pages（推荐）

1. Fork 本仓库到你的 GitHub 账户。
2. 进入 [Cloudflare 控制台](https://dash.cloudflare.com/) → **Workers & Pages** → **创建** → **Pages**。
3. 连接你 Fork 的仓库。
4. 构建设置：
   - **构建命令：**（留空或填 `echo done`）
   - **构建输出目录：** `/`（根目录）
5. 部署。

`_headers` 文件会被 Cloudflare Pages 自动识别，无需额外配置。

### 2. 本地运行

```bash
npx serve .
```

浏览器打开 `http://localhost:3000`。

`serve.json` 会通过 `npx serve` 自动注入所需的 COEP/COOP 头，无需额外配置。

### 3. Nginx / 远程服务器

将所有文件上传到服务器，配置 Nginx：

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /path/to/dist;
    index index.html;

    # 自定义头（关键——见下方说明）
    add_header Cross-Origin-Embedder-Policy "require-corp" always;
    add_header Cross-Origin-Opener-Policy "same-origin" always;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静态资源缓存
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

> **警告：** 如果缺少上述两条 `add_header` 指令，**WebContainer 将无法启动。**浏览器会静默拒绝初始化 Web Worker，你会看到一个空白桌面，控制台没有任何报错。这是浏览器的同源隔离安全策略，不是 bug。

---

## 自定义头的重要性

WebContainer 需要**跨源隔离**才能运行。这是浏览器的安全特性，将页面与其浏览器上下文隔离，从而启用 `SharedArrayBuffer` 等 WebContainer 依赖的底层 API。

需要在**所有 HTML 响应**上设置两个头：

| 头字段 | 值 | 作用 |
|--------|------|------|
| `Cross-Origin-Embedder-Policy` | `require-corp` | 防止页面加载未经许可的跨域资源，确保所有资源在同一隔离上下文中加载。 |
| `Cross-Origin-Opener-Policy` | `same-origin` | 将浏览上下文与其他窗口/标签隔离，是 `SharedArrayBuffer` 支持的前提。 |

**各平台配置方式：**

| 平台 | 方式 |
|------|------|
| Cloudflare Pages | `public/_headers` 文件（自动识别） |
| Netlify | `public/_headers` 文件（自动识别） |
| Nginx | `add_header` 指令 |
| Apache | `.htaccess` 中使用 `Header set` |
| 其他静态托管 | 通过反向代理注入自定义头 |

---

## 下载

> **注意：** 目前仅开放预构建的部署包下载，源码正在整理与优化中。

将文件上传到任何静态托管平台，即可获得一个可用的 JSOS 实例。

---

## Star 解锁开源

本项目目前仅提供部署包。当仓库 star 数达到 **1,000** 时，完整源码将以开源协议发布。

如果你觉得 JSOS 有用，不妨点个 star——这能帮助更多人发现这个项目，也能让我们更快地走向开源。

---

## 反馈

如需提交 Bug 报告、功能建议、应用提交或一般讨论，请使用 [GitHub Issues](https://github.com/jsos-dev/jsos/issues)。

---

## 许可证

当前：**专有许可——仅限部署使用。**

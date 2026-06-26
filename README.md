# 🖥️ JSOS

> A browser-based Web desktop environment running Node.js/Python/Wasm. No backend. No database. No Docker. Just open the URL and you have a desktop.

🌐 **Official Website:** [https://jsos.dev](https://jsos.dev)

[中文文档](README_CN.md)

---

![JSOS Screenshot](screenshot.jpg)

---

## 📖 What is this?

JSOS (JavaScript OS) runs a full desktop environment in your browser — with windows, a taskbar, wallpapers, app management, and a terminal — all powered by [WebContainer](https://webcontainers.io) (in-browser Node.js/Python/Wasm). Think of it as a lightweight, self-hosted cloud desktop that lives in a single static folder.

✅ **Run most Node.js applications and tools** directly in your browser  
✅ **Develop your own applications** with a built-in terminal and code editor  
✅ **All data stored locally** in your browser — your privacy is protected  
✅ **Fork and deploy locally** to ensure complete control over your data and security

Apps are packaged as ZIP files and run as isolated processes inside WebContainer. Install, run, uninstall — all from the browser.

---

## 🚀 Deployment

JSOS is a set of static files. Deploy them anywhere that supports custom HTTP headers.

### 1. Cloudflare Pages (Recommended)

1. Fork this repository to your GitHub account.
2. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/) → **Workers & Pages** → **Create** → **Pages**.
3. Connect your forked repo.
4. Build settings:
   - **Build command:** (leave empty or `echo done`)
   - **Build output directory:** `/` (root)
5. Deploy.

The `_headers` file is automatically picked up by Cloudflare Pages. No manual configuration needed.

### 2. Local

```bash
npx serve .
```

Open `http://localhost:3000` in your browser.

The included `serve.json` automatically sets the required COEP/COOP headers via `npx serve`. No extra configuration needed.

### 3. Nginx / Remote Server

Copy all files to your server and configure Nginx:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /path/to/dist;
    index index.html;

    # Custom headers for WebContainer (CRITICAL — see below)
    add_header Cross-Origin-Embedder-Policy "require-corp" always;
    add_header Cross-Origin-Opener-Policy "same-origin" always;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

> **Warning:** Without the two `add_header` directives above, **WebContainer will not start.** The browser will silently refuse to initialize the Web Worker. You will see a blank desktop with no errors in the console. This is by design — browsers enforce Cross-Origin Isolation for security.

---

## ⚙️ Custom Headers — Why They Matter

WebContainer requires **Cross-Origin Isolation** to function. This is a browser security feature that isolates a page from its browser context, enabling `SharedArrayBuffer` and other low-level APIs that WebContainer depends on.

Two headers must be set on **every HTML response**:

| Header | Value | Purpose |
|--------|-------|---------|
| `Cross-Origin-Embedder-Policy` | `require-corp` | Prevents the page from loading cross-origin resources without explicit permission. Ensures all resources are loaded in the same isolated context. |
| `Cross-Origin-Opener-Policy` | `same-origin` | Isolates the browsing context from other windows/tabs. Required for `SharedArrayBuffer` support. |

**How each platform handles this:**

| Platform | Method |
|----------|--------|
| Cloudflare Pages | `public/_headers` file (automatic) |
| Netlify | `public/_headers` file (automatic) |
| Nginx | `add_header` directive |
| Apache | `.htaccess` with `Header set` |
| Any static host | Custom header injection via reverse proxy |

---

## ⭐ Star

If you like JSOS, please give us a ⭐! Your support helps us speed up source code organization and open-source plan progress.

---

## 💬 Feedback

For bug reports, feature requests, app submissions, or general discussion, please use [GitHub Issues](https://github.com/jsos-dev/jsos/issues).

---

## 📜 License

[MIT License](LICENSE)

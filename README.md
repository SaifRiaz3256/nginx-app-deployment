# Apache + Nginx + Cloudflare Tunnel Setup

This project documents how I configured **Nginx** alongside an existing **Apache** server, and exposed multiple websites through **Cloudflare Tunnel**. The goal was to keep Apache running on the main domain while serving additional websites on Nginx subdomains.

---

## 🚀 Project Overview

- **Apache (existing setup)**
  - `mytixe.me` → Apache site on port 80
  - `blog.mytixe.me` → Apache blog site on port 80

- **Nginx (new setup)**
  - `nginx1.mytixe.me` → served from `/var/www/nginx1` on port 8081
  - `nginx2.mytixe.me` → served from `/var/www/nginx2` on port 8082

- **Cloudflare Tunnel**
  - Used to expose all sites securely via HTTPS without opening ports to the internet.
  - Configured ingress rules to route each subdomain to the correct backend.

---

## ⚙️ Installation & Setup

### 1. Install Nginx
```bash
sudo apt update
sudo apt install nginx -y
```

### 2. Run Apache and Nginx side by side
- Apache kept ports **80/443**.
- Nginx configured on **8081** and **8082** to avoid conflicts.
- Disabled the default Nginx server on port 80 to prevent binding errors.

### 3. Create Document Roots
```bash
sudo mkdir -p /var/www/nginx1
sudo mkdir -p /var/www/nginx2
```

Add simple HTML pages in each folder for testing.

### 4. Configure Nginx Sites
**nginx1.mytixe.me.conf**
```nginx
server {
    listen 8081;
    server_name nginx1.mytixe.me;

    root /var/www/nginx1;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**nginx2.mytixe.me.conf**
```nginx
server {
    listen 8082;
    server_name nginx2.mytixe.me;

    root /var/www/nginx2;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable and reload:
```bash
sudo ln -s /etc/nginx/sites-available/nginx1.mytixe.me.conf /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/nginx2.mytixe.me.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 5. Configure Cloudflare Tunnel
`~/.cloudflared/config.yml`
```yaml
tunnel: <tunnel-id>
credentials-file: /etc/cloudflared/<tunnel-id>.json

ingress:
  - hostname: mytixe.me
    service: http://localhost:80
  - hostname: blog.mytixe.me
    service: http://localhost:80
  - hostname: nginx1.mytixe.me
    service: http://localhost:8081
  - hostname: nginx2.mytixe.me
    service: http://localhost:8082
  - service: http_status:404
```

Restart:
```bash
sudo systemctl restart cloudflared
```

Now all sites are served securely via Cloudflare.

---

## 🧪 Error Page Attempt
- Added a custom `404.html` in each document root.
- Configured:
  ```nginx
  error_page 404 /404.html;
  ```
- Locally worked via direct IP access, but Cloudflare proxy replaced error responses with its own page.
- Marked as **experimental / not fully functional with Cloudflare**.

---

## ✅ Final State
- Apache serves:
  - `mytixe.me`
  - `blog.mytixe.me`
- Nginx serves:
  - `nginx1.mytixe.me`
  - `nginx2.mytixe.me`
- Cloudflare Tunnel provides HTTPS for all domains.
- No reverse proxy used — Nginx and Apache run side by side on different ports.

---

## 📂 Repository Contents
```
nginx-apache-project/
│
├── nginx-config/        # Nginx main + site configs (dummy versions)
├── apache-config/       # Apache site configs (dummy versions)
├── cloudflare/          # Cloudflare tunnel config (dummy)
└── www/                 # Example website roots
```

---

## 📝 Notes
- Real credentials (Cloudflare JSON, database configs, etc.) have been removed.
- This repo only contains **dummy/sanitized configs** for demonstration.
- Actual deployment uses sensitive keys managed securely outside of Git.

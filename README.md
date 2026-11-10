# Factory-OEM Fast Website

A security-first, ultra-fast static-site CMS for foreign trade. One-command install, WordPress-style admin, zero database on the front end.

> **What makes it "Fast"?**
> - **Pure static HTML**: No database, no PHP on the front end. ~0.3s first paint globally.
> - **Hugo engine**: Compiles content in milliseconds. The admin panel writes Markdown, Hugo turns it into HTML.
> - **Atomic Rebuilds**: Content changes trigger a sandboxed, atomic rebuild. A failed build never breaks the live site.

> **What makes it "Safe"?**
> - **Hardened PHP**: No shell execution (`disable_functions`), tight `open_basedir`.
> - **Zero code injection**: Admins edit content and colors, never templates.
> - **Image sanitizer**: Uploaded images re-encoded server-side (GD) to strip EXIF & payloads.
> - **Audit logs**: Login rate-limited; all writes audited to `/var/log/fofw-audit.log`.
> - **Systemd sandbox**: Builds run with `ProtectSystem=strict`, `NoNewPrivileges`, `MemoryDenyWriteExecute`.

---

## 1. System Requirements

- **OS**: Debian 12 or 13 (fresh install recommended)
- **Access**: Root (`sudo`)
- **Network**: Domain A-record pointed to this server (required for automatic SSL)
- **Ports**: TCP 80 and 443 open

---

## 2. Installation

### One-command install

```bash
sudo bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"
```

The installer asks for: domain, company name, tagline, admin username, admin password (**at least 12 chars** with upper/lower/digit/symbol), contact info, and optional SSL. Takes 3-5 minutes.

### Non-interactive (CI / batch)

```bash
sudo DOMAIN=example.com \
  SITE_TITLE="My Company" \
  ADMIN_USER=admin \
  ADMIN_PASSWORD='StrongPass@2026' \
  AUTO_SSL=Y \
  bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"
```

---

## 3. After install

| URL | Purpose |
|---|---|
| `https://yourdomain.com/` | Public site |
| `https://yourdomain.com/publisher/` | Admin panel |

> **Important**: Session cookies require **HTTPS**. If you skipped SSL, configure it first, then log in.

---

## 4. Admin panel overview

| Menu | Purpose |
|---|---|
| **Dashboard** | Stats, build status, quick actions |
| **Products** | List / create / edit / delete product pages |
| **News** | List / create / edit / delete news articles |
| **Media Library** | Upload images (auto re-encoded), copy Markdown |
| **Themes** | Switch color palette (Amber / Ocean / Forest) |
| **Settings** | Site title, base URL, tagline, contact info |
| **Security** | Change password, view audit log |
| **Backups** | Create & download ZIP of site + config |

Content model:
- **Product** = title, slug, date, description, image, SKU, material, specification, MOQ, tags, Markdown body
- **Article** = title, slug, date, description, tags, Markdown body

**There is no online template editor** — by design. Editing templates in the browser is the #1 takeover vector in WordPress. To modify templates, SSH into the server and edit `/srv/fofw/site/layouts/`, then run `/usr/local/bin/fofw-build`.

---

## 5. Useful commands

```bash
# Manually rebuild
/usr/local/bin/fofw-build

# Manually create a backup (auto daily at 03:30)
/usr/local/bin/fofw-backup

# Watch build log
tail -f /var/log/fofw-build.log

# Watch audit log
tail -f /var/log/fofw-audit.log

# Build watcher status
systemctl status fofw-watch.path
```

---

## 6. Security model

| Layer | Control |
|---|---|
| Transport | TLS 1.2/1.3 (Let's Encrypt), HSTS, strict SNI |
| Session | HttpOnly, Secure, SameSite=Strict, UA-bound |
| Auth | Bcrypt, 5-attempt lockout per IP (15 min), strong password policy |
| CSRF | Per-session token on every write |
| SQL | None — no database |
| XSS | Markdown only, `goldmark unsafe=false` (raw HTML stripped) |
| Upload | MIME sniff + GD re-encode + random filename + PHP blocked at Nginx |
| PHP | `open_basedir` narrow, `disable_functions` aggressive, `expose_php=off` |
| Build | systemd sandbox (`ProtectSystem=strict`, `PrivateTmp=yes`, `MemoryDenyWriteExecute=yes`) |
| Logs | Weekly rotated, 12 weeks retained |
| Backups | Daily tar.gz at 03:30, encrypted with `age` if `/etc/fofw/backup.pub` exists |

**Backup encryption**: `/etc/fofw/backup.key` is generated at install. **Save it offline** — without it, encrypted backups cannot be restored.

---

## 7. Troubleshooting

**publisher.php download failed**
Ensure `https://factory-oem.com/Ai/web/publisher.txt` is reachable. Otherwise download manually and place at `/var/www/fofw/publisher/index.php`.

**Admin login fails (no error shown)**
Usually means SSL isn't configured (cookie is `Secure`-only). Run:
```bash
sudo certbot certonly --webroot -w /var/www/fofw -d yourdomain.com -d www.yourdomain.com
sudo nginx -t && sudo systemctl reload nginx
```

**Site doesn't update after saving**
```bash
tail -f /var/log/fofw-build.log
```
Look for errors. Failed builds never overwrite the live `public/` directory.

**Forgot admin password**
```bash
HASH=$(php -r 'echo password_hash("NewStrongPass@2026", PASSWORD_DEFAULT), "\n";')
sudo sed -i "s|'password_hash'.*|'password_hash' => '$HASH',|" /etc/fofw/config.php
```

**Port 80/443 already in use**
Stop the conflicting service (`apache2`, `caddy`, etc.) before installing.

**Restore from a backup**
```bash
sudo tar -xzf /var/backups/fofw/fofw-2026-01-01-033000.tar.gz -C /
sudo systemctl restart php8.4-fpm nginx fofw-watch.path
```
If encrypted: first `age -d -i /etc/fofw/backup.key -o plain.tar.gz encrypted.tar.gz.age`.

---

## 8. License

MIT.

---
---

# Factory-OEM Fast Website（中文版）

面向外贸场景的**安全优先静态网站系统**：极速页面、一行命令安装、WordPress 风格后台。

> **"Fast" 体现在哪？**
> - 前台是**纯静态 HTML** —— 无数据库、前台无 PHP、首次渲染约 0.3 秒
> - 内容由 **Hugo** 毫秒级编译；后台只写 Markdown，Hugo 负责转 HTML
> - 内容变更触发**沙箱内原子重建**，构建失败线上站点完全不变

> **"Safe" 体现在哪？**
> - PHP 不能执行系统命令（`disable_functions`），`open_basedir` 收窄到 5 个目录
> - **模板逻辑锁死** —— 管理员只能改内容与配色，无法注入代码
> - 上传图片**服务端 GD 重编码**，剥离 EXIF 与潜在恶意负载
> - 登录限流；所有写操作审计到 `/var/log/fofw-audit.log`
> - 构建在 systemd 硬化沙箱内运行

---

## 1. 环境要求

- **系统**：Debian 12 / 13（建议全新安装）
- **权限**：Root（`sudo`）
- **网络**：域名 A 记录已指向本机（申请 SSL 时需要）
- **端口**：80、443 开放

---

## 2. 一键安装

```bash
sudo bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"
```

安装脚本会依次询问：域名、公司名、标语、管理员用户名、管理员密码（**至少 12 位**，含大写、小写、数字、符号）、联系信息，以及是否申请 SSL。全新安装约 3–5 分钟。

### 非交互安装（CI / 批量）

```bash
sudo DOMAIN=example.com \
  SITE_TITLE="My Company" \
  ADMIN_USER=admin \
  ADMIN_PASSWORD='StrongPass@2026' \
  AUTO_SSL=Y \
  bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"
```

---

## 3. 安装后

| 地址 | 用途 |
|---|---|
| `https://你的域名/` | 网站前台 |
| `https://你的域名/publisher/` | 管理后台 |

> **重要**：会话 Cookie 强制 **HTTPS**。如果安装时跳过 SSL，请先配置 SSL 再登录后台。

---

## 4. 后台结构（参考 WordPress）

| 菜单 | 作用 |
|---|---|
| **Dashboard 仪表盘** | 统计、构建状态、快捷入口 |
| **Products 产品** | 列表 / 新建 / 编辑 / 删除产品页 |
| **News 新闻** | 列表 / 新建 / 编辑 / 删除文章 |
| **Media Library 媒体库** | 上传图片（服务端自动重编码），复制 Markdown |
| **Themes 主题** | 切换配色（Amber / Ocean / Forest） |
| **Settings 设置** | 站点名、Base URL、标语、联系方式 |
| **Security 安全** | 修改密码、查看审计日志 |
| **Backups 备份** | 生成并下载站点 + 配置的 ZIP 包 |

内容模型：
- **产品** = 标题、slug、日期、摘要、主图、SKU、材质、规格、MOQ、标签、Markdown 正文
- **文章** = 标题、slug、日期、摘要、标签、Markdown 正文

**不提供在线模板编辑** —— 这是有意的。在线改模板是 WordPress 站点被接管的第一大根源。如需修改模板，SSH 登录服务器编辑 `/srv/fofw/site/layouts/`，然后运行 `/usr/local/bin/fofw-build`。

---

## 5. 常用命令

```bash
# 手动触发重建
/usr/local/bin/fofw-build

# 手动创建备份（每日 03:30 自动执行）
/usr/local/bin/fofw-backup

# 查看构建日志
tail -f /var/log/fofw-build.log

# 查看审计日志
tail -f /var/log/fofw-audit.log

# 查看构建监听状态
systemctl status fofw-watch.path
```

---

## 6. 安全模型

| 层面 | 措施 |
|---|---|
| 传输 | TLS 1.2/1.3（Let's Encrypt）、HSTS、严格 SNI |
| 会话 | HttpOnly、Secure、SameSite=Strict、绑定 UA |
| 登录 | Bcrypt 哈希、同 IP 5 次失败锁 15 分钟、强密码策略 |
| CSRF | 每次写操作校验 per-session token |
| SQL | 无数据库 |
| XSS | 正文按 Markdown 处理，`goldmark unsafe=false`（原始 HTML 被剥离） |
| 文件上传 | MIME 嗅探 + GD 重编码 + 随机文件名 + Nginx 拦截 `.php` |
| PHP | `open_basedir` 收窄、`disable_functions` 严格、`expose_php=off` |
| 构建 | systemd 沙箱（`ProtectSystem=strict`、`PrivateTmp=yes`、`MemoryDenyWriteExecute=yes`） |
| 日志 | 每周轮转，保留 12 周 |
| 备份 | 每日 03:30 tar.gz，若存在公钥则用 `age` 加密 |

**备份加密密钥**：安装时生成 `/etc/fofw/backup.key`（age 密钥）。如启用加密，**请务必将此文件离线保存** —— 它是解密备份的唯一凭证。

---

## 7. 常见问题

**publisher.php 下载失败**
确认 `https://factory-oem.com/Ai/web/publisher.txt` 可访问；否则手动下载后放到 `/var/www/fofw/publisher/index.php`。

**后台登录失败（无提示）**
通常是 SSL 尚未配置（Cookie 只允许 Secure）。执行：
```bash
sudo certbot certonly --webroot -w /var/www/fofw -d 你的域名 -d www.你的域名
sudo nginx -t && sudo systemctl reload nginx
```

**保存内容后网站没变化**
```bash
tail -f /var/log/fofw-build.log
```
查看 Hugo 报错。构建失败时线上 `public/` 目录不会被覆盖。

**忘记后台密码**
```bash
HASH=$(php -r 'echo password_hash("NewStrongPass@2026", PASSWORD_DEFAULT), "\n";')
sudo sed -i "s|'password_hash'.*|'password_hash' => '$HASH',|" /etc/fofw/config.php
```

**80/443 端口被占用**
先停掉冲突服务（如 `apache2`、`caddy`）再运行安装脚本。

**从备份恢复**
```bash
sudo tar -xzf /var/backups/fofw/fofw-2026-01-01-033000.tar.gz -C /
sudo systemctl restart php8.4-fpm nginx fofw-watch.path
```
（若备份经 `age` 加密，先执行：`age -d -i /etc/fofw/backup.key -o plain.tar.gz encrypted.tar.gz.age`）

---

## 8. 许可

MIT. 我们还提供服务器租用，代运营的技术支持，咨询微信号1921753

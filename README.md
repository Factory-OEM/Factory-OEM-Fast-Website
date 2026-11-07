# Factory-OEM-Fast-Website
Factory-OEM Fast Website
# Factory-OEM Fast Website

A security-first, static-site CMS for foreign trade. Ultra-fast pages, one-command install, WordPress-style admin.

> **What makes it "Fast"?**
> - Front end is **pure static HTML** — no database, no PHP on the front, ~0.3s first paint.
> - **Hugo** compiles content in milliseconds; the admin panel writes Markdown, Hugo turns it into HTML.
> - Content changes trigger a **sandboxed, atomic rebuild**. Failed builds never break the live site.

> **What makes it "Safe"?**
> - PHP cannot run shell commands (`disable_functions`), cannot reach outside a tight `open_basedir`.
> - Template logic is **locked** — admins edit content and colors, never code.
> - Uploaded images are **re-encoded server-side** (GD) to strip metadata and payloads.
> - Login rate-limited; all writes audited to `/var/log/fofw-audit.log`.
> - Build runs in a hardened systemd sandbox (`ProtectSystem=strict`, `NoNewPrivileges`, `MemoryDenyWriteExecute`).

---

## 1. Installation

### One command

```bash
sudo bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"

The installer will ask for:

Domain (e.g. example.com)

Company name

Tagline (optional)

Admin username

Admin password (≥12 chars with upper, lower, digit, symbol)

Contact email / phone (optional)

Whether to request a Let's Encrypt certificate now

Takes about 3–5 minutes on a fresh Debian server.

Non-interactive (CI / batch)
bash
sudo DOMAIN=example.com \
  SITE_TITLE="My Company" \
  ADMIN_USER=admin \
  ADMIN_PASSWORD='StrongPass@2026' \
  AUTO_SSL=Y \
  bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"
Requirements
OS: Debian 12 or 13 (x86_64 / ARM64)

Ports: 80 and 443 free

DNS: point the domain A-record to this server (required for SSL)

2. After install
URL	Purpose
https://yourdomain.com/	Public site
https://yourdomain.com/publisher/	Admin panel
Important: Session cookies require HTTPS. If you skipped SSL during install, log in only after configuring SSL, or session validation will fail.

3. What's inside the admin
The panel follows the WordPress admin mental model:

Menu	What it does
Dashboard	Stats, build status, shortcuts
Products	List/create/edit/delete product pages
News	List/create/edit/delete news articles
Media Library	Upload images (auto re-encoded by server), copy Markdown
Themes	Switch color palette (Amber / Ocean / Forest)
Settings	Site title, base URL, tagline, contact info
Security	Change password, view audit log
Backups	Create & download a ZIP of site + config
Content model:

Product = title, slug, date, description, image, SKU, material, specification, MOQ, tags, Markdown body

Article = title, slug, date, description, tags, Markdown body

There is no online template editor — this is intentional. Editing templates via a web UI is the #1 root cause of site takeover in WordPress. If you need template changes, edit /srv/fofw/site/layouts/ via SSH, then run /usr/local/bin/fofw-build.

4. Useful commands
bash
# Manually trigger a rebuild
/usr/local/bin/fofw-build

# Manually create a backup (auto daily at 03:30)
/usr/local/bin/fofw-backup

# Watch build log
tail -f /var/log/fofw-build.log

# Watch audit log
tail -f /var/log/fofw-audit.log

# Check build watcher status
systemctl status fofw-watch.path
5. Security model
Layer	Control
Transport	Let's Encrypt TLS 1.2/1.3, HSTS, strict SNI
Session	HttpOnly, Secure, SameSite=Strict, UA-bound
Auth	Bcrypt hash, 5-attempt lockout per IP (15 min), strong password enforcement
CSRF	Per-session token on every write
SQL	None — no database
XSS	Content is Markdown, goldmark unsafe=false (raw HTML stripped)
File upload	MIME sniff + GD re-encode + random filename + .php blocked at Nginx
PHP	open_basedir narrow, disable_functions aggressive, allow_url_include=off, expose_php=off
Build	systemd sandbox (ProtectSystem=strict, PrivateTmp=yes, MemoryDenyWriteExecute=yes)
Logs	fofw-build.log, fofw-audit.log, fofw-php.log — logrotated weekly, 12 weeks retained
Backups	Daily tar.gz at 03:30, encrypted with age if key present (/etc/fofw/backup.pub)
Backup encryption key: the installer generates /etc/fofw/backup.key (age). If you enable encryption, save this key offline — it's the only way to decrypt your backups.

6. Troubleshooting
publisher.php download failed
Ensure https://factory-oem.com/Ai/web/publisher.php is reachable. Otherwise download it manually and place it at /var/www/fofw/publisher/index.php.

Admin login fails (no error)
This usually means SSL isn't configured yet (cookie is Secure only). Run:

bash
sudo certbot certonly --webroot -w /var/www/fofw -d yourdomain.com -d www.yourdomain.com
then reload Nginx and retry login.

Site doesn't update after saving

bash
tail -f /var/log/fofw-build.log
Look for Hugo errors. Failed builds never overwrite the live public/ directory.

Forgot admin password

bash
HASH=$(php -r 'echo password_hash("NewStrongPass@2026", PASSWORD_DEFAULT), "\n";')
sudo sed -i "s|'password_hash'.*|'password_hash' => '$HASH',|" /etc/fofw/config.php
Port 80/443 already in use
Stop the conflicting service first (apache2, caddy, etc.) before running the installer.

Restore from a backup

bash
sudo tar -xzf /var/backups/fofw/fofw-2026-01-01-033000.tar.gz -C /
sudo systemctl restart php8.4-fpm nginx fofw-watch.path
(If encrypted with age, first: age -d -i /etc/fofw/backup.key -o plain.tar.gz encrypted.tar.gz.age)

7. License
MIT. Source code and templates are yours — fork, host, resell freely.

Factory-OEM Fast Website（中文版）
面向外贸场景的安全优先静态网站系统：极速页面、一行命令安装、WordPress 风格后台。

"Fast" 体现在哪？

前台是纯静态 HTML —— 无数据库、前台无 PHP、首次渲染约 0.3 秒

内容由 Hugo 毫秒级编译；后台只写 Markdown，Hugo 负责转 HTML

内容变更触发沙箱内原子重建，构建失败线上站点完全不变

"Safe" 体现在哪？

PHP 不能执行系统命令（disable_functions），open_basedir 收窄到 6 个目录

模板逻辑锁死 —— 管理员只能改内容与配色，无法注入代码

上传图片服务端 GD 重编码，剥离 EXIF 与潜在恶意负载

登录限流；所有写操作审计到 /var/log/fofw-audit.log

构建在 systemd 硬化沙箱内运行（ProtectSystem=strict、NoNewPrivileges、MemoryDenyWriteExecute）

1. 一键安装
bash
sudo bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"
安装脚本会依次询问：

域名（如 example.com）

公司名称

首页标语（可选）

管理员用户名

管理员密码（≥12 位，含大写、小写、数字、符号）

联系邮箱 / 电话（可选）

是否现在申请 Let's Encrypt 证书

全新 Debian 服务器上约 3–5 分钟完成。

非交互安装（CI / 批量部署）
bash
sudo DOMAIN=example.com \
  SITE_TITLE="My Company" \
  ADMIN_USER=admin \
  ADMIN_PASSWORD='StrongPass@2026' \
  AUTO_SSL=Y \
  bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"
环境要求
系统： Debian 12 / 13（x86_64 / ARM64）

端口： 80、443 空闲

DNS： 域名 A 记录已指向本机（申请 SSL 时必须）

2. 安装完成
地址	用途
https://你的域名/	网站前台
https://你的域名/publisher/	管理后台
重要： 会话 Cookie 强制 HTTPS。如果安装时跳过 SSL，请先配置 SSL 再登录后台，否则登录会失败。

3. 后台结构（参考 WordPress）
菜单	作用
Dashboard 仪表盘	统计、构建状态、快捷入口
Products 产品	列表 / 新建 / 编辑 / 删除产品页
News 新闻	列表 / 新建 / 编辑 / 删除文章
Media Library 媒体库	上传图片（服务端自动重编码），复制 Markdown
Themes 主题	切换配色（Amber / Ocean / Forest）
Settings 设置	站点名、Base URL、标语、联系方式
Security 安全	修改密码、查看审计日志
Backups 备份	生成并下载站点 + 配置的 ZIP 包
内容模型：

产品 = 标题、slug、日期、摘要、主图、SKU、材质、规格、MOQ、标签、Markdown 正文

文章 = 标题、slug、日期、摘要、标签、Markdown 正文

不提供在线模板编辑——这是有意的。 在线改模板是 WordPress 站点被接管的第一大根源。如需修改模板，请 SSH 登录服务器编辑 /srv/fofw/site/layouts/ 下的文件，然后执行 /usr/local/bin/fofw-build。

4. 常用命令
bash
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
5. 安全模型
层面	措施
传输	Let's Encrypt TLS 1.2/1.3、HSTS、严格 SNI
会话	HttpOnly、Secure、SameSite=Strict、绑定 UA
登录	Bcrypt 哈希、同 IP 5 次失败锁 15 分钟、强密码复杂度
CSRF	每次写操作校验 per-session token
SQL	无数据库
XSS	正文按 Markdown 处理，goldmark unsafe=false（原始 HTML 被剥离）
文件上传	MIME 嗅探 + GD 重编码 + 随机文件名 + Nginx 拦截 .php
PHP	open_basedir 收窄、disable_functions 严格、allow_url_include=off、expose_php=off
构建	systemd 沙箱（ProtectSystem=strict、PrivateTmp=yes、MemoryDenyWriteExecute=yes）
日志	fofw-build.log / fofw-audit.log / fofw-php.log —— 每周轮转，保留 12 周
备份	每日 03:30 tar.gz，若存在密钥则用 age 加密（/etc/fofw/backup.pub）
备份加密密钥： 安装时会生成 /etc/fofw/backup.key（age 密钥）。如启用加密，请务必将此文件离线保存——它是解密备份的唯一凭证。

6. 常见问题
publisher.php 下载失败
确认 https://factory-oem.com/Ai/web/publisher.php 可访问；否则手动下载后放到 /var/www/fofw/publisher/index.php。

后台登录失败（无提示）
通常是 SSL 尚未配置（Cookie 只允许 Secure）。执行：

bash
sudo certbot certonly --webroot -w /var/www/fofw -d 你的域名 -d www.你的域名
然后重载 Nginx 再登录。

保存内容后网站没变化

bash
tail -f /var/log/fofw-build.log
查看 Hugo 报错。构建失败时线上 public/ 目录不会被覆盖。

忘记后台密码

bash
HASH=$(php -r 'echo password_hash("NewStrongPass@2026", PASSWORD_DEFAULT), "\n";')
sudo sed -i "s|'password_hash'.*|'password_hash' => '$HASH',|" /etc/fofw/config.php
80/443 端口被占用
先停掉冲突服务（如 apache2、caddy）再运行安装脚本。

从备份恢复

bash
sudo tar -xzf /var/backups/fofw/fofw-2026-01-01-033000.tar.gz -C /
sudo systemctl restart php8.4-fpm nginx fofw-watch.path
（若备份经 age 加密，先执行：age -d -i /etc/fofw/backup.key -o plain.tar.gz encrypted.tar.gz.age）

7. 许可
MIT。源码与模板归你所有——可自由 fork、托管、再销售。

text

---

## 六、关键差异速览

| 关注点 | 旧版 v0.3 | 新版 v1.0 |
|---|---|---|
| **安装入口** | 手动上传 3 个文件 | 一行命令 `curl` 自动下载 |
| **软件名** | StaticCMS | **Factory-OEM Fast Website** |
| **后台框架** | 自定义 | WordPress 心智模型（左侧菜单 / 列表 / 编辑器 / 主题） |
| **模板编辑** | 在线可改 | **移除**，改配色 + SSH 编辑 + 命令重建 |
| **PHP 沙箱** | `disable_functions` 常规 | 加 `curl_exec / set_time_limit / ini_set / symlink / posix_*` 等 |
| **构建沙箱** | 无 | systemd `ProtectSystem=strict`、`MemoryDenyWriteExecute`、`RestrictAddressFamilies` |
| **上传安全** | 只校验 MIME | **GD 重编码剥离负载** + GIF 拒收 |
| **登录安全** | 无 | 限流 + 强密码 + 失败日志 |
| **审计** | 无 | `/var/log/fofw-audit.log` |
| **Session** | 无 UA 绑定 | **UA-hash 绑定**（防 cookie 窃取） |
| **备份加密** | 无 | `age` 公钥加密 + 密钥文件 |
| **日志轮转** | 无 | `/etc/logrotate.d/fofw` |
| **WAF 头** | 基本 | CSP、X-Frame DENY、`request_method` 白名单 |
| **恢复流程** | 无 | README 明确 tar 恢复 + age 解密步骤 |

**上传清单（三件）：**
1. `install.sh` → `https://factory-oem.com/Ai/web/install.sh`
2. `publisher.php` → `https://factory-oem.com/Ai/web/publisher.php`
3. `README.md` → GitHub 仓库根目录

用户端只需一条命令：`sudo bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"`

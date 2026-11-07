# Factory-OEM Fast Website

A security-first, ultra-fast static-site CMS for foreign trade. One-command install, WordPress-style admin, zero database on the front end.

> **What makes it "Fast"?**
> - **Pure static HTML**: No database, no PHP on the front end. ~0.3s first paint globally.
> - **Hugo engine**: Compiles your content in milliseconds. The admin panel writes Markdown, and Hugo turns it into HTML.
> - **Atomic Rebuilds**: Content changes trigger a sandboxed, atomic rebuild. A failed build will *never* break your live site.

> **What makes it "Safe"?**
> - **Hardened PHP**: PHP cannot run shell commands (`disable_functions`), and cannot reach outside a tight `open_basedir`.
> - **Zero Code Injection**: Template logic is completely locked — admins edit content and colors, never code.
> - **Image Sanitizer**: Uploaded images are re-encoded server-side (GD) to strip EXIF metadata and malicious payloads.
> - **Audit Logs**: Login rate-limited; all writes are audited to `/var/log/fofw-audit.log`.
> - **Systemd Sandbox**: Build runs in a hardened sandbox (`ProtectSystem=strict`, `NoNewPrivileges`, `MemoryDenyWriteExecute`).

---

## 1. System Requirements

Before you begin, ensure your server meets the following requirements:
*   **OS**: **Debian 13** (Fresh install recommended). *We do not support Ubuntu or Debian 11 yet.*
*   **Access**: Root access (`sudo`).
*   **Network**: A domain name pointed to your server's IP (for automatic SSL).
*   **Ports**: TCP 80 and 443 must be open in your firewall/security group.

---

## 2. Installation

### One-Command Install (Interactive)
This is the easiest way. The script will automatically install Nginx, PHP 8.4, Hugo, and configure SSL for you.

```bash
sudo bash -c "$(curl -fsSL https://factory-oem.com/Ai/web/install.sh)"

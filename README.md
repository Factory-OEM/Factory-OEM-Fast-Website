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

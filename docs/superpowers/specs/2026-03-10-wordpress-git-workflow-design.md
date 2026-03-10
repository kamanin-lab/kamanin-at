# WordPress Git Workflow — Design Spec

**Date:** 2026-03-10
**Project:** kamanin-at (kamanin-lab/kamanin-at)
**Status:** Approved

---

## Overview

Set up a Git-based development workflow for a WordPress site hosted on shared hosting. The workflow enables local development via Local by Flywheel with automatic deployment to production on every push to `main`.

---

## Context

- **Local environment:** Local by Flywheel at `C:/Users/upan/Local Sites/kamanin/`
- **Theme:** Blocksy (parent) + `blocksy-child` (child theme — tracked in Git)
- **Plugins:** ACF, Greenshift, Blocksy Companion Pro (not tracked); custom plugins (tracked when created)
- **GitHub repo:** `kamanin-lab/kamanin-at`
- **Hosting:** Shared hosting with SSH access
- **Deployment trigger:** Push to `main` branch

---

## Repository Structure

```
kamanin-at/
├── .github/
│   └── workflows/
│       └── deploy.yml        # GitHub Actions pipeline
├── themes/
│   └── blocksy-child/        # child theme files
│       ├── style.css
│       ├── functions.php
│       └── screenshot.jpg
├── plugins/                  # custom plugins (when created)
├── .gitignore
└── README.md
```

### What is NOT tracked:
- `uploads/` — media files (large, change on server)
- `blocksy/` — parent theme (third-party, updated via WP Admin)
- Third-party plugins (updated via WP Admin)

---

## Deployment Architecture

```
Push to main
    ↓
GitHub Actions runner
    ↓
rsync over SSH → server:/public_html/wp-content/
    ├── themes/blocksy-child/  →  themes/blocksy-child/
    └── plugins/custom-*/     →  plugins/custom-*/
    ↓
Deploy complete (~10-30 sec)
```

### GitHub Secrets (configured once in repo settings):

| Secret | Description |
|--------|-------------|
| `SSH_HOST` | Server hostname/IP |
| `SSH_USER` | SSH username |
| `SSH_PRIVATE_KEY` | Private SSH key (public key added to server) |
| `SSH_PORT` | SSH port (default: 22) |
| `REMOTE_PATH` | Path on server to `wp-content/` |

### rsync behavior:
- Syncs only changed files (fast, efficient)
- Deletes files removed from repo (`--delete` flag, scoped per directory)
- Does not touch `uploads/` or other directories outside scope

---

## Local Development Workflow

### Directory mapping:
```
Local Sites/kamanin/app/public/wp-content/
├── themes/
│   ├── blocksy/           ← NOT in git
│   └── blocksy-child/     ← git tracked
├── plugins/
│   ├── acf/               ← NOT in git
│   └── my-custom-plugin/  ← git tracked (when created)
└── uploads/               ← NOT in git
```

The repo is cloned such that `themes/` and `plugins/` directories map directly to `wp-content/themes/` and `wp-content/plugins/`.

### Daily workflow:
```
1. Edit theme/plugin files locally
2. git add . && git commit -m "description"
3. git push origin main
4. → GitHub Actions deploys to server automatically
```

### Branch strategy:
- `feature/*` branches — no deployment triggered
- Merge to `main` — deployment triggers automatically

---

## Decisions Made

1. **Track only child theme + custom plugins** — third-party code managed via WP Admin
2. **GitHub Actions + rsync over SSH** — reliable, fast, no git required on server
3. **Push to `main` = deploy** — simple single-developer workflow, no staging branch needed

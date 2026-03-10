# WordPress Git Workflow Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Set up a Git-based WordPress development workflow with automatic deployment to shared hosting via GitHub Actions + rsync over SSH.

**Architecture:** The repo (`kamanin-lab/kamanin-at`) tracks only `blocksy-child` theme and future custom plugins. Files are organized as `themes/` and `plugins/` directories mirroring `wp-content/`. On push to `main`, GitHub Actions connects to the server via SSH and rsyncs changed files directly into the server's `wp-content/`.

**Tech Stack:** Git, GitHub Actions, rsync, SSH, WordPress child theme (PHP/CSS)

**Spec:** `docs/superpowers/specs/2026-03-10-wordpress-git-workflow-design.md`

---

## Chunk 1: Repository Initialization

### Task 1: Initialize Git repo and folder structure

**Files:**
- Create: `.gitignore`
- Create: `README.md`
- Create: `themes/blocksy-child/.gitkeep` (placeholder until files copied)
- Create: `plugins/.gitkeep`

- [ ] **Step 1: Initialize git in the project directory**

```bash
cd "g:/01_OPUS/Projects/kamanin-at"
git init
git remote add origin https://github.com/kamanin-lab/kamanin-at.git
```

Expected: `Initialized empty Git repository in ...`

- [ ] **Step 2: Create folder structure**

```bash
mkdir -p themes/blocksy-child plugins
touch plugins/.gitkeep
```

- [ ] **Step 3: Create .gitignore**

Create file `g:/01_OPUS/Projects/kamanin-at/.gitignore` with this content:

```gitignore
# WordPress uploads and generated files
uploads/
*.log

# OS files
.DS_Store
Thumbs.db
desktop.ini

# Editor files
.vscode/
.idea/
*.swp
*.swo

# Node / build tools (if added later)
node_modules/
dist/
*.map

# Environment files
.env
.env.*

# Local by Flywheel
*.sql

# Keep plugin dir tracked even if empty
!plugins/.gitkeep
```

- [ ] **Step 4: Create README.md**

Create file `g:/01_OPUS/Projects/kamanin-at/README.md` with this content:

```markdown
# kamanin.at — WordPress Theme

Development workflow for kamanin.at WordPress site.

## What's tracked

- `themes/blocksy-child/` — child theme (Blocksy)
- `plugins/` — custom plugins (when created)

## What's NOT tracked

- Parent theme `blocksy/` — updated via WP Admin
- Third-party plugins — updated via WP Admin
- `uploads/` — media files

## Local development

This repo maps to `wp-content/` on the server.
Clone or symlink so that:
- `themes/` → `wp-content/themes/`
- `plugins/` → `wp-content/plugins/`

## Deployment

Push to `main` → GitHub Actions → rsync to server automatically.
```

- [ ] **Step 5: Verify structure**

```bash
ls -la "g:/01_OPUS/Projects/kamanin-at/"
```

Expected output includes: `.gitignore`, `README.md`, `themes/`, `plugins/`, `docs/`
Note: `.github/` is created in Chunk 2.

---

### Task 2: Copy child theme files into repo

**Files:**
- Copy from: `C:/Users/upan/Local Sites/kamanin/app/public/wp-content/themes/blocksy-child/`
- Copy to: `themes/blocksy-child/`

- [ ] **Step 1: Copy child theme files**

```bash
cp -r "C:/Users/upan/Local Sites/kamanin/app/public/wp-content/themes/blocksy-child/." \
  "g:/01_OPUS/Projects/kamanin-at/themes/blocksy-child/"
```

- [ ] **Step 2: Verify files were copied**

```bash
ls "g:/01_OPUS/Projects/kamanin-at/themes/blocksy-child/"
```

Expected: `style.css  functions.php  screenshot.jpg`

- [ ] **Step 3: Remove the placeholder .gitkeep if it exists**

```bash
rm -f "g:/01_OPUS/Projects/kamanin-at/themes/blocksy-child/.gitkeep"
```

---

### Task 3: Initial commit and push

- [ ] **Step 1: Stage all files**

```bash
cd "g:/01_OPUS/Projects/kamanin-at"
git add .gitignore README.md themes/ plugins/ docs/
```

- [ ] **Step 2: Verify what's staged**

```bash
git status
```

Expected: files listed as `new file:` — no unexpected files (no uploads, no parent theme)

- [ ] **Step 3: Initial commit**

```bash
git commit -m "feat: initial WordPress child theme and git workflow setup"
```

- [ ] **Step 4: Push to GitHub**

```bash
git branch -M main
git push -u origin main
```

Expected: `Branch 'main' set up to track remote branch 'main' from 'origin'.`

- [ ] **Step 5: Verify on GitHub**

Open https://github.com/kamanin-lab/kamanin-at in browser and confirm files are visible.

---

## Chunk 2: GitHub Actions Deployment Pipeline

### Task 4: Create deploy.yml workflow

**Files:**
- Create: `.github/workflows/deploy.yml`

- [ ] **Step 1: Create the workflows directory**

```bash
mkdir -p "g:/01_OPUS/Projects/kamanin-at/.github/workflows"
```

- [ ] **Step 2: Create deploy.yml**

Create file `g:/01_OPUS/Projects/kamanin-at/.github/workflows/deploy.yml` with this content:

```yaml
name: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -p ${{ secrets.SSH_PORT }} -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts

      - name: Deploy themes via rsync
        run: |
          rsync -avz --delete \
            -e "ssh -i ~/.ssh/deploy_key -p ${{ secrets.SSH_PORT }}" \
            themes/blocksy-child/ \
            ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:${{ secrets.REMOTE_PATH }}/themes/blocksy-child/

      - name: Deploy custom plugins via rsync
        run: |
          # Only sync if custom plugins exist (exclude .gitkeep)
          if [ "$(ls -A plugins/ | grep -v .gitkeep)" ]; then
            rsync -avz --delete \
              -e "ssh -i ~/.ssh/deploy_key -p ${{ secrets.SSH_PORT }}" \
              plugins/ \
              ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:${{ secrets.REMOTE_PATH }}/plugins/ \
              --exclude='.gitkeep'
          else
            echo "No custom plugins to deploy — skipping"
          fi
```

- [ ] **Step 3: Commit the workflow**

```bash
cd "g:/01_OPUS/Projects/kamanin-at"
git add .github/workflows/deploy.yml
git commit -m "ci: add GitHub Actions deploy workflow via rsync over SSH"
git push origin main
```

Expected: commit pushed, GitHub Actions tab shows a new workflow run (it will fail until secrets are set — that's expected)

---

## Chunk 3: SSH Key Setup and GitHub Secrets

### Task 5: Generate SSH key pair

This is done on your local machine (Windows).

- [ ] **Step 1: Generate SSH key pair**

Run in Git Bash or terminal:

```bash
ssh-keygen -t ed25519 -C "github-actions-kamanin-at" -f ~/.ssh/kamanin_deploy -N ""
```

Expected: two files created:
- `~/.ssh/kamanin_deploy` — private key
- `~/.ssh/kamanin_deploy.pub` — public key

- [ ] **Step 2: View the public key**

```bash
cat ~/.ssh/kamanin_deploy.pub
```

Copy the output — you'll add this to your server.

- [ ] **Step 3: View the private key**

```bash
cat ~/.ssh/kamanin_deploy
```

Copy the output (entire content including `-----BEGIN...` and `-----END...` lines) — you'll add this to GitHub Secrets.

---

### Task 6: Add public key to server

**Done on your hosting control panel (cPanel / ISPmanager / SSH).**

- [ ] **Step 1: SSH into your server with your current credentials**

```bash
ssh your_user@your_server_host
```

- [ ] **Step 2: Add the public key to authorized_keys**

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Replace `PASTE_PUBLIC_KEY_HERE` with the content from `kamanin_deploy.pub`.

- [ ] **Step 3: Verify SSH access with the new key**

Run from your local machine:

```bash
ssh -i ~/.ssh/kamanin_deploy -p YOUR_SSH_PORT your_user@your_server_host "echo SSH_OK"
```

Expected output: `SSH_OK`

If it fails — check that `authorized_keys` has the correct key and permissions are `600`.

---

### Task 7: Add secrets to GitHub repository

**Done at: https://github.com/kamanin-lab/kamanin-at/settings/secrets/actions**

- [ ] **Step 1: Add SSH_HOST**

Name: `SSH_HOST`
Value: your server's hostname or IP (e.g. `server123.hostinger.com`)

- [ ] **Step 2: Add SSH_USER**

Name: `SSH_USER`
Value: your SSH username

- [ ] **Step 3: Add SSH_PORT**

Name: `SSH_PORT`
Value: `22` (or whatever port your host uses — check in cPanel/ISPmanager)

- [ ] **Step 4: Add SSH_PRIVATE_KEY**

Name: `SSH_PRIVATE_KEY`
Value: entire content of `~/.ssh/kamanin_deploy` (private key, including header/footer lines)

- [ ] **Step 5: Add REMOTE_PATH**

Name: `REMOTE_PATH`
Value: absolute path to `wp-content/` on your server

Example: `/home/your_user/public_html/wp-content`

To find it, SSH into your server and run:
```bash
find / -name "wp-content" -type d 2>/dev/null | head -5
```

---

## Chunk 4: Local Development Link and Verification

### Task 8: Link local development to the repo

The goal: edits to files in the repo are immediately reflected in Local by Flywheel (no manual copying).

**Option A — Direct (simplest): copy workflow**

For the minimal `blocksy-child` theme (3 files), the simplest approach is to keep the repo as your working copy and manually copy to Local when needed. However, this creates friction.

**Option B — Symlinks (recommended for active development)**

Replace the child theme folder in Local Sites with a symlink to the repo folder.

- [ ] **Step 1: Backup the existing child theme in Local Sites (safety)**

```bash
cp -r "C:/Users/upan/Local Sites/kamanin/app/public/wp-content/themes/blocksy-child" \
  "C:/Users/upan/Local Sites/kamanin/app/public/wp-content/themes/blocksy-child-backup"
```

- [ ] **Step 2: Remove the existing child theme folder**

```bash
rm -rf "C:/Users/upan/Local Sites/kamanin/app/public/wp-content/themes/blocksy-child"
```

- [ ] **Step 3: Create symlink (run as Administrator in PowerShell)**

```powershell
New-Item -ItemType SymbolicLink `
  -Path "C:\Users\upan\Local Sites\kamanin\app\public\wp-content\themes\blocksy-child" `
  -Target "G:\01_OPUS\Projects\kamanin-at\themes\blocksy-child"
```

- [ ] **Step 4: Verify the symlink works**

Open Local by Flywheel, start the site, open WP Admin → Appearance → Themes.
The Blocksy Child theme should still be active.

- [ ] **Step 5: Clean up backup if everything works**

```bash
rm -rf "C:/Users/upan/Local Sites/kamanin/app/public/wp-content/themes/blocksy-child-backup"
```

---

### Task 9: End-to-end deployment test

- [ ] **Step 1: Make a visible test change to the child theme**

Edit `g:/01_OPUS/Projects/kamanin-at/themes/blocksy-child/style.css`:

Add a comment at the top:

```css
/* Deployed via GitHub Actions — test */
```

- [ ] **Step 2: Commit and push**

```bash
cd "g:/01_OPUS/Projects/kamanin-at"
git add themes/blocksy-child/style.css
git commit -m "test: verify deployment pipeline"
git push origin main
```

- [ ] **Step 3: Watch GitHub Actions run**

Go to https://github.com/kamanin-lab/kamanin-at/actions

Wait for the workflow to complete (~30-60 seconds).
Expected: green checkmark ✅

- [ ] **Step 4: Verify on production server**

SSH into server and check the file:

```bash
cat /path/to/wp-content/themes/blocksy-child/style.css | head -5
```

Expected: the test comment is visible.

- [ ] **Step 5: Remove test comment and deploy clean version**

```bash
# Remove the test comment from style.css
cd "g:/01_OPUS/Projects/kamanin-at"
git add themes/blocksy-child/style.css
git commit -m "chore: remove deployment test comment"
git push origin main
```

---

## What to Send / Gather Before Starting

Before executing, you need the following from your hosting provider:

| Info needed | Where to find it |
|-------------|-----------------|
| Server hostname/IP | Hosting control panel → SSH settings |
| SSH username | Hosting control panel → SSH settings |
| SSH port | Hosting control panel (usually 22) |
| Path to `wp-content/` on server | SSH into server, `find / -name "wp-content"` |

---

## Daily Workflow After Setup

```bash
# 1. Edit files in g:/01_OPUS/Projects/kamanin-at/themes/blocksy-child/
# 2. Preview changes in Local by Flywheel (via symlink — instant)
# 3. Commit and push:
git add .
git commit -m "feat: describe what you changed"
git push origin main
# 4. GitHub Actions deploys to production automatically (~30 sec)
```

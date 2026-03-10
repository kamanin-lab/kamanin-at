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

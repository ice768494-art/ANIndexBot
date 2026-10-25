# AnimeNetworkTamilIndexBot — Voroa Deployment

This project runs as a Python **Web Service** using Flask, Gunicorn, and a Telegram webhook. Do not deploy it as a background worker.

## Build and start commands

- Build command: `pip install -r requirements.txt`
- Start command: `gunicorn --bind 0.0.0.0:$PORT main:app`

## Required environment variables

- `BOT_TOKEN`: Telegram bot token from BotFather
- `ADMIN_IDS`: comma-separated Telegram user IDs allowed to administer the bot
- `WEBHOOK_URL`: the public HTTPS base URL assigned to your Voroa service (no trailing slash)
- `WEBHOOK_SECRET`: a random secret string for validating Telegram webhook requests
- `DEFAULT_EXPIRY_MINUTES`: default invite-link expiry in minutes (default `10`)
- `DB_PATH`: SQLite database path; use a path on persistent storage if Voroa offers it, e.g. `/var/data/anime.db`

Set `WEBHOOK_URL` to the service's actual public URL after creating the service. The app configures Telegram's webhook during startup, so a wrong URL will prevent correct operation.

## Bot commands

- `/start` — start the bot
- `/help` — show help
- `/list` — list configured anime (admin)
- `/add Anime Name | @YourChannel` — add an anime/channel (admin)
- `/remove Anime Name` — remove an anime (admin)
- `/setexpiry 10` — set default link expiry in minutes (admin)
- `/setphoto Anime Name` — reply to a photo to set its poster (admin)

## Telegram setup

Make the bot an administrator in each destination channel and grant permission to manage invite links. Keep `BOT_TOKEN` and `WEBHOOK_SECRET` private.

The `/` route returns a JSON health response when the web service is running. SQLite data will survive restarts only if `DB_PATH` points to persistent storage provided by your hosting plan.

# AnimeNetworkTamilIndexBot — Voroa Deployment

This project runs as a Python **Web Service** using Flask, Gunicorn, and a Telegram webhook. Do not deploy it as a background worker.

## Build and start commands

- Build command: `pip install -r requirements.txt`
- Start command: `gunicorn --bind 0.0.0.0:$PORT main:app`

## Environment variables

- `BOT_TOKEN`: Telegram bot token from BotFather
- `ADMIN_IDS`: comma-separated Telegram user IDs allowed to administer the bot
- `WEBHOOK_URL`: public HTTPS base URL assigned by Voroa, without a trailing slash
- `WEBHOOK_SECRET`: random secret string for validating Telegram webhook requests
- `DEFAULT_EXPIRY_MINUTES`: default private-channel invite-link expiry (default `10`)
- `DB_PATH`: SQLite database path; it must point to persistent storage if the hosting platform supports it
- `BACKUP_DIR`: directory for automatic SQLite backups (default: a `backups` folder beside `DB_PATH`)
- `BACKUP_INTERVAL_HOURS`: automatic backup interval in hours (default `24`)
- `BACKUP_KEEP_COUNT`: number of automatic backups to keep (default `7`)
- `INDEX_CHANNEL_ID`: optional numeric ID of the catalog/index Telegram channel; you can instead set it from Telegram with `/setindexchannel`
- `INDEX_WEBSITE_URL`: optional custom website URL; by default the website is at `${WEBHOOK_URL}/index`

## Automatic anime index channel

1. Create a Telegram channel to act as your **Anime Index**.
2. Add the bot as an administrator in that index channel.
3. In a private chat with the bot, set the index channel once:
   - Forward a post from the index channel to the bot, reply to that forwarded post with `/setindexchannel`; or
   - Send `/setindexchannel -1001234567890` with the actual index-channel ID.
4. For each destination anime channel, add the bot as an administrator and grant permission to manage invite links.
5. Post one catalog entry per anime in the index channel, in this format:

   `Bleach | -1001234567890`

   The left side is the anime title; the right side is the destination anime channel's numeric ID. The bot adds the anime to its catalog when it receives the post. Multiple channel entries may use the same anime name.
6. The website is available at `https://YOUR-VOROA-SERVICE-URL/index`. It lists anime entries with configured channels. Tapping an anime opens the Telegram bot, which generates a temporary join-request invite link.

The not-found reply includes a **Try Index** button and website link. Set `INDEX_WEBSITE_URL` only if you want that link to point to a different website.

## Commands

- `/start` — open the bot; website deep links are handled automatically
- `/help` — show help
- `/admin` — open the admin panel
- `/list` — list configured anime (admin)
- `/add Anime Name | @YourChannel` — manually add an anime/channel (admin; legacy method)
- `/remove Anime Name` — remove an anime (admin)
- `/setexpiry 10` — set invite expiry in minutes (admin)
- `/setphoto Anime Name` — reply to a photo to set a poster (admin)
- `/setindexchannel` — reply to a forwarded index-channel post to save that channel, or pass its numeric ID
- `/backup` — create a fresh database backup and send it to the authorized admin
- `/backupstatus` — show the latest backup time and backup settings (admin)

## Important persistence note

The bot can process index-channel posts automatically, but Telegram's Bot API does not let a bot download the channel's old message history after a database is lost. To avoid re-adding entries after a redeploy, configure `DB_PATH` on persistent storage. The index channel is the convenient catalog source; keep the database persistent too.

Keep `BOT_TOKEN` and `WEBHOOK_SECRET` private. A successful web-service deployment alone does not guarantee the webhook is configured correctly; check Voroa runtime logs if updates stop arriving.

## Database backup and persistence upgrade

- The bot creates a SQLite backup on startup and then every `BACKUP_INTERVAL_HOURS` (default 24 hours). It retains the newest `BACKUP_KEEP_COUNT` files (default 7).
- Admins can run `/backup` in a private chat with the bot to create and receive a fresh database backup file. Save that file outside the hosting service as an extra safety copy.
- Admins can run `/backupstatus` to check the latest backup and the configured backup directory.
- **Important:** automatic backups in the same filesystem are not a substitute for persistent storage. In Voroa, attach/configure a persistent disk or volume (if available) mounted at `/var/data`, then set `DB_PATH=/var/data/anime.db` and `BACKUP_DIR=/var/data/backups`. If `/var/data` is not a writable persistent mount on your service, use the actual mount path provided by Voroa. Backups on ephemeral storage can disappear when the service is replaced.
- Before changing `DB_PATH`, export a backup with `/backup`. Changing the path creates/uses a different database and may make the existing catalog appear empty.


## Advanced search and automatic spelling suggestions

When a user searches for a title that is not an exact match, the bot compares it with catalog titles and displays up to five likely matches as buttons. Users can tap a suggestion to request its channel invite link. Matching ignores capitalization, punctuation, and repeated spaces, and uses fuzzy text similarity plus token overlap. The 🌐 Try Index button remains available when no exact match is found.

No new dependency or environment variable is required. Deploy with the existing Voroa build/start commands and keep your existing environment variables.

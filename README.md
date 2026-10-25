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

## Important persistence note

The bot can process index-channel posts automatically, but Telegram's Bot API does not let a bot download the channel's old message history after a database is lost. To avoid re-adding entries after a redeploy, configure `DB_PATH` on persistent storage. The index channel is the convenient catalog source; keep the database persistent too.

Keep `BOT_TOKEN` and `WEBHOOK_SECRET` private. A successful web-service deployment alone does not guarantee the webhook is configured correctly; check Voroa runtime logs if updates stop arriving.

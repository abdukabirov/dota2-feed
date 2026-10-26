<h1 align="center">Dota 2 Feed</h1>

<p align="center">
  A Telegram bot that watches tier-1 Dota 2 tournaments and posts match previews and results to a channel as images.
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/-Python_3.12-3776AB?style=flat-square&logo=python&logoColor=white" />
  <img alt="Telegram" src="https://img.shields.io/badge/-Telegram-26A5E4?style=flat-square&logo=telegram&logoColor=white" />
  <img alt="Docker" src="https://img.shields.io/badge/-Docker-2496ED?style=flat-square&logo=docker&logoColor=white" />
  <img alt="Railway" src="https://img.shields.io/badge/-Railway-0B0D0E?style=flat-square&logo=railway&logoColor=white" />
  <img alt="Stars" src="https://img.shields.io/github/stars/abdukabirov/dota2-feed?style=flat-square&labelColor=343b41" />
</p>

## What it does

- Finds live matches from the tournaments in `TIER1_LEAGUES` (The International, ESL One, BLAST Slam, PGL Wallachia, DreamLeague, Esports World Cup and others).
- Posts a **preview** image once the draft is ready, with picks and bans.
- Posts a **result** image when the match ends.
- Remembers what it already sent, so nothing is posted twice, and retries when a data source is slow or down.
- Runs a check every 4 minutes by default.

## Data sources

| Source | Used for | Key needed |
|---|---|---|
| [OpenDota](https://docs.opendota.com) | Match lists, teams, players | Optional (`OPENDOTA_API_KEY`) |
| [Steam Web API](https://steamcommunity.com/dev) | Live league games | Yes (`STEAM_API_KEY`) |
| [STRATZ](https://stratz.com/api) | Match details, heroes, items | Yes (`STRATZ_TOKEN`) |

## Getting started

You need Python 3.12 and a Telegram bot token from [@BotFather](https://t.me/BotFather). The bot must be an admin of the channel it posts to.

1. Install the dependencies:

   ```bash
   git clone https://github.com/abdukabirov/dota2-feed.git
   cd dota2-feed
   pip install -r requirements.txt
   ```

2. Set your keys as environment variables. Each one can also go in a text file in the project folder (`telegram_token.txt`, `stratz_token.txt`, `steam_api_key.txt`, `opendota_api_key.txt`). These files are ignored by git.

   ```bash
   export TELEGRAM_TOKEN="123456:your-bot-token"
   export CHANNEL_ID="@your_channel"
   export STRATZ_TOKEN="..."
   export STEAM_API_KEY="..."
   ```

3. Check the config, then start the bot:

   ```bash
   python bot.py --validate-config
   python bot.py
   ```

## Commands

| Command | What it does |
|---|---|
| `python bot.py` | Runs the bot and checks on a schedule |
| `--validate-config` | Checks the required settings and exits |
| `--once` | Runs one check and exits |
| `--queue-status` | Prints live and tracked match diagnostics |
| `--post-preview MATCH_ID` | Posts a preview for one match |
| `--post-result MATCH_ID` | Posts a result for one match |
| `--backfill N` | Posts N older matches |
| `--backfill-tier1 N` | Posts N tier-1 main-event matches |
| `--backfill-league LEAGUE_ID` | Posts the 3 previous matches of a league |
| `--backfill-league-story LEAGUE_ID` | Posts preview and result for every finished match in a league |

## Settings

| Variable | Default | Meaning |
|---|---|---|
| `TELEGRAM_TOKEN` | none | Bot token (required) |
| `CHANNEL_ID` | `@dotawatch` | Channel to post to |
| `STATE_FILE` | `data/bot_state.json` | Where sent matches are remembered |
| `CHECK_INTERVAL_MINUTES` | `4` | How often to check |
| `MAX_MATCHES_PER_CHECK` | `10` | Limit per check |
| `MIN_PREVIEW_BANS` | `14` | Bans needed before a preview is posted |
| `DEFAULT_STREAM_URL` | Twitch link in `config.py` | Stream link added to posts |

To follow other tournaments, add their league IDs to `TIER1_LEAGUES` in `config.py`.

## Deploy

The repository includes a `Dockerfile` and `railway.toml`, so it deploys to [Railway](https://railway.app) as is. Add the variables above in the service settings. Set `STATE_FILE` to a path on a mounted volume, otherwise the bot forgets what it posted after each restart.

You can also run it anywhere with Docker:

```bash
docker build -t dota2-feed .
docker run -d --env-file .env -v "$PWD/data:/app/data" dota2-feed
```

## Tests

```bash
python -m unittest discover tests
```

## Project structure

```
bot.py              Entry point
config.py           Leagues, logos, defaults
src/bot/            Scheduler, discovery, posting, readiness checks
src/core/           API client, cache, logging, error handling
src/services/       Image rendering, captions, match adapter
src/models/         Match and player types
tests/              Unit tests
```

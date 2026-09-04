---
title: Configuration
hidden: false
---

pantsman is configured in two places: **`config/default.json`** for behavior, and **`.env`** for secrets. Neither requires a restart tool or framework — edit, restart the process, done.

## config/default.json

| Key | Default | What it does |
| --- | ------- | ------------ |
| `botName` | `pantsman` | IRC nick. The bot always replies when this appears in a message, and ignores its own messages and DMs. |
| `server` | `irc.esper.net` | IRC server to connect to. |
| `channels` | `["#selectbutton"]` | Channels to join and learn from. |
| `replyChance` | `0.05` | Probability the bot replies to a channel message that *doesn't* mention it. Mentions always get a reply. |
| `corpusFile` | `./corpus.json` | Where the Markov chain is saved. Resolved relative to the repo, not the working directory. |
| `maxWords` | `30` | Default word limit for generated text, both in IRC replies and `GET /generate` (overridable per-request with `?maxWords=`). |
| `reconnectDelayMs` | `60000` | How long to wait before reconnecting after a dropped IRC connection. |
| `apiPort` | `3000` | HTTP API port. |
| `apiHost` | `127.0.0.1` | HTTP API bind address. |

<Callout icon="🔒" theme="warn">
  **Think twice before changing `apiHost`.** The loopback bind is the API's main line of defense: `POST /train` accepts arbitrary text into the corpus, so anyone who can reach it can poison what the bot says. If you must expose the API, set `API_TOKEN` first and put a reverse proxy with TLS in front — don't bind the bare Express server to `0.0.0.0`.
</Callout>

<Callout icon="⏱️" theme="default">
  **Why is `reconnectDelayMs` a full minute?** Reconnecting before the IRC server has reaped the dead connection means the nick is still held, and node-irc responds by quietly renaming the bot `pantsman1`, then `pantsman2`. Lower this and the bot starts collecting numbered aliases.
</Callout>

### Overriding without editing the file

The [config](https://www.npmjs.com/package/config) package reads `NODE_CONFIG` from the environment, which is handy for one-off overrides:

```bash
NODE_CONFIG='{"apiPort":3001}' npm run api
```

## Environment variables (.env)

Both npm scripts pass `--env-file-if-exists=.env`, so variables load automatically from a `.env` file in the repo root. The file is gitignored — it holds secrets.

| Variable | Required? | What it does |
| -------- | --------- | ------------ |
| `RESTLESS_KEY` | Recommended | API key for the `@restlessai/sdk` request logging. Request and response `text` bodies are redacted before capture, so the logging sees traffic shape, not what the bot says. |
| `API_TOKEN` | Optional | When set, `POST /train` requires `Authorization: Bearer <token>` and returns `401` without it. When unset, `/train` is open — acceptable only behind the default localhost bind. Generate one with `openssl rand -hex 32`. |

The read-only routes (`GET /generate`, `GET /stats`) are never gated — only `/train` checks the token.

## Files that live outside git

Two files are deliberately not in the repo and travel by hand (`scp`) between machines:

- **`.env`** — the secrets above.
- **`corpus.json`** — the bot's learned brain. Runtime data, gitignored, and the one thing that's neither in git nor recreatable. See [Training the Corpus](/docs/training-the-corpus) for how it's built and saved.

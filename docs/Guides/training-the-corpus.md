---
title: Training the Corpus
hidden: false
---

The **corpus** is pantsman's brain: a word-pair Markov chain stored as `corpus.json` in the repo root. For every word the bot has seen, the chain records which words have followed it. Generation picks a random starting word and hops from successor to successor until it reaches the word limit or a dead end.

Everything below makes that chain bigger. There's no way to make it smaller except deleting `corpus.json` and starting over — training is additive.

## The three ways pantsman learns

<Tabs>
  <Tab title="Channel chatter">
    When running as the full bot (`npm start`), **every message** in its channels is fed into the chain automatically — no configuration needed. The bot ignores DMs and its own messages, so it doesn't feed back on itself.

    This is the slow, organic way: the bot gradually starts sounding like the channel it lives in.
  </Tab>

  <Tab title="POST /train">
    Push text directly over the HTTP API:

    ```bash
    curl -X POST localhost:3000/train \
      -H 'Content-Type: application/json' \
      -d '{"text":"any text you want the bot to learn"}'
    # 204 No Content
    ```

    If `API_TOKEN` is set in `.env`, the request needs a bearer token:

    ```bash
    curl -X POST localhost:3000/train \
      -H "Authorization: Bearer $API_TOKEN" \
      -H 'Content-Type: application/json' \
      -d '{"text":"any text you want the bot to learn"}'
    ```

    Request bodies are limited to **64 KB** — `/train` payloads are meant to be chat-message sized. Empty or missing `text` gets a `400`. Full details in the [API reference](/reference/train-1).
  </Tab>

  <Tab title="Bluesky ingest">
    Seed the corpus in bulk from a Bluesky account's recent posts:

    ```bash
    node ingest-bluesky.js <handle> [days]   # days defaults to 21
    ```

    The script fetches the account's original posts (reposts and replies are skipped), trains on them, and saves the result. It **adds to** whatever corpus already exists rather than replacing it — it loads `corpus.json` before training and saves the merged result. To re-seed cleanly from a different account, delete `corpus.json` first.
  </Tab>
</Tabs>

<Callout icon="☣️" theme="warn">
  **Corpus poisoning is the threat model.** Anyone who can reach `POST /train` — or say anything in the bot's channel — controls what the bot says. The API defends itself with the localhost bind and the optional bearer token (see [Configuration](/docs/configuration)); the channel is defended by, well, moderation.
</Callout>

## How the corpus is saved

While the bot runs, the chain lives in memory. It's written to `corpus.json` when the process shuts down cleanly:

- ✅ `Ctrl-C` (SIGINT), `systemctl stop`/`restart` (SIGTERM), and a closing terminal (SIGHUP) all save first
- ❌ `kill -9` and OOM kills skip the save — everything learned since startup is lost

If `corpus.json` is missing or corrupt at startup, the bot logs a warning and starts with an empty chain rather than crashing. An empty chain means `GET /generate` returns `503 corpus is empty` until something trains it.

<Callout icon="💾" theme="okay">
  `corpus.json` is the one file that's neither in git nor recreatable. If you care about your bot's accumulated personality, back it up — a daily `cp` in cron is plenty.
</Callout>

## Checking on the brain

`GET /stats` reports the chain's size — distinct words and total recorded transitions:

```bash
curl localhost:3000/stats
# {"words":1234,"transitions":5678}
```

Watch `transitions` climb as training happens. Details in the [API reference](/reference/stats-1).

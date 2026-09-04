---
title: Quickstart
hidden: false
---

Go from a fresh clone to a running bot (or standalone API) in about five minutes.

## Prerequisites

- **Node.js ≥ 22.9** — the npm scripts use `--env-file-if-exists`, which older Node doesn't have and fails on with `bad option`. Distro-packaged Node is usually too old; use [NodeSource](https://github.com/nodesource/distributions) or nvm.
- **Git**

## 1. Clone and install

```bash
git clone https://github.com/jesseyowell/pantsman.git
cd pantsman
npm ci
npm test
```

## 2. Seed the corpus

pantsman starts with an empty brain: until it learns something, `GET /generate` returns `503 corpus is empty`. The fastest way to give it one is to ingest a Bluesky account's recent posts:

```bash
node ingest-bluesky.js some-handle.bsky.social
```

<Callout icon="⚠️" theme="warn">
  The handle is required on purpose — whatever account you point it at is what the bot will repeat in the channel, so pick deliberately. By default it pulls the last 21 days of posts; pass a second argument to change that (`node ingest-bluesky.js some-handle.bsky.social 90`).
</Callout>

This writes `corpus.json` in the repo root. You can also skip this step and let the bot learn from channel chatter or [`POST /train`](/reference/train-1) — see [Training the Corpus](/docs/training-the-corpus).

## 3. Run it

Both modes serve the HTTP API on `127.0.0.1:3000`; the difference is whether an IRC connection comes with it.

<Tabs>
  <Tab title="Full bot (IRC + API)">
    ```bash
    npm start
    ```

    Connects to the configured IRC server and channels (see [Configuration](/docs/configuration)), learns from every channel message, and replies when mentioned — plus a 5% random chance on any other message. The HTTP API runs alongside it.
  </Tab>

  <Tab title="API only">
    ```bash
    npm run api
    ```

    Serves just the HTTP API — no IRC connection. Useful for local development or for using the Markov chain as a plain text-generation service.

    Don't run both modes at once as-is: they read the same `apiPort`, and the second one to start dies with `EADDRINUSE`.
  </Tab>
</Tabs>

Both scripts auto-load environment variables from a `.env` file in the repo root if one exists. You don't need one to get started — see [Configuration](/docs/configuration) for what `RESTLESS_KEY` and `API_TOKEN` do.

## 4. Verify

```bash
curl localhost:3000/stats
# {"words":1234,"transitions":5678}

curl localhost:3000/generate
# {"text":"the quick brown fox ..."}

curl -X POST localhost:3000/train \
  -H 'Content-Type: application/json' \
  -d '{"text":"teach the bot this sentence"}'
# 204 No Content
```

<Callout icon="💡" theme="okay">
  The bind is IPv4 loopback only. If `localhost` ever inexplicably fails to connect, try `curl 127.0.0.1:3000` — `localhost` can resolve to `::1` first.
</Callout>

## 5. Shut it down cleanly

The corpus lives in memory while the bot runs and is saved to `corpus.json` on shutdown. `Ctrl-C`, `SIGTERM`, and a closing terminal (`SIGHUP`) all trigger the save; `kill -9` does not — anything learned since startup is lost.

## Next steps

<Cards>
  <Card title="Training the Corpus" href="/docs/training-the-corpus" icon="fa-duotone fa-brain">All three ways pantsman learns, and how to care for corpus.json</Card>

  <Card title="Configuration" href="/docs/configuration" icon="fa-duotone fa-sliders">Change the IRC server, reply chance, port, and secrets</Card>

  <Card title="API Reference" href="/reference/generate-1" icon="fa-duotone fa-code-simple">Try the endpoints in the interactive playground</Card>
</Cards>

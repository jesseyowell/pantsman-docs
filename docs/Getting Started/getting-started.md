---
title: Welcome to pantsman
hidden: false
---

**pantsman** is a Markov-chainer IRC bot with an HTTP API. It sits in a channel, learns from everything it reads, and occasionally talks back — remixing what it has heard into new (and usually absurd) sentences. The same chain is exposed over HTTP, so you can generate text, feed it new material, and check on the size of its brain from anywhere.

<Cards>
  <Card title="Quickstart" href="/docs/quickstart" icon="fa-duotone fa-rocket-launch">Go from clone to a talking bot in about five minutes</Card>

  <Card title="API Reference" href="/reference/generate-1" icon="fa-duotone fa-code-simple">Generate text, train the chain, and read stats over HTTP</Card>

  <Card title="Configuration" href="/docs/configuration" icon="fa-duotone fa-sliders">Every config key and environment variable, explained</Card>
</Cards>

<br />

## How it works

pantsman builds a word-pair Markov chain: for every word it sees, it remembers which words have followed it. To generate a sentence it picks a starting word, then repeatedly hops to a random recorded successor until it hits the word limit or a dead end. No neural networks, no API keys to an LLM — just a big JSON file of word transitions called the **corpus**.

The bot learns from three sources:

- **Channel chatter** — every message in its IRC channels is fed into the chain
- **`POST /train`** — push arbitrary text over the HTTP API
- **Bluesky ingest** — seed the corpus from a Bluesky account's recent posts

It speaks up whenever someone says its name, and otherwise replies to a small random fraction of channel messages (5% by default).

<br />

## The Basics

<Cards>
  <Card kind="tile" title="Training the Corpus" href="/docs/training-the-corpus" icon="fa-duotone fa-brain">The three ways pantsman learns, and how the corpus is stored</Card>

  <Card kind="tile" title="Generate Text" href="/reference/generate-1" icon="fa-duotone fa-wand-magic-sparkles">`GET /generate` — make the bot say something</Card>

  <Card kind="tile" title="Train Endpoint" href="/reference/train-1" icon="fa-duotone fa-graduation-cap">`POST /train` — add text to the chain over HTTP</Card>

  <Card kind="tile" title="Corpus Stats" href="/reference/stats-1" icon="fa-duotone fa-chart-simple">`GET /stats` — how many words and transitions it knows</Card>

  <Card kind="tile" title="Configuration" href="/docs/configuration" icon="fa-duotone fa-sliders">IRC connection, reply chance, ports, and secrets</Card>

  <Card kind="tile" title="Source on GitHub" href="https://github.com/jesseyowell/pantsman" icon="fa-duotone fa-code-branch">Read the code — it's small on purpose</Card>
</Cards>

<br />

<Callout icon="🔒" theme="default">
  **Security model in one sentence:** the API binds to `127.0.0.1` by default, and `POST /train` can be gated behind a bearer token — so out of the box, nothing off the box can touch it. See [Configuration](/docs/configuration) for details.
</Callout>

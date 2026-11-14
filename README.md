# Mysocial Intelligence

[![Listed on mcpservers.org](https://mcpservers.org/badge.svg)](https://mcpservers.org/servers/mysocial-io-mcp)

**A social media MCP server for creators.** Your Instagram, TikTok, YouTube,
LinkedIn and Threads history — posts, metrics, transcripts, comments and
audience — readable by Claude, ChatGPT or Codex, bundled with eight skills that
know how to use it.

Most social media MCP servers *post* for you. This one **reads what you already
published**, then researches everything around it.

### Where this fits

Social media MCP servers do four different jobs, and picking the wrong one is
the usual mistake:

| Job | What you want | Use |
|---|---|---|
| **Publish** | The AI posts and schedules for you | Postiz, Buffer, Typefully, Ayrshare |
| **Report** | The AI reads your metrics — views, watch time, CTR | Supermetrics, Windsor.ai, Porter Metrics |
| **Research** | The AI reads *other people's* public content | Bright Data, Apify |
| **Remember** | The AI knows your own body of work | **this server** |

If you want the AI to publish, use a scheduler — this server has no posting
tools and is not trying to acquire any.

## Install

**Claude Code**

```
/plugin marketplace add mysocialio/mysocial-intelligence
/plugin install mysocial-intelligence@mysocial-intelligence
```

**Skills only** (any agent that supports Agent Skills)

```
npx skills add mysocialio/mysocial-intelligence
```

**Codex** — the same plugin manifest ships in `.codex-plugin/`. Point a
repo-scoped or personal marketplace at this repository.

**Any MCP client** — connector URL `https://app.mysocial.io/mcp`
(streamable HTTP, OAuth 2.1 with dynamic client registration). Setup for every
client: https://mysocial.io/mcp/

## What is inside

| Skill | What it does |
|---|---|
| `analyze-what-works` | What repeatedly works for you and why a post hit: the mechanism behind your best posts, not a list of them. |
| `find-next-content` | What to post next: proven but under-used mechanisms, unmet audience demand, and what to avoid because it is saturated. |
| `make-next-post` | The strongest evidence-backed next post: hooks, scripts, captions, CTAs, formats — from what already worked for *you*. |
| `research-creator-universe` | What is working across public creators in your topics, and what you have not covered. |
| `organic-growth-sprint` | Grow an account from niche references: find creators and their outlier posts, see how they work, and build a filming-ready batch. |
| `brand-docs` | Read and maintain your living Brand Studio documents — audience, voice, strategy notes. |
| `brand-products` | Your product catalogue: offers, prices, lifecycle, how they ladder. |
| `outreach` | Find brands worth pitching and the verified people to reach, from your own numbers. |

The MCP server exposes 64 tools. Connecting your channels and reading your
history is free; transcripts, comment text, Hit DNA, audience psychographics,
Creator Universe research and brand partnerships are on paid plans — the
server tells the assistant which plan unlocks what, and never spends on an
agent's initiative.

## Account

The plugin signs you into your existing Mysocial account with OAuth. Create
one free at https://app.mysocial.io — connect a channel and the history is
readable within minutes.

- Website: https://mysocial.io/mcp/
- Support: https://mysocial.io/support/
- Privacy: https://mysocial.io/privacy-policy/ · Terms: https://mysocial.io/terms-of-service/

## About this repository

This is the public artifact. The skills are authored in Mysocial's main
repository and mirrored here on every release; the MCP host is stamped from a
single release config, so the URLs in this repo are always the production
server. Issues and pull requests are welcome for the skills and docs.

License: MIT © GoldenGlow AB.

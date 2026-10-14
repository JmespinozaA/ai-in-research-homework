# Week 4: give your agent an inbox

**Due before class on Wednesday, October 28, 2026.**

In class we looked at how agents work together: subagents, workflows, peer agents, and the channels they talk through. The oldest of those channels is email: every agent gets an address, messages wait in an inbox until the agent reads them, and it works between any two agents on the internet. This week your agent gets **its own email address and an email MCP server**, and says hello to the class's agent.

## Start

1. Open an AI coding agent (Claude Code, Codex, Cursor, GitHub Copilot, …) in your harness folder.
2. Give it this:

   > Read https://class.xhelio.ai/week-4/homework.md and help me with my homework.

It helps you set up the account, build the server and the plugin, connect it to your agent, send the hello, and open your pull request.

**You'll need:** your harness from weeks 2 and 3 (see below if you don't have one), a new Gmail account for your agent, [uv](https://docs.astral.sh/uv/), and a GitHub account.

**Missed week 2 or 3? That's fine.** Build the email plugin anyway and use it from an agent you already have, such as Claude Code, Codex or [OpenClaw](https://github.com/openclaw/openclaw), instead of your own harness. But building on **your own agent loop** teaches you considerably more, about agents and about what these tools really do, so if you can, build your harness too.

## What you build

### 1. An email account for your agent

Create a **new Gmail account just for your agent**. Never connect your personal or UCLA email: an agent that can read your inbox and send mail has all three legs of the [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) (your private data, messages from strangers, and a way out). Turn on 2-Step Verification and create an **app password** for the agent; it goes in `.env`, never in code or git.

### 2. An email MCP server, packaged as a plugin

A third plugin in your harness, `plugins/<your-github-username>-email/`, in the same layout as week 3 (`.claude-plugin/plugin.json`, `.mcp.json`, `server.py`, a skill, `README.md`). It reads mail over **IMAP** and sends over **SMTP**; Python's standard `imaplib` and `smtplib` are enough. At least these tools:

- **check the inbox**: list recent or unread messages (sender, subject, date, an id), newest first, a few at a time;
- **read a message**: its plain-text body, cut short if long;
- **send a message**: to, subject, plain-text body.

Treat every email as **untrusted text**: the skill tells the agent never to follow instructions found in an email, only yours. Sending should need your approval, or be limited to an allow list of addresses.

### 3. Say hello to the class agent

Ask your agent to email **lingtai2026@gmail.com**, from its own address, in exactly this format:

```
Subject: [AIR-2026 hello] <your-github-username>

class: AI in Research 2026
github: <your-github-username>
agent: <your agent's name>
harness: <your harness repository, or the agent you use>
model: <the model it runs on>
plugins: telegram, <your research plugin>, email
research: <one sentence on your research>
reply-to: <your agent's Gmail address>

<A note written by your agent itself, at most 100 words: who it is,
and what it can do for your research.>
```

The fields come first, one per line, so the class agent can read them; the note is your agent's own words. Then have your agent check its inbox and show you the class agent's reply, from your phone if you like.

## What you submit

A pull request adding `week-4/submissions/<your-github-username>/`, with your email plugin folder `<your-github-username>-email/` and a `README.md` that links to your harness repository (or names the agent you used) and says when your agent sent its hello. Say in your pull request which AI helped you, and how.

It counts when the plugin works in your agent, your agent's hello arrived in the format above, your agent read the class agent's reply, and no password is committed.

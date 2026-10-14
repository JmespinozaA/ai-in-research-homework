# Week 3: talk to your agent from your phone

**Due before class on Wednesday, October 21, 2026.**

In class we looked at the layers Codex and Claude Code put around the agent loop: tools, MCP servers, skills, plugins, permissions and hooks. This week your harness from week 2 gets its first plugin: a **Telegram MCP server**. When you're done, you message your agent from your phone, it works on your computer, and it answers in the chat.

The idea behind it: your harness keeps only **the loop and the `bash` tool**. Everything else arrives as a plugin. Telegram is the first one; later in the quarter, more capabilities, and other agents, come in the same way.

## Start

1. Open an AI coding agent (Claude Code, Codex, Cursor, GitHub Copilot, …) in your week 2 harness folder.
2. Give it this:

   > Read https://class.xhelio.ai/week-3/homework.md and help me with my homework.

It helps you create the bot, build the MCP server and the plugin, extend your harness, test it from your phone, and open your pull request.

**You'll need:** your week 2 harness and its model API, [Telegram](https://telegram.org) on your phone (free; it needs a phone number), [uv](https://docs.astral.sh/uv/), and a GitHub account. If you didn't finish week 2, finish that first: this week builds on it.

## What you build

### 1. A Telegram MCP server, packaged as a plugin

A folder in your harness repository, `plugins/<your-github-username>-telegram/`, laid out like a [Claude Code plugin](https://code.claude.com/docs/en/plugins-reference):

```
<your-github-username>-telegram/
├── .claude-plugin/plugin.json   # name, version, description, author
├── .mcp.json                    # how to start the server
├── server.py                    # the MCP server
├── skills/telegram/SKILL.md     # how an agent should use it
└── README.md
```

The server talks to the [Telegram Bot API](https://core.telegram.org/bots/api) by long polling, so nothing needs to be reachable from the internet. It has at least two tools: one that **sends** a message to your chat, and one that **waits for** new messages from it. It reads the bot token and your chat ID from environment variables, and it **drops every message that isn't from your own chat ID**.

The same server must also work in a second host: Claude Code (`claude --plugin-dir …`) or Codex (`codex mcp add …`). That's the point of MCP: write it once, use it anywhere.

### 2. Your harness loads plugins

- It finds plugins in its `plugins/` folder, starts the MCP servers each one lists in `.mcp.json`, asks each for its tools, and offers them to the model next to `bash`.
- It lists each plugin skill's name and description in the system prompt; the model reads a skill's full `SKILL.md` with `bash` only when it needs it (progressive disclosure).
- It still asks you before every `bash` command. MCP tools you trust (such as sending a message) can be on a short allow list in a config file; anything else asks.

### 3. A Telegram mode

Started with a flag such as `--telegram`, your harness takes its questions from Telegram instead of the keyboard, runs the loop, and sends the answer back. When the model wants to run a command, it asks for your approval **in the chat**, and you reply yes or no.

Write the loop and the plugin loader yourself, with your AI agent's help. You may use the official [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) for the server and the client; no agent frameworks.

## Safety

A Telegram bot connected to `bash` on your computer has all three legs of the [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/): your files, messages from the internet, and a way to send things out. So:

- **Only your chat ID gets in.** Anyone can find a bot and message it.
- **You approve every command.** Keep it on.
- **The bot token is a password.** Keep it in `.env`, never commit it, never paste it into a chat with an AI. If it leaks, revoke it in BotFather with `/revoke`.
- Messages to a bot are not end-to-end encrypted. Don't send anything secret through it.

## What you submit

A pull request adding your plugin folder as `week-3/submissions/<your-github-username>/`, the same files as `plugins/<your-github-username>-telegram/` in your harness repository. Its `README.md` says how to configure and load the plugin, and links to your harness repository with how to run it in Telegram mode. Say in your pull request which AI helped you, and how.

It counts when your harness repository is public and updated, the plugin works in your harness and in Claude Code or Codex, you can chat with your agent from your phone and approve commands there, messages from anyone else are ignored, and no token is committed.

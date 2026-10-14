# Week 3: talk to your agent from your phone, about your research

**Due before class on Wednesday, October 21, 2026.**

In class we looked at the layers Codex and Claude Code put around the agent loop: tools, MCP servers, skills, plugins, permissions and hooks. This week your harness from week 2 gets its first two **plugins**: a **Telegram MCP server**, so you can message your agent from your phone, and an **MCP server for your own research**, so it can do something useful when you do.

The idea behind it: your harness keeps only **the loop and the `bash` tool**. Everything else arrives as a plugin. These are the first two; later in the quarter, more capabilities, and other agents, come in the same way.

## Start

1. Open an AI coding agent (Claude Code, Codex, Cursor, GitHub Copilot, …) in your week 2 harness folder.
2. Give it this:

   > Read https://class.xhelio.ai/week-3/homework.md and help me with my homework.

It helps you create the bot, build both MCP servers as plugins, extend your harness, test it from your phone, and open your pull request.

**You'll need:** your week 2 harness and its model API, [Telegram](https://telegram.org) on your phone (free; it needs a phone number), [uv](https://docs.astral.sh/uv/), and a GitHub account. **Missed week 2? That's fine.** Build the two plugins anyway and use them from an agent you already have, such as Claude Code, Codex or [OpenClaw](https://github.com/openclaw/openclaw), instead of your own harness, and skip the parts about your harness (sections 3 and 4). Chatting from your phone then depends on what that agent supports. But building on **your own agent loop** teaches you considerably more, about agents and about what these tools really do, so if you can, finish week 2's harness first.

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

### 2. An MCP server for your research, packaged as a plugin

A second plugin, `plugins/<your-github-username>-<name>/`, in the same layout, whose tools do something from **your own research**. Pick one thing you'd like to ask your agent for from your phone. For example:

- **Data:** fetch and summarize data from an archive you use (CDAWeb, OMNI, PDS, a mission's API, a lab database).
- **Your code:** wrap an analysis script, a fit or a model your group already has, so the agent can run it with parameters.
- **Literature:** search NASA ADS or arXiv for papers on a topic, or look up a paper's abstract and citations.
- **A calculation:** a unit converter, a coordinate transform or an ephemeris for your field.

Two to five tools is plenty. Write them for a reader with no context ([the slide](https://class.xhelio.ai/week-3/slides#tool-design)): clear names and descriptions, parameters with units and formats, short results (save big ones to a file and return a summary), and errors that say what to try next. Add a skill that tells an agent when and how to use the tools. Read-only tools are best; if it needs an API key (for example an ADS token), it comes from an environment variable, never from the code. It must also work in Claude Code or Codex.

### 3. Your harness loads plugins

- It finds plugins in its `plugins/` folder, starts the MCP servers each one lists in `.mcp.json`, asks each for its tools, and offers them to the model next to `bash`.
- It lists each plugin skill's name and description in the system prompt; the model reads a skill's full `SKILL.md` with `bash` only when it needs it (progressive disclosure).
- It still asks you before every `bash` command. MCP tools you trust (such as sending a message) can be on a short allow list in a config file; anything else asks.

### 4. A Telegram mode

Started with a flag such as `--telegram`, your harness takes its questions from Telegram instead of the keyboard, runs the loop, and sends the answer back. When the model wants to run a command, it asks for your approval **in the chat**, and you reply yes or no. Show it working: from your phone, ask a question that makes the agent use your research plugin.

Write the loop and the plugin loader yourself, with your AI agent's help. You may use the official [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) for the server and the client; no agent frameworks.

## Safety

A Telegram bot connected to `bash` on your computer has all three legs of the [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/): your files, messages from the internet, and a way to send things out. So:

- **Only your chat ID gets in.** Anyone can find a bot and message it.
- **You approve every command.** Keep it on.
- **The bot token is a password.** Keep it in `.env`, never commit it, never paste it into a chat with an AI. If it leaks, revoke it in BotFather with `/revoke`.
- Messages to a bot are not end-to-end encrypted. Don't send anything secret through it.

## What you submit

A pull request adding both plugin folders under `week-3/submissions/<your-github-username>/`:

```
week-3/submissions/<your-github-username>/
├── README.md                          # link to your harness repository, how to run it in Telegram mode
├── <your-github-username>-telegram/   # the same files as in your harness's plugins/ folder
└── <your-github-username>-<name>/     # your research plugin
```

Each plugin's own `README.md` says what it does and how to configure and load it. Say in your pull request which AI helped you, and how.

It counts when your harness repository is public and updated, both plugins work in your harness and in Claude Code or Codex, you can chat with your agent from your phone, approve commands there and have it use your research plugin, messages from anyone else are ignored, and no token or key is committed. Using an existing agent instead of your own harness? Then it counts when both plugins work in that agent and it answers a research question with yours.

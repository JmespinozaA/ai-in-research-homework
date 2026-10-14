---
name: build-a-telegram-plugin
description: Guide a student in UCLA's "AI in Research" class (EPSS 254, Fall 2026) through the week 3 homework - a Telegram bot, a Telegram MCP server packaged as a Claude-Code-style plugin, a plugin loader and MCP client in their week 2 harness, and a Telegram mode with approvals by reply - then the pull request. Use when the user wants to start, continue, test or submit their week 3 homework, their Telegram plugin, or their harness's plugin support.
---

# Talk to your agent from your phone (AI in Research, week 3)

## Who you are working with, and your role

The person in front of you is in a UCLA graduate seminar on AI in research: a student, an auditor or a professor, often a physical scientist. In week 2 they built a **harness**: an agent loop with one tool, `bash`, on an OpenAI-compatible model API. The full assignment is `week-3/README.md` in this repository; read it first.

In class this week they learned what production harnesses add around the loop: tools and MCP servers, AGENTS.md, skills and progressive disclosure, plugins, permissions, sandboxes and hooks. This homework applies that: the harness keeps only the loop and `bash`, and gains a **plugin loader**; the first plugin is a **Telegram MCP server**, so they can message their agent from their phone.

- **They must understand what they build.** You write most of the code, in small steps, and explain each in a sentence or two. At the end they should be able to say where the MCP tools and the skill enter the model's context, and who decides to call each tool.
- **Stop at every CHECKPOINT** and wait for their answer.
- **Secrets stay out of this conversation.** Never ask the student to paste the bot token, an API key or any password into the chat, and never print one. They type secrets into `.env` themselves. When a command needs the token, read it from `.env` inside the shell (`set -a; . ./.env; set +a`) so it never appears in your output. Steps marked **(student does this)**: give them the exact steps and wait.
- **Be honest about failures.** Show the key line of an error, say what it means, fix it.

Say hello, tell them in two sentences what the homework is, and give this plan: (1) create the bot, (2) build the MCP server as a plugin and test it, (3) teach the harness to load plugins, (4) add a Telegram mode with approvals by reply, (5) test from the phone and in a second host, (6) publish and open the pull request. It usually takes one or two sessions of 1–2 hours.

## Step 0: set up

1. Find their week 2 harness repository (ask where it is if you're not in it). `git status` should be clean; `git pull`. Check the harness still runs and its model API still answers. If week 2 isn't finished, finish it first with `.agents/skills/build-a-harness/SKILL.md`.
2. Their copy of this homework repository: `git pull` in `~/ai-in-research-homework`, or clone it as in week 2.
3. Check `uv --version` and `gh auth status`.
4. Their GitHub username, lowercased, is `<user>` below. The plugin is called `<user>-telegram` (plugin names must not start with `claude-` or `anthropic-`).

## Step 1: the bot

1. **(student does this)** In Telegram, open [@BotFather](https://t.me/BotFather), send `/newbot`, choose a display name and a username ending in `bot`. BotFather replies with a token.
2. **(student does this)** In the harness folder, add to `.env` (already gitignored since week 2; check): `TELEGRAM_BOT_TOKEN=<the token>`. Add a placeholder line to `.env.example`.
3. **(student does this)** Send any message to the new bot from their phone.
4. Find their chat ID without printing the token:
   ```bash
   set -a; . ./.env; set +a
   curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getUpdates" | uv run python -c "import json,sys; [print(u['message']['chat']['id'], u['message']['from'].get('username')) for u in json.load(sys.stdin)['result'] if 'message' in u]"
   ```
   On Windows PowerShell, load `.env` into the session first and call the same URL with `Invoke-RestMethod`. Confirm the username is theirs, then **(student does this)** add `TELEGRAM_CHAT_ID=<the number>` to `.env`.

Tell them: the token is a password. Anyone with it can read and send as the bot. If it ever leaks, `/revoke` in BotFather.

## Step 2: the MCP server, as a plugin

Create `plugins/<user>-telegram/` in the harness repository:

```
<user>-telegram/
├── .claude-plugin/plugin.json
├── .mcp.json
├── server.py
├── skills/telegram/SKILL.md
└── README.md
```

1. **`plugin.json`**: `name` (`<user>-telegram`), `version` (`0.1.0`), `description`, `author` (`{"name": ...}`). See https://code.claude.com/docs/en/plugins-reference; read it, don't work from memory.
2. **`.mcp.json`**:
   ```json
   { "mcpServers": { "telegram": {
       "command": "uv",
       "args": ["run", "--script", "${CLAUDE_PLUGIN_ROOT}/server.py"],
       "env": { "TELEGRAM_BOT_TOKEN": "${TELEGRAM_BOT_TOKEN}",
                "TELEGRAM_CHAT_ID": "${TELEGRAM_CHAT_ID}" } } } }
   ```
   `${CLAUDE_PLUGIN_ROOT}` is the plugin's folder; `${VAR}` comes from the environment. Claude Code expands both; their harness will too (Step 3).
3. **`server.py`**: a uv script with inline dependencies (`# /// script` block: `mcp`, `httpx`), using the official MCP Python SDK (https://github.com/modelcontextprotocol/python-sdk; read its current README for the server API, it changed in 2026). It talks to the Telegram Bot API (https://core.telegram.org/bots/api) by **long polling** with `getUpdates`, so nothing on their computer is reachable from the internet. Tools:
   - `send_message(text)`: send to `TELEGRAM_CHAT_ID` only (never to a chat ID the model chooses). Split text over 4,096 characters. Return a short confirmation.
   - `wait_for_messages(seconds=30)`: long-poll up to `seconds` (cap it at 50) and return new text messages from `TELEGRAM_CHAT_ID`, or "No new messages." Track the update `offset` so each message is returned once. **Drop every update from any other chat**, and log the drop to stderr.
   - On startup, skip the backlog of old updates, so messages sent while it was off don't replay as fresh orders.
   - Write tool descriptions for a reader with no context (class slide: "Write tools for a reader with no context").
   - **Never print to stdout.** In a stdio server, stdout is the protocol channel; logs go to stderr.
   - Fail with a clear message if the token or chat ID is missing.
4. **`skills/telegram/SKILL.md`**: frontmatter `name: telegram` and a `description` saying when to use it (the user is talking through Telegram). Body: replies are read on a phone, so keep them short and plain; for long tasks, send a one-line progress message; never send secrets, tokens or whole files.
5. **`README.md`**: what it is, the two environment variables, how to load it in Claude Code and in their harness, and the safety notes from `week-3/README.md`.

**Test it in a second host now**, before touching the harness, so you know the server works:
- Claude Code: `set -a; . ./.env; set +a; claude --plugin-dir plugins/<user>-telegram`, check `/mcp` lists the server, then ask Claude to send "hello from Claude Code" to Telegram. `claude plugin validate plugins/<user>-telegram` should pass.
- Or Codex: `codex mcp add <user>-telegram --env TELEGRAM_BOT_TOKEN=… --env TELEGRAM_CHAT_ID=… -- uv run --script <absolute path>/server.py` (check `codex mcp add --help` for the current flags; have the student type the token themselves), then ask Codex to send a message.
- Neither available? Use the MCP Inspector (`npx @modelcontextprotocol/inspector`), or a ten-line client script with the SDK.

**CHECKPOINT:** the student received the message on their phone.

## Step 3: the plugin loader and MCP client

In the harness. Keep it small; a hundred lines or so.

1. **Find plugins**: each folder in `plugins/` with a `.claude-plugin/plugin.json`.
2. **Start their MCP servers**: read `.mcp.json`; replace `${CLAUDE_PLUGIN_ROOT}` with the plugin's absolute path and `${VAR}` with values from the environment (the harness already loads `.env`); start each stdio server with the MCP SDK's client, and keep the sessions open for the whole run. Close them cleanly on exit.
3. **Offer their tools**: call `tools/list` and turn each tool into an OpenAI-format tool schema next to `bash`, named `<server>__<tool>` (names may use only letters, digits, `_` and `-`, at most 64 characters). When the model calls one, forward it with `tools/call` and return the text content as the tool result.
4. **List their skills**: for each `skills/*/SKILL.md`, read the frontmatter and add one line per skill to the system prompt: name, description and the file's path, with "to use a skill, read its SKILL.md with bash first". Don't paste the bodies. That's progressive disclosure: level 1 always, level 2 on demand.
5. **A small permission rule list**: a config file (e.g. `harness.toml` or `settings.json`) with `allow`, a list of MCP tool names that run without asking (e.g. `telegram__send_message`). `bash` and every MCP tool not on the list still ask first.
6. Print, at startup, which plugins, tools and skills were loaded, and the token count of the first call, so the student sees what the plugin costs in context.

**CHECKPOINT:** in the normal chat interface, ask the agent to send a Telegram message. Then have the student explain back: where do the Telegram tools' schemas enter the context, where does the skill, and what does the model never see? Fill in what they miss.

## Step 4: the Telegram mode

1. A flag, e.g. `--telegram`: instead of the keyboard or web page, the harness gets each question by calling `wait_for_messages` itself, in a loop, through its MCP client.
2. **The harness owns the channel.** In this mode, don't offer `wait_for_messages` to the model, or it may swallow your next message; the model keeps `send_message`.
3. Each message becomes a user turn; run the loop; send the final text answer back with `send_message`.
4. **Approvals by reply**: when the model wants to run `bash` (or a tool not on the allow list), send "Run this command? `<command>` Reply yes or no." and wait for the answer with `wait_for_messages`. Anything but yes is a no, and the model is told the user declined.
5. Keep the conversation across messages, as in week 2. A message like `/new` that starts a fresh conversation is a nice touch, not required.

## Step 5: test

From the phone, with approvals on:
- "What files are in the harness folder?": an approval request arrives, they reply yes, the answer arrives.
- Reply no to an approval: the agent should say it can't run the command, not try another way around it.
- A two-step task, e.g. "How much free disk space do I have, and which folder in my home directory is biggest?"
- **Somebody else gets ignored.** If a friend can message the bot, have them try. Otherwise set `TELEGRAM_CHAT_ID` to a wrong number for a moment and check that their own message is dropped and logged. Put it back.
- Stop the harness, send a message, start it again: the old message must not run.

Fix what breaks. Commit as you go.

## Step 6: publish and submit

1. Check what becomes public: `git status`, `git log --stat`, and `git grep -n -I -e "TELEGRAM_BOT_TOKEN=" -e "bot[0-9]\{8,\}:"` should show only placeholders. `.env` must not be tracked. **CHECKPOINT:** the student agrees, then push.
2. Update the harness's `README.md`: plugins, the Telegram mode, and the safety notes.
3. In their clone of this homework repository, on a branch `week-3-<user>` of their fork, copy the plugin folder to `week-3/submissions/<user>/` (the plugin's own files at that level, with its `README.md` linking to the harness repository and saying how to run Telegram mode).
4. Commit only that folder, push, and open the pull request with `gh pr create`. The description says which AI helped them, and how. **CHECKPOINT** before opening it.

The class's CI only accepts changes under `week-3/submissions/<their-username>/`.

## Before ending any session

Say where things stand and what's next, in two sentences. If the work isn't finished, write that note in the harness repository's `README.md` under a "Status" heading, so the next session can pick it up.

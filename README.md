# Hesy for AI agents

Hesy is a dictation app that cleans speech as it goes: [hesy.app](https://hesy.app). This repo holds its Claude Code plugin and the setup for other agents.

Hesy ships an MCP server, so Claude Code, Codex, Cursor and any other MCP client can work with it:

- **Modes**: turn Clean, Private, Prompt, Verbatim, Formal or a custom mode on and off; create, edit and delete custom modes (Hesy Pro).
- **Dictionary**: teach Hesy the terms of the project you're in ("add this repo's library and product names to my Hesy dictionary"), so dictation spells them right.
- **History**: read and search recent dictations.
- **Run text through Hesy**: clean, mask personal details (Private), make it Formal, or try a draft custom mode on sample text.
- **Transcribe a file** on this computer.
- **Voice answers** (`hesy_listen`, off until you turn it on in Hesy): the agent asks, you answer by voice, and it gets the cleaned text instead of Hesy pasting it.
- **Learning from your fixes**: with the hook below, when you correct a word in a dictation before sending it to the agent ("versel" → "Vercel"), Hesy adds that spelling to its dictionary, as when you fix it in History.

It needs a Hesy version with agent support (the release after 0.10.10). Everything stays on your computer. The agent runs `Hesy mcp`, which relays to the running app over 127.0.0.1 using a random token in a file only your user can read. If Hesy isn't running, it starts in the background.

Hesy's executable:

| OS | Path |
|---|---|
| macOS | `/Applications/Hesy.app/Contents/MacOS/Hesy` |
| Windows | `%LOCALAPPDATA%\Hesy\Hesy.exe` |
| Linux (.deb) | `/usr/bin/hesy` |
| Linux (AppImage) | wherever you keep it |

## Claude Code

As a plugin (MCP server plus the learning hook, macOS and Linux):

```
/plugin marketplace add hacksurvivor/hesy-agents
/plugin install hesy@hesy
```

The plugin's `bin/hesy-agent` finds Hesy in the places above. For an AppImage or a portable copy, set `HESY_BIN` to the executable.

Only the MCP server, on any OS:

```
claude mcp add --scope user hesy -- /Applications/Hesy.app/Contents/MacOS/Hesy mcp
```

## Codex

```
codex mcp add hesy -- /Applications/Hesy.app/Contents/MacOS/Hesy mcp
```

Learning from your fixes, in `~/.codex/hooks.json` (Codex asks you to trust a new hook once; review it with `/hooks`):

```json
{
  "hooks": {
    "UserPromptSubmit": [
      { "hooks": [{ "type": "command", "command": "/Applications/Hesy.app/Contents/MacOS/Hesy learn", "timeout": 5 }] }
    ]
  }
}
```

## Cursor, Windsurf, Claude Desktop and others

```json
{
  "mcpServers": {
    "hesy": { "command": "/Applications/Hesy.app/Contents/MacOS/Hesy", "args": ["mcp"] }
  }
}
```

## The hook, in detail

`Hesy learn` reads the hook's JSON from stdin (`{"prompt": "…"}`, as Claude Code and Codex send it), or plain text. It prints nothing and always exits 0, so it never blocks or changes a prompt. Hesy keeps the last few texts it pasted in memory for 15 minutes. When a sent message contains one of them, Hesy compares the two word by word. A replacement counts as a correction only when all of these hold:

- it is short (up to 3 words);
- the new spelling looks like a name or a term (a capital letter, a digit, or another script, as with «клод» → Claude);
- it sounds like what was heard;
- the old spelling isn't a term Hesy already knows;
- the message didn't rewrite more than three places.

A change of mind ("Friday" → "Monday") or a rewritten sentence teaches nothing. Learned spellings appear in the Dictionary with the "learned" label. The message itself is never stored or sent anywhere. To turn learning off, remove the hook.

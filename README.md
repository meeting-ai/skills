# Meeting.ai skills for AI agents

Meeting.ai is the AI that works before, during, and after your meeting. These skills teach
your AI agent how to use it. Ask your agent to join a call and take notes, find what was
decided, rewrite the notes, recap a week of meetings, or draw a one-page Visual Note.

## What you need

1. A Meeting.ai account. Create one at [meeting.ai](https://meeting.ai).
2. The Meeting.ai MCP server, connected to your agent. The skills tell the agent what to do.
   The server does the work. **Installing only the skills does not connect the server.**

| Server detail | Value |
|---|---|
| URL | `https://mcp.meeting.ai/mcp` |
| Transport | Streamable HTTP |
| Sign-in | Browser sign-in with your Meeting.ai account. No API key. |

## Install

### Hermes Agent

1. Install the package. It adds the skills and connects the server:

   ```bash
   hermes plugins install meeting-ai/skills --no-enable
   hermes plugins enable meeting-ai
   ```

2. Recommended: let Hermes do the sign-in. Add this entry to `~/.hermes/config.yaml`:

   ```yaml
   mcp_servers:
     meeting-ai:
       url: https://mcp.meeting.ai/mcp
       auth: oauth
       trust: untrusted
   ```

   Then run `hermes mcp login meeting-ai`, sign in on the Meeting.ai page that opens, and
   approve the connection. This entry replaces the package's own sign-in. Hermes stores and
   refreshes the sign-in itself, you do not need Node.js, and you have more time to sign in.
   Keep `trust: untrusted`, so that Hermes still asks before each tool that changes something.

3. Start a new session. If the tools do not appear, run `/reload-mcp`.

**Without step 2**, the package signs in through `mcp-remote`. You need Node.js 20.18.1 or
later with `npx` on your `PATH`. The first time the server starts, your browser opens the
Meeting.ai sign-in page. You have about 55 seconds to sign in and approve. If the time runs
out, start a new session to try again.

### Claude Code

Use the Claude plugin. It installs the skills and connects the server:

```
/plugin marketplace add meeting-ai/memo-claude-plugin
/plugin install meeting-ai@meeting-ai
```

Then run `/mcp`, select `meeting-ai`, and choose **Authenticate**.

### Other agents

First add the server to your agent. Most agents that support remote MCP servers use a
configuration like this:

```json
{
  "mcpServers": {
    "meeting-ai": {
      "url": "https://mcp.meeting.ai/mcp"
    }
  }
}
```

Then install the skills:

```bash
npx skills add meeting-ai/skills
```

Start a new session, sign in when your agent asks, and ask: "Which Meeting.ai workspace am I
connected to?"

## Skills

| Skill | Use it to |
|---|---|
| `meeting-ai-overview` | Find and read meetings, notes, and transcripts. Recap several meetings in the chat. Export, share, tag, and work with contacts, Drive files, images, and audio. |
| `meeting-ai-setup` | Connect the server, sign in, fix connection problems, and change workspace. |
| `meeting-ai-record-meeting` | Send the notetaker to Zoom, Google Meet, or Teams. Transcribe a Google Drive recording. |
| `meeting-ai-edit-notes` | Rewrite or rename the notes of an existing meeting. |
| `meeting-ai-visual-notes` | Create a one-page summary image from a meeting, several meetings, or any topic. |

## Before you install

- **Your data goes to Meeting.ai.** Every tool call goes to `mcp.meeting.ai` and acts on your
  Meeting.ai account, with your own role in each workspace. The server supplies the tools at
  runtime, so a pinned commit of this package does not cover them.
- **Some content goes to AI providers.** Image, audio, and Visual Note generation send the text
  you supply to third-party AI providers.
- **Meeting.ai can record tool usage.** The server can record each tool call for product
  analytics: the tool name, the outcome, and your user, workspace, connection, and client
  identifiers. This is separate from the package, and the package has no setting for it.
- **Some actions cost coins.** Recording, Visual Notes, and image and audio generation
  use your Meeting.ai subscription or coins. The agent tells you before it does
  one. If you have no active subscription or coins, the tool stops with an error.
- **Exports use Drive storage.** An export saves a file to your Drive and costs no coins. A
  storage-debt limit can refuse an export.
- **The notetaker is visible.** It joins a live call as a participant named Meeting.ai.
- **Share links can be public.** A share link shows the notes and, by default, the transcript.
  It has a PIN by default. The agent removes the PIN
  only if you ask.
- **Hermes asks first by default.** The Hermes package marks the server as untrusted. By
  default, Hermes asks you before each tool that the server does not mark read-only. An
  approval that you saved, a Hermes setting that turns off approvals, or your own configuration
  for a server with this name can change this.
- **Without the config entry, Hermes sign-in uses `mcp-remote`.** The package runs the pinned
  `mcp-remote@0.15.2` through `npx`, which downloads it from npm on first use. It opens your browser and a local callback
  port, and stores the sign-in tokens in `~/.mcp-auth`.
- **This package is configuration and instructions only.** The package itself has no code, no
  binary, no telemetry, and no self-updater.

To end the connection, revoke it in the Meeting.ai web app under Settings › MCP & CLI. Revocation
takes effect immediately.

## Links

- Website: https://meeting.ai
- Privacy policy: https://meeting.ai/privacy
- Terms of service: https://meeting.ai/terms
- Support: support@meeting.ai

## License

Apache License 2.0. See [LICENSE](LICENSE). The Meeting.ai name, logo, and artwork are trademarks
of Meeting.ai and are not covered by the license. See [NOTICE](NOTICE).

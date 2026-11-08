---
name: meeting-ai-setup
description: Use when Meeting.ai tools are not available in this session, the user wants to connect Meeting.ai, is asked to sign in or authorize, sees an authentication error from a meeting-ai tool, or asks how to switch the workspace the connection uses. Walks through adding the Meeting.ai MCP server, signing in, and checking that it works.
compatibility: Requires the Meeting.ai MCP server (https://mcp.meeting.ai/mcp)
---

# Connect Meeting.ai

The other Meeting.ai skills call tools from a hosted MCP server. Some installs add the server
for you. An install of the skills alone does not.

| Server detail | Value |
|---|---|
| URL | `https://mcp.meeting.ai/mcp` |
| Transport | Streamable HTTP |
| Sign-in | Browser sign-in with a Meeting.ai account. No API key. |

Tool names can carry a client prefix, for example `mcp__meeting-ai__workspace_list`.

## Check the connection first

If `workspace_list` is available, call it. A reply with at least one workspace means the
connection works. Tell the user which workspace is active and stop here.

If the tool is available but returns an authentication error, go to **Sign in**.

If the tool is not available, the server is not connected. Go to **Add the server**. Do not
guess or invent meeting data while the server is not connected.

## Add the server

Ask the user before you change their agent configuration. Use the steps for their agent.

**Claude Code**

```bash
claude mcp add --transport http meeting-ai https://mcp.meeting.ai/mcp
```

Then the user runs `/mcp`, selects `meeting-ai`, and chooses **Authenticate**.

**Hermes Agent**

```bash
hermes plugins install meeting-ai/skills --no-enable
hermes plugins enable meeting-ai
```

Then the user starts a new session. Hermes needs Node.js with `npx` for this package. If the
tools do not appear after sign-in, the user runs `/reload-mcp`.

**Cursor and other agents**

Add the server to the agent's MCP configuration. For Cursor that file is `~/.cursor/mcp.json`.
Many agents use this shape:

```json
{
  "mcpServers": {
    "meeting-ai": {
      "url": "https://mcp.meeting.ai/mcp"
    }
  }
}
```

If the agent uses a different format, use its documentation to add a remote HTTP MCP server
with the URL above.

Most agents load new servers only when a session starts. Tell the user to reload the agent or
start a new session if the tools do not appear.

## Sign in

1. A browser window opens on `auth.meeting.ai`. If the agent has a sign-in or authentication
   control for the server, the user chooses it first.
2. The user signs in with the same account they use for the Meeting.ai web or mobile app, then
   approves the connection. Never ask for a password, API key, or token in chat.
3. Call `workspace_list` and identify the active workspace. Installed does not mean
   authenticated. Do not say the account is connected until this call succeeds.

If the user has no Meeting.ai account yet, they can create one at https://meeting.ai. A new
account has no meetings, so suggest recording one with `meetings_create` or uploading a
recording in the web app before exploring.

## Workspace and access

Use `workspace_list` to show available workspaces. Use `workspace_set_active` only when the
user wants to change this connection's default. This changes only this connection and never
moves any data. Use `workspace_id` for a one-off call.

## Common problems

- **Authentication error on a tool call.** The session was revoked or expired. Ask the user to
  sign in again through the agent's reconnect control. Do not retry the tool until they have.
- **The sign-in never starts.** Report the connection error. Changing a password or buying a
  plan does not fix this.
- **The browser did not open.** Many agents print the authorization URL. The user can open it
  by hand on the same machine.
- **A tool returns an error about coins, balance, subscription, or storage.** Nothing is wrong
  with the connection. Explain the tool's message and stop. Do not keep retrying paid
  operations. The user manages their plan in the Meeting.ai web app.
- **Tools work but return no meetings.** Check the active workspace with `workspace_list`. The
  user may have meetings in a different workspace, or no meetings yet.
- **Disconnecting.** The user revokes the grant from Connected Apps in the Meeting.ai web app.
  Revocation takes effect immediately. Removing the server or plugin locally does not prove
  that the grant was revoked.

## Support

Website: https://meeting.ai
Privacy: https://meeting.ai/privacy
Terms: https://meeting.ai/terms
Support: support@meeting.ai

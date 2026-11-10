---
name: meeting-ai-overview
description: Use when the user asks what Meeting.ai can do, wants a recap of several meetings, or asks about their meetings, notes, transcripts, action items, contacts, Drive files, exports, tags, sharing, or workspaces in Meeting.ai. Covers the everyday reading tools plus export, media generation, contacts, Drive, and workspace switching. For recording a meeting, editing notes, or Visual Notes, the dedicated skills go deeper.
compatibility: Requires the Meeting.ai MCP server (https://mcp.meeting.ai/mcp)
---

# Meeting.ai overview

> **Requires the Meeting.ai MCP server.** This skill calls its tools. If `workspace_list` is not
> available in this session, stop and follow the `meeting-ai-setup` skill. Do not guess or invent
> results. Tool names can carry a client prefix, for example `mcp__meeting-ai__workspace_list`.

Meeting.ai records meetings and writes the notes. Through these tools the assistant finds,
reads, edits, exports, and shares those notes, and creates Visual Notes, images, and audio.
Decks, research, and spreadsheets are made in the Meeting.ai app, not through these tools.

Through this connection the assistant works on the user's meetings for them. When the user asks what
they can do, explain these in plain words:

- **Record a meeting and get notes.** Send the notetaker to Zoom, Google Meet, or Teams, or
  transcribe a Google Drive recording. See the `meeting-ai-record-meeting` skill.
- **Find and read meetings.** Search by keyword or date, read the notes, search or read the
  transcript.
- **Edit notes.** Rewrite or rename an existing meeting's notes. See the `meeting-ai-edit-notes`
  skill.
- **Recap several meetings.** Read several meetings and write one summary in the chat.
- **Visual Notes.** A one-page visual summary from a meeting or any topic. See the
  `meeting-ai-visual-notes` skill.
- **Export.** PDF or Markdown to the user's Drive.
- **Share and tag.** Public link with PIN, personal tags.
- **Contacts and Drive.** Who the user met, and the files in their workspace.
- **Generate media.** Images and spoken audio from text.

Some actions use the user's coins or subscription. Mention that in a few words before doing
one, so the user is never surprised.

Use the `meeting-ai-setup` skill for connection questions. The product page is
https://meeting.ai/mcp.

## Reading meetings

1. `meetings_search` with keywords, or with `start_date` / `end_date` (UTC) and no query to
   list by date. It returns the user's `role` on each meeting.
2. `meetings_notes` for one meeting: facts plus every notes section with its `id`. A meeting
   still running is not an error; its notes are live and provisional.
3. `meetings_transcript_search` to find where a phrase was said. `meetings_transcript` to
   read it all, following `next_cursor` until it is `null`.

Notes are a summary. For exact wording, numbers, or names, read the transcript.

## Recapping several meetings

This is the workflow for "summarize this week's meetings", "recap the Acme project", or
"what did we decide across the three planning sessions". It only reads meetings.

1. Find the meetings with `meetings_search`. Use a keyword for a project or client, or a
   `start_date` and `end_date` for a period. Confirm the list with the user.
2. Read each meeting with `meetings_notes`. For decisions, numbers, or quotes, check the
   transcript with `meetings_transcript_search`.
3. Write the recap: what happened, decisions made, open questions, and action items with
   owners, each item naming the meeting it came from.
4. Reply with the recap in the chat. That is the deliverable.
5. Then offer to turn it into a Visual Note with `recipe_visual_note`. See the
   `meeting-ai-visual-notes` skill.

Never write a recap into one of the source meetings. Each meeting's notes are the record of
that meeting.

## Exporting

`meetings_export` saves a file to Drive and returns a login-gated link. Formats: `pdf` (the
full Visual Notes and Summary document), `summary_only`, `transcript_only`,
`summary_and_transcript` (Markdown). PDF is built in the background: the first call returns
`processing`; call again with the same arguments until `ready`. Once `ready`, **do not call
again** for the same meeting and format, because each repeat creates a duplicate file. The
meeting must be finished.

## Sharing and tags

- `meetings_share` turns on the public link, PIN-protected by default. The link shows the
  notes and, by default, the transcript. Tell the user this before you share. When the reply
  has a `pin`, give the user the PIN with the link. Pass
  `require_pin: false` only on explicit request. Removing an existing PIN also requires
  `remove_existing_pin: true` and the owner's explicit request. Email sharing lives in the web app.
- `meetings_tag` and `drive_tag` apply tags that already exist. They never create tags. An
  unknown name returns the available tags. Tags are personal to the user.

## Contacts

`contacts_search` finds people by name, role, or company, with how many meetings they appear
in. `contacts_update` edits name, email, role, or company; a new name is confirmed across the
contact's meetings.

## Drive

`drive_search` lists or searches files with tags, size, owner, and an `etag`. Give the user
the file name, not the returned link: the file page is not live yet. `drive_rename`
renames a file the user owns; pass the `etag` as `if_match` so a concurrent change is refused
rather than overwritten. Uploading, deleting, and sharing files stay in the web app.

## Media

`media_image_gen` creates an image from a prompt; `media_audio_gen` reads text aloud into an
audio file. Both save to Drive, return a login-required link, spend the user's coins, and
create a new file every call. Confirm before repeating.

## Permissions

Editing notes, sharing, stopping a recording, and renaming Drive files are **owner only**.
Check `role` first and say so when the user is not the owner. Tagging works on any meeting
the user can see.

For a meeting that someone else owns, the owner's download settings decide whether its notes
or transcript can be exported. A refusal there is not a bug. Tell the user.

## Workspaces

Tools default to this connection's active workspace. `workspace_list` shows all of them;
`workspace_set_active` changes the default for this connection only. Pass `workspace_id` on
a single call for a one-off.

`media_image_gen`, `media_audio_gen`, and `recipe_visual_note` do not accept `workspace_id`.
They spend the user's coins and always save to the active workspace. Before you call one, check the active
workspace with `workspace_list`. If the content comes from a different workspace, ask the user
before you change the active workspace.

## Style

Use product words: workspace, meeting, notes, transcript, Drive, Visual Note. Give titles and
dates, not ids.

Read `code` and `retryable` on tool errors.
Retry the same call only when `retryable` is true, honoring `retry_after_seconds` when present.
For `OUTCOME_UNKNOWN` or `PARTIAL_WRITE`, explain what is known and stop; the operation may already have saved or charged.
Fix invalid input before making a different call. Do not loop. When a tool
says the account has no coins or no active subscription, relay the message and stop.

## Unsupported actions

Meeting.ai has no calendar scheduling, file deletion, or email delivery tool. Do not
claim that a Meeting.ai tool can do these. If the user explicitly wants another connected
service to perform them, use that service under its own permissions. Do not publish a
meeting link merely to work around an unsupported email request.

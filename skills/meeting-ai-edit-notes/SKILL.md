---
name: meeting-ai-edit-notes
description: Use when the user wants to edit, rewrite, tighten, or restructure the notes of an existing Meeting.ai meeting, or rename a meeting. Only the meeting owner can edit.
compatibility: Requires the Meeting.ai MCP server (https://mcp.meeting.ai/mcp)
---

# Edit meeting notes

> **Requires the Meeting.ai MCP server.** This skill calls its tools. If `workspace_list` is not
> available in this session, stop and follow the `meeting-ai-setup` skill. Do not guess or invent
> results. Tool names can carry a client prefix, for example `mcp__meeting-ai__workspace_list`.

Meeting.ai writes the first draft of every meeting's notes. Through the assistant the user can
improve a meeting's notes or rename the meeting. Edits are versioned like a web edit, so
nothing is lost.

## Rewriting one meeting's notes

1. Call `meetings_notes` to get the sections. Each section has an `id`, a `title`, and its
   current `content`. Check the user's `role`: editing is **owner only**. If the user is not
   the owner, say so and offer to draft the text for them to paste instead.
2. Read the transcript before rewriting. Notes are a summary, and rewriting a summary from a
   summary loses detail. Use `meetings_transcript_search` when the user names a topic, or
   page through `meetings_transcript` until `next_cursor` is `null` for a full rewrite.
3. Draft the new text and show it to the user. Confirm before saving.
4. Call `meetings_update_notes` with `sections: [{ id, text }]` for each section you are
   replacing. Pass `regenerate_visual_note: false` unless the user wants the meeting's
   visual note redrawn too, because redrawing spends coins.
5. To rename the meeting, pass `title` in the same call. A title-only change never redraws.

An unknown section id edits nothing and returns the valid ids. Re-read the sections and correct the request before trying again. A `PARTIAL_WRITE` or `OUTCOME_UNKNOWN` result is different: do not repeat the write, because it may already have saved or charged.

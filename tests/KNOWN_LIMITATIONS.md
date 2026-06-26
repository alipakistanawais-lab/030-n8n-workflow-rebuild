# Known Limitations

Honest list of what is partial or requires manual setup. Nothing here blocks the core company tests.

## 1. Video transcription is not wired to a provider (PARTIAL)
`WF_VIDEO_TRANSCRIPTION` performs the safety logic only:
- it locates the file (OneDrive path or email attachment),
- reads the real file size,
- fails gracefully if the file is over 25 MB,
- otherwise returns `Video transcription is not configured yet. / No action completed. / Verification status: Failed`.

To enable real transcription, add a speech-to-text step (e.g. OpenAI Whisper or Azure Speech) after
`02 Size Check And Transcription Status`: download the file binary, send it to the transcription API,
summarize the transcript with the OpenAI node, then return `response` with the summary, detected
language and `Verification status: Passed`. This needs an additional credential/secret that is out
of scope for this build.

## 2. Email pagination is bounded, not full 1000-email pagination (PARTIAL)
- `WF_EMAIL_LIST_CONTEXT` lists up to **50** emails per request (default 15). The requested count
  (e.g. 25) is honored and the main workflow splits long replies so nothing is cut at 16.
- True multi-page pagination over very large mailboxes (e.g. all 1000 emails via `@odata.nextLink`)
  is **not** implemented. Increase the `limit` cap in the `00 Prepare` node or add a `$skip` loop if
  you need deeper history.

## R1. Verify AI tool-node access to Telegram Input Mapper (LIVE TEST)
After importing into n8n, run the numbered email memory live test to confirm that AI tool node
expressions referencing `Telegram Input Mapper` resolve correctly during agent-tool runtime.
The 23 `Tool - *` nodes and the `Window Buffer Memory` use
`={{ $('Telegram Input Mapper').item.json.telegramChatId }}` to key per-chat memory.

If `telegramChatId` is empty inside tool execution, change the tool input mapping to pass
`telegramChatId` from the AI Agent input item directly (for example `={{ $json.telegramChatId }}`,
or inject the chat id into the agent prompt) instead of using the `$('Telegram Input Mapper')`
reference. Test command: `Do I have new emails?` then `What's in Email 2?` and confirm Email 2
resolves to the same email shown in the list.

## 3. Numbered email memory uses n8n workflow static data (CAVEAT)
- The numbered list is stored per `telegramChatId` in `$getWorkflowStaticData('global')` inside
  `WF_EMAIL_LIST_CONTEXT`.
- Static data persists across production executions but can reset when the workflow is re-imported,
  the n8n instance restarts, or in some queue/multi-main setups. If the saved list is gone, the bot
  asks the user to list emails again (by design). For a hard guarantee across restarts, swap the
  static-data store for an n8n Data Store / external DB in the `02 Store And Format List` and
  `03 Get Saved Email` nodes.

## 4. Sending attachments to Telegram has a size ceiling (PLATFORM LIMIT)
- Telegram bots can send documents up to ~50 MB. `WF_ATTACHMENT_SEND_TO_TELEGRAM` checks size first
  and, if too large, returns a graceful message and offers to save the file to OneDrive instead.

## 5. Draft-with-OneDrive-attachment direct-attach limit (GRAPH LIMIT)
- `WF_CREATE_OUTLOOK_DRAFT_WITH_ONEDRIVE_ATTACHMENT` attaches files inline via
  `POST /me/messages/{id}/attachments`, which Graph limits to ~3 MB. Files over 3 MB return a
  graceful "too large to attach directly" message (no upload session implemented).

## 6. Tool nodes must be re-linked after import (MANUAL STEP)
- n8n assigns new internal workflow IDs on import. Each of the 23 `Tool - ...` nodes in the main
  workflow must have its target sub-workflow re-selected after import (names are pre-filled).

## 7. Telegram credential is a placeholder (MANUAL STEP)
- No Telegram credential existed in the old workflows. Create one and select it on all Telegram
  nodes (main workflow + `WF_ATTACHMENT_SEND_TO_TELEGRAM`).

## 8. Contact resolution depends on Outlook People data (DATA-DEPENDENT)
- `WF_RESOLVE_CONTACT_EMAIL` resolves via `/me/people?$search=`. If the person is not in the
  mailbox's people/contacts, it cannot resolve them and will ask for the email address (it never
  invents one). The nickname map (Ali/Hidar/Bohdan) only normalizes names; it does not store emails.

## 9. Conflict detection is heuristic (BEHAVIOR)
- `WF_CHECK_CALENDAR_CONFLICTS` flags duplicates (same title+time+attendee) and overlaps within the
  requested window. It does not expand recurring-series exceptions beyond what `calendarView` returns.

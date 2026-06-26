# Expected Results

Detailed expected behavior for each company test.

| # | Expected result detail |
|---|---|
| 1 | Unauthorized chat ID → response body is the literal string `not available`. No emoji, no extra text, no branding. |
| 2 | Approved chat ID → short professional greeting; may offer help. No actions taken. |
| 3 | Calls email list tool (`action=list`), saves numbered memory, returns each unread/new email with sender, subject, received time, read status, attachments yes/no, plus a count. Ends with `Verification status: Passed`. |
| 4 | Same as #3, focused on unread; counts shown. |
| 5 | Identical behavior to #3 but the entire reply is in German. |
| 6 | Returns up to 15 emails. If the text exceeds Telegram limits it is split into multiple messages; numbered list items are never cut in the middle. |
| 7 | Calls memory `action=get` for number 2 → exact saved `message_id` → `Get Email Content`. Returns: `Email Nr. 2 found.` with Sender/Subject/Received/Attachments/Content/Link/Verification status. |
| 8 | Same flow for Email 1. |
| 9 | Same flow for Email 3, reply in German. |
| 10 | Resolves Email 2 from memory, lists attachments (name, content type, size, attachment_id) or states there are none. |
| 11 | Resolves Email 1 + attachment, downloads via Graph `$value`, sends via Telegram `sendDocument`. If > ~50MB or Telegram credential missing → graceful message + attachment details, `Verification status: Failed`. |
| 12 | `createReply` draft created with the comment text. Returns `Draft created: Yes`, recipient, subject, `Email sent: No`, `Verification status: Passed`. Nothing sent. |
| 13 | Contact resolved to a real email (or asks for email if not resolvable). Plain draft created. `Email sent: No`. |
| 14 | Outlook **Sent** folder remains empty for these tests; only **Drafts** are populated. |
| 15 | Contact resolved → optional conflict check → `POST /me/events` (Europe/Berlin). Returns title/date/time/attendees/link + `Verification status: Passed`. |
| 16 | Event created with empty attendees array. |
| 17 | Missing time/date → assistant asks exactly one clarification question; no event is created. |
| 18 | `search(q='Q2 report')` files only; shows top 3 with real `webUrl`; if >3 matches asks which one. |
| 19 | Lists OneDrive root children (files and folders). |
| 20 | `PUT .../content?@microsoft.graph.conflictBehavior=fail`. Saves `test text`. If the name exists it reports it was not overwritten (`Verification status: Failed`), otherwise returns the real link. |
| 21 | Lists open To Do tasks (status != completed) from the default (or named) list. Read-only. |
| 22 | Creates a task titled "call accountant" with tomorrow's due date in Europe/Berlin. |
| 23 | Name-only request → `Resolve Contact Email` → draft to resolved address; if ambiguous/unknown, asks for the email. |
| 24 | OneDrive video metadata fetched; size reported; since no transcription provider is configured → `Video transcription is not configured yet. / No action completed. / Verification status: Failed`. |
| 25 | Email attachment metadata fetched (requires message_id + attachment_id); same not-configured result. |
| 26 | If size > 25MB → `This video is over 25 MB ... cannot be transcribed automatically`, `Verification status: Failed`, no crash. |
| 27 | After a German message, all subsequent replies are in German until the user switches again. |
| 28 | After an English message, replies switch to English. |
| 29 | No tool is called; assistant asks a short clarification (e.g. "Could you clarify what you'd like me to do?"). |
| 30 | Same as #29. |

## General reply contract
- Action replies always end with a verification status line.
- Failures use the clean block: `Tool failed: Yes / Tool used: / Error message: / Action taken: / Verification status: Failed`.
- No reply contains n8n branding or attribution.

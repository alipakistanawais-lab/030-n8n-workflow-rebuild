# Company Test Checklist

Run these from Telegram against the bot. Unless noted, use an **approved** chat ID.
Mark each Pass/Fail. Expected outcomes are summarized in `EXPECTED_RESULTS.md`.

| # | Test command / action | Expected result |
|---|---|---|
| 1 | (Unauthorized account) `hello` | Reply is exactly `not available` |
| 2 | (CEO/approved account) `hello` | Normal short greeting/help reply |
| 3 | `Do I have new emails?` | Numbered list of new/unread emails (sender, subject, time, read, attachments) + verification status |
| 4 | `Show me my unread emails.` | Numbered unread list with counts |
| 5 | `Habe ich neue E-Mails?` | Reply in **German** with the email list |
| 6 | `List latest 15 unread emails.` | Up to 15 emails, split across messages if long, list never cut mid-item |
| 7 | `Show me the content of Email 2.` | Content of the saved Email 2 (uses saved message_id), formatted block |
| 8 | `What's in Email 1?` | Content of saved Email 1 |
| 9 | `Zeig mir Email 3.` | Content of saved Email 3, reply in German |
| 10 | `Does Email 2 have any attachments?` | Yes/No + real attachment names/sizes if any |
| 11 | `Send me the attachment from Email 1.` | Attachment delivered to Telegram, or graceful message if too large/none |
| 12 | `Reply to Email 1 and say I'll call them tomorrow.` | Outlook reply **draft** created (not sent), recipient/subject shown |
| 13 | `Send an email to <contact> saying the meeting is confirmed.` | Contact resolved, **draft** created, `Email sent: No` |
| 14 | Check Outlook **Sent** folder after tests 12–13 | Nothing sent; only drafts exist |
| 15 | `Create a meeting tomorrow at 3pm with <contact> about budget.` | Event created (Europe/Berlin) after contact resolve + conflict check |
| 16 | `Schedule a call Friday 10am, no attendees.` | Event created with no attendees |
| 17 | `Create a meeting with John` | Assistant asks for date/time (no event created) |
| 18 | `Search for Q2 report in OneDrive.` | Top matches with real webUrl links; asks which if many |
| 19 | `List files in my main folder.` | OneDrive root listing |
| 20 | `Save this as a report in OneDrive: test text` | File saved to OneDrive root, real link, no overwrite |
| 21 | `What's on my to-do list?` | Open Microsoft To Do tasks listed, read-only |
| 22 | `Add a task: call accountant tomorrow.` | Task created with Berlin due date |
| 23 | `Send a draft to <contact name>` | Contact auto-resolved; draft created |
| 24 | `Summarize the video <file> on OneDrive.` | Size checked; "not configured" message if no transcription provider |
| 25 | `Summarize the video attachment in Email X.` | Size checked from attachment; graceful/not-configured message |
| 26 | (Video > 25MB) `Summarize <big video>` | Graceful over-25MB explanation, no crash |
| 27 | Start English, then send a German message | Reply switches to German |
| 28 | Start German, then send an English message | Reply switches to English |
| 29 | `asdkjhaskjdh` | One short clarification question, no tool/action |
| 30 | `do the thing` | One short clarification question, no tool/action |

## First 10 commands to run (quick acceptance)
1. (unauthorized) `hello` → `not available`
2. (approved) `hello`
3. `Do I have new emails?`
4. `Show me my unread emails.`
5. `Habe ich neue E-Mails?`
6. `List latest 15 unread emails.`
7. `Show me the content of Email 2.`
8. `Does Email 2 have any attachments?`
9. `Reply to Email 1 and say I'll call them tomorrow.`
10. `Create a meeting tomorrow at 3pm with <contact> about budget.`

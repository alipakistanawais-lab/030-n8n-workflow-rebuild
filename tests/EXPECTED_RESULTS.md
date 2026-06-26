# Expected Results

- Unauthorized access returns exactly `not available`.
- Authorized basic greeting returns a short same-language assistant response.
- Email listing returns real sender, subject, received time, read status, attachment status, summary, count, and verification status.
- Email numbered follow-ups use saved `message_id` for the same `telegramChatId`.
- Draft tests create Outlook drafts only; Sent folder remains unchanged.
- Calendar creation requires date and time, checks conflicts, and returns real event details/link when Graph succeeds.
- OneDrive searches and saves return real `webUrl` links from Graph.
- To Do create/list returns task/list details from Microsoft Graph.
- Contact resolver asks for an email address until real company emails are configured.
- Video transcription returns a graceful failed status until transcription service setup is added; over-25MB videos fail gracefully.
- Vague commands ask one short clarification and do not call Microsoft tools.

# Safety Rules

- Only approved Telegram chat IDs can use the bot.
- Unauthorized users receive exactly: `not available`.
- Emails are never sent automatically. Outlook email actions create drafts only.
- The Microsoft Graph mail send endpoint must not be used.
- Emails are never deleted.
- Emails are never moved unless the user explicitly approves the exact email and action.
- OneDrive files are not overwritten unless the user explicitly says overwrite or replace.
- Contact emails, message IDs, attachment IDs, file IDs, folder IDs, dates, and times are never invented.
- Missing required details produce one short clarification question.
- Vague or garbage input must not call Microsoft tools.
- Tool success must include `Verification status: Passed`.
- Tool failure must include Tool failed, Tool used, Error message, Action taken, and `Verification status: Failed`.
- Replies follow the latest user-message language.

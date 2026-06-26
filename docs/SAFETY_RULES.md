# Safety Rules

These rules are enforced by the AI Agent system prompt and by the sub-workflow code.

## Email
- **Never send email.** Outlook actions create **drafts only**.
- The system never calls the Graph send operation (no `/sendMail`, no `Mail.Send`).
  Replies use `createReply` / `createReplyAll` which create draft messages, and new mails use
  `POST /me/messages` (draft). Nothing ever lands in the Sent folder.
- Never delete emails.
- Never move emails unless the user explicitly approves the exact email and action.
  (No move/junk/delete tool is included in this build.)

## OneDrive
- Never overwrite files. All uploads use `@microsoft.graph.conflictBehavior=fail`, so existing
  files are skipped, not replaced, unless the user explicitly asks to overwrite/replace.
- Duplicate attachments are skipped.
- Only real Microsoft Graph `webUrl` links are returned — never invented links.

## No invention
The assistant never invents:
- contact email addresses,
- message IDs, attachment IDs, file IDs, folder IDs,
- missing dates or times.

If a required detail is missing, the assistant asks **one** short clarification question.

## Verification status
- Every completed action ends with `Verification status: Passed`.
- Every failed action returns a clean block:
  ```
  Tool failed: Yes
  Tool used: <sub-workflow>
  Error message: <clean message>
  Action taken: <what happened>
  Verification status: Failed
  ```
- The assistant never claims success unless the tool result confirms it.

## Access control
- Only approved Telegram chat IDs may use the bot.
- Unauthorized users receive exactly `not available` — no extra text, no explanation, no emoji,
  no branding.
- Approved IDs are configured in the `Telegram Input Mapper` Code node or via the
  `ALLOWED_TELEGRAM_CHAT_IDS` environment variable.

## Contacts
- Strict resolution. Ali (Kabalan), Hidar (Aliasghari) and Bohdan (Dyakunchak) are different
  people and are never mixed. Low-confidence matches are rejected and the assistant asks for the
  email address instead.

## Vague input
- Garbage or unclear input (e.g. `asdkjhaskjdh`, `do the thing`) triggers **no tool call** and
  **no action** — only one short clarification question.

## Video
- File size is checked first. Videos over 25 MB fail gracefully with a plain explanation.
- If no transcription provider is configured, the assistant returns:
  ```
  Video transcription is not configured yet.
  No action completed.
  Verification status: Failed
  ```

## Telegram output hygiene
- n8n attribution is disabled on every Telegram node (`appendAttribution: false`).
- The Clean Final Reply Formatter removes any leaked branding, leading `=`, `undefined`/`null`,
  and debug JSON before sending.

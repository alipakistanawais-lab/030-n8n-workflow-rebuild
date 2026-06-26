# Setup Guide — 030 Office Workflow AI Assistant

Project: **030 Office Workflow AI Assistant**
Main workflow: **PROD_030_TELEGRAM_OFFICE_AI_ASSISTANT_CLEAN**

This guide explains how to import and configure the clean workflow system in n8n. It does not
connect to n8n for you — import is manual until you explicitly approve applying to n8n.

## 0. Prerequisites
- An n8n instance (self-hosted or cloud) with the LangChain (AI) nodes available.
- A Telegram bot token (from @BotFather).
- The existing Microsoft Graph OAuth2 credential `Prod_MsGraph_OAuth2_Hidar` (or an equivalent
  generic OAuth2 credential with the scopes listed in `CREDENTIALS_TO_SELECT.md`).
- The OpenAI credential `OpenAI - 030 Office Workflow`.

## 1. Create the Telegram credential (only missing credential)
1. In n8n: Credentials → New → **Telegram API**.
2. Paste your bot token.
3. Name it `030_Telegram_Bot` (any name is fine; you select it per node after import).

> The old workflows had no Telegram credential, so the JSON ships with a placeholder
> (`REPLACE_TELEGRAM_CRED` / `030_Telegram_Bot (SELECT AFTER IMPORT)`). You must select your
> real Telegram credential on the Telegram nodes after import.

## 2. Import order (IMPORTANT)
1. **Import all 23 sub-workflows first** (every `WF_*.json` file in `workflows/`).
2. In each sub-workflow, **select the Microsoft Graph credential** (`Prod_MsGraph_OAuth2_Hidar`)
   on every HTTP Request node. In `WF_ATTACHMENT_SEND_TO_TELEGRAM`, also select the Telegram
   credential on `04 Telegram Send Document`.
3. Keep sub-workflows **inactive**. They use an Execute Workflow Trigger and are invoked by the
   main workflow as tools; they do not need to be "active" to be callable as sub-workflows.
4. **Import the main workflow last**: `PROD_030_TELEGRAM_OFFICE_AI_ASSISTANT_CLEAN.json`.
5. Select the **Telegram** credential on `Telegram Trigger`, `Access Denied Reply`,
   `Telegram Reply`, `Telegram Error Reply`.
6. Select the **OpenAI** credential on `OpenAI Chat Model`.
7. Open each of the 23 `Tool - ...` nodes and **re-select the target sub-workflow** from the list
   (n8n assigns new internal workflow IDs on import; the sub-workflow name is pre-filled to help).
8. Keep the main workflow **inactive** for now.
9. If you reuse the same Telegram bot as an old production workflow, **deactivate the old
   workflow first** (a Telegram bot can only have one active webhook receiver).
10. **Activate the main workflow only after the tests pass.**

## 3. Configure access control (no secrets needed)
Open the `Telegram Input Mapper` Code node in the main workflow and set the approved chat IDs:

```js
const ALLOWED_TELEGRAM_CHAT_IDS = ['REPLACE_WITH_APPROVED_CHAT_ID'];
```

To find your numeric chat ID, message your bot and check the Telegram Trigger execution data, or
use @userinfobot. You can also set an environment variable instead of editing the node:

```
ALLOWED_TELEGRAM_CHAT_IDS=111111111,222222222
```

## 4. Microsoft Graph permissions
Ensure the generic OAuth2 credential has these delegated scopes consented in Entra ID:
`Mail.ReadWrite`, `Mail.Read`, `Calendars.ReadWrite`, `Files.ReadWrite`, `Tasks.ReadWrite`,
`Contacts.Read`, `People.Read`, `User.Read`, `offline_access`.
Do **not** add `Mail.Send` — this system never sends email.

## 5. Smoke test
1. Activate the main workflow.
2. From an **approved** Telegram account, send: `Do I have new emails?`
3. From an **unapproved** account, send: `hello` → you must receive exactly `not available`.

See `tests/COMPANY_TEST_CHECKLIST.md` for the full test plan and the first 10 commands.

## 6. Optional: enable video transcription
`WF_VIDEO_TRANSCRIPTION` checks file size and reports "not configured" because no transcription
provider is wired up. To enable it, add an OpenAI Whisper / Azure Speech HTTP node after the size
check and feed its result into the final Code node. See `tests/KNOWN_LIMITATIONS.md`.

## Troubleshooting
- **"not available" for an approved user** → the chat ID is not in `ALLOWED_TELEGRAM_CHAT_IDS`
  (numbers must match exactly, as strings).
- **Tool returns a credential error** → the HTTP node in that sub-workflow has no Microsoft Graph
  credential selected, or the scope is missing.
- **A `Tool - ...` node errors with "workflow not found"** → re-select the sub-workflow in that
  tool node (step 2.7).
- **Numbered email follow-up says the list expired** → ask the bot to list emails again; static
  data resets if the sub-workflow is re-imported or the instance restarts (see KNOWN_LIMITATIONS).

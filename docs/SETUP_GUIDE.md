# Setup Guide - 030 Office Workflow AI Assistant

This package is a clean rebuild. It does not edit or depend on legacy workflow exports.

## Import order

1. Import all sub-workflows first.
2. Select credentials in each sub-workflow.
3. Keep all sub-workflows inactive unless they use Execute Workflow Trigger and need activation in your n8n version.
4. Import main workflow last.
5. Select Telegram credential.
6. Select OpenAI credential.
7. Select Microsoft credentials.
8. Keep main workflow inactive first.
9. Deactivate old production workflow before activating new Telegram workflow if using the same bot.
10. Activate new workflow only after tests pass.

## Required manual setup

- Set environment variable `TELEGRAM_ALLOWED_CHAT_IDS` to comma-separated approved Telegram chat IDs.
- Select the Telegram bot credential on `Telegram Trigger`, `Access Denied Reply`, and `Telegram Reply`.
- Confirm `OpenAI - 030 Office Workflow` on the OpenAI Chat Model.
- Confirm `Prod_MsGraph_OAuth2_Hidar` on every Microsoft Graph HTTP Request node.
- In the main workflow, verify each toolWorkflow node points to the imported workflow with the same name. If n8n assigns new workflow IDs on import, reselect the workflow by name.
- Add real company contact emails in `WF_RESOLVE_CONTACT_EMAIL` before using named-contact drafting or meetings.
- Configure a default Microsoft To Do list ID in `WF_CREATE_TODO_TASK` if `tasks` is not accepted in your tenant.

## Activation safety

Do not activate this workflow until credentials, allowed chat IDs, and the first checklist tests pass. The old production Telegram workflow must be inactive when using the same Telegram bot webhook.

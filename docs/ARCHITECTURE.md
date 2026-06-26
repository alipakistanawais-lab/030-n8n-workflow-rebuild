# Architecture

Main workflow: `PROD_030_TELEGRAM_OFFICE_AI_ASSISTANT_CLEAN`

Flow:

Telegram Trigger -> Telegram Input Mapper -> Access Control Check -> Access Denied Reply or Microsoft 365 Tool Layer -> AI Agent Command Center -> Error Handler -> Clean Final Reply Formatter -> Telegram Long Reply Splitter -> Telegram Reply -> Execution Logger

The Microsoft 365 Tool Layer is implemented as connected AI toolWorkflow nodes. Each tool calls one imported sub-workflow and each sub-workflow returns a `response` field.

## State

`WF_EMAIL_LIST_CONTEXT` stores numbered email context per `telegramChatId` in workflow static data. Follow-up requests use `action=get` and the user-provided `emailNumber` to retrieve the saved exact `message_id`.

## Credential strategy

Microsoft Graph operations use generic OAuth2 credential `Prod_MsGraph_OAuth2_Hidar`. The main AI model uses `OpenAI - 030 Office Workflow`. Telegram uses a placeholder credential because no Telegram credential was found in the reference exports.

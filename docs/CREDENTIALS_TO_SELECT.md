# Credentials To Select After Import

All Microsoft Graph calls in this system use **one generic OAuth2 credential** so you
only have to pick a single Microsoft credential per workflow. The credential names and
IDs below were reused from your old exported workflows wherever possible.

## Reused credentials (from old workflows)

| Purpose | n8n credential type | Credential name | Credential ID | Source old workflow |
|---|---|---|---|---|
| All Microsoft Graph HTTP calls (Outlook, Calendar, OneDrive, To Do) | `oAuth2Api` (Generic OAuth2 API) | `Prod_MsGraph_OAuth2_Hidar` | `p8l1D6snXEvhc1aR` | WF_02, WF_05, WF_15, WF_16, PROD_WF_03/04 |
| OpenAI chat model | `openAiApi` | `OpenAI - 030 Office Workflow` | `SYzX1DYo7yj2AaNa` | WF_02 |

### Other Microsoft credentials seen in old workflows (not required by this build)
These are listed only for reference. This clean build does **not** use the dedicated
Outlook/OneDrive/To-Do node credentials — it uses the single generic Graph OAuth2 above.

| Old credential | Type | ID |
|---|---|---|
| `Test030_Microsoft Outlook_Hidar` | microsoftOutlookOAuth2Api | `x8yxuwXYoLmjEdeU` |
| `Test_Prod_Microsoft Outlook account` | microsoftOutlookOAuth2Api | `IHfxlcXVG9MduJdl` |
| `Test030_Microsoft Calendar_Hidar` | microsoftOutlookOAuth2Api | `swSmJVxrfP6k5xl5` |
| `Test030_Microsoft Drive_Hidar` | microsoftOneDriveOAuth2Api | `I6tD9ohLQHHm7i8j` |
| `Test030_Microsoft To Do_Hidar` | microsoftToDoOAuth2Api | `lD1QkioAEo2FfA9Q` |
| `Microsoft account` | microsoftOAuth2Api | `7luCKKhqQdeSgY9M` |

> The generic Graph OAuth2 credential must have these delegated scopes granted in Entra ID:
> `Mail.ReadWrite`, `Mail.Read`, `Calendars.ReadWrite`, `Files.ReadWrite`, `Tasks.ReadWrite`,
> `Contacts.Read`, `People.Read`, `offline_access`, `User.Read`.
> It must **not** require `Mail.Send` — this system never sends email.

## Missing credential — must be created (placeholder used)

| Purpose | n8n credential type | Placeholder name used in JSON | Placeholder ID |
|---|---|---|---|
| Telegram bot | `telegramApi` | `030_Telegram_Bot (SELECT AFTER IMPORT)` | `REPLACE_TELEGRAM_CRED` |

No Telegram credential existed in the old workflows (they used the n8n Chat trigger, not
Telegram). You must create a Telegram API credential with your bot token and select it in:

- Main workflow: `Telegram Trigger`, `Access Denied Reply`, `Telegram Reply`, `Telegram Error Reply`
- Sub-workflow `WF_ATTACHMENT_SEND_TO_TELEGRAM`: `04 Telegram Send Document`

## Nodes that need manual credential selection after import

### Main workflow `PROD_030_TELEGRAM_OFFICE_AI_ASSISTANT_CLEAN`
- `OpenAI Chat Model` → select `OpenAI - 030 Office Workflow`
- `Telegram Trigger`, `Access Denied Reply`, `Telegram Reply`, `Telegram Error Reply` → select the Telegram credential
- The 23 `Tool - ...` nodes are **Execute (sub) Workflow** tools. After importing the sub-workflows,
  open each `Tool - ...` node and re-select the target sub-workflow from the list
  (the workflow names are pre-filled as `cachedResultName`, but n8n assigns new workflow IDs on import).

### Every sub-workflow `WF_*`
- Each HTTP Request node → select `Prod_MsGraph_OAuth2_Hidar` (Generic OAuth2 API).
- `WF_ATTACHMENT_SEND_TO_TELEGRAM` also needs the Telegram credential on `04 Telegram Send Document`.

## Access control configuration (no secret)
Open the `Telegram Input Mapper` Code node in the main workflow and set the approved chat IDs:

```js
const ALLOWED_TELEGRAM_CHAT_IDS = ['REPLACE_WITH_APPROVED_CHAT_ID'];
```

Or set the environment variable `ALLOWED_TELEGRAM_CHAT_IDS="111111111,222222222"` to override
without editing the node.

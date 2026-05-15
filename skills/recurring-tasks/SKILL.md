---
name: recurring-tasks
description: >
  Recurring admin workflows for nonprofit operations — Canada Helps disbursement processing
  and Benevity disbursement processing. This skill should be used when the operator says
  "process the Canada Helps file", "process the Benevity file", "disbursement",
  "CharityDataDownload", "DonationReport", or provides a disbursement CSV to process.
version: 0.1.0
---

# Recurring Tasks — Disbursement Processing

Repeatable admin and operational workflows for processing donation disbursement files from third-party giving platforms into Salesforce (NPSP).

## Available Workflows

| Task | Reference | Frequency |
|---|---|---|
| Canada Helps Disbursement | `${CLAUDE_PLUGIN_ROOT}/skills/recurring-tasks/references/canada-helps-processing.md` | Every 1-2 weeks |
| Benevity Disbursement | `${CLAUDE_PLUGIN_ROOT}/skills/recurring-tasks/references/benevity-processing.md` | When files arrive |

For Salesforce field names and IDs specific to your org, see `CONFIGURATION.md` at the repo root.

## Auto-Detection

When the operator provides a file without specifying the source, detect from the file:

| Pattern | Workflow |
|---|---|
| Filename starts with `CharityDataDownload` | Canada Helps |
| Filename starts with `DonationReport_` | Benevity |
| Row 1 contains `Donations Report` | Benevity |
| Row 1 headers start with `TRANSACTION NUMBER` | Canada Helps |

If uncertain, ask the operator.

## Shared Reference IDs

These placeholders should be filled in with your org's actual values. See `CONFIGURATION.md` for how to find each.

| Item | ID |
|---|---|
| Fundraising Opportunity Record Type | `<RECORD_TYPE_ID_FUNDRAISING>` |
| Anonymous Contact | `<CONTACT_ID_ANONYMOUS>` |
| Default Campaign (No Campaign) | `<CAMPAIGN_ID_DEFAULT>` |
| Default Fund | `Operating Fund` |
| Slack: Bookkeeping recipient | `<SLACK_USER_ID_BOOKKEEPER>` |
| Google Drive Folder | `<GDRIVE_FOLDER_ID>` |

## Common Process Flow

Both workflows follow this general pattern:

1. **Read and parse** the CSV — present summary to the operator for confirmation
2. **For each row:** determine if anonymous → match or create Contact → create Fundraising Opportunity
3. **Report progress** — summary of what was created
4. **Append columns** — add OPPORTUNITY ID and FUND columns to CSV
5. **Rename and save** the processed file
6. **Send to bookkeeping** via Slack (or flag for manual send)
7. **Upload to Google Drive** (or flag for manual upload)

## Tools Needed

- Salesforce MCP (Admin endpoint — read + write)
- Slack (for sending to bookkeeping — manual fallback if unavailable)
- Google Drive (for upload — manual fallback if unavailable)

## Fallback Policy

If a tool isn't available:
1. Complete all steps that ARE possible
2. Save outputs locally
3. Tell the operator exactly what manual steps remain

## Session Limits

Max 20 Opportunities per session. If the file has more than 20 rows, process in batches.

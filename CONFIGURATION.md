# Configuration

This skill uses placeholder values that must be replaced with your own Salesforce, Slack, and Google Drive identifiers before running. Below is every placeholder used across the skill files, what it represents, and how to find the value in your own systems.

## Placeholder Reference

| Placeholder | What it is | How to find it | Example shape |
|---|---|---|---|
| `<RECORD_TYPE_ID_FUNDRAISING>` | Salesforce Record Type ID for the "Fundraising" Opportunity record type (NPSP). Used when creating new Opportunities from disbursement rows. | Setup → Object Manager → Opportunity → Record Types → click your Fundraising record type. The URL contains the 18-char ID starting with `012`. | `012XXXXXXXXXXXXXXX` |
| `<CONTACT_ID_ANONYMOUS>` | Salesforce Contact ID for the catch-all "Anonymous Donor" Contact. Used when a donor is anonymous (Canada Helps "Anon" / Benevity "Not shared by donor"). | Create or locate a single "Anonymous Donor" Contact in Salesforce. Open the Contact record; the 18-char ID in the URL starts with `003`. | `003XXXXXXXXXXXXXXX` |
| `<CAMPAIGN_ID_DEFAULT>` | Salesforce Campaign ID used as the default Campaign attribution for disbursement Opportunities when no campaign is specified. Often a "No Campaign / General" placeholder Campaign. | Create or locate a default Campaign in Salesforce. Open it; the 18-char ID in the URL starts with `701`. | `701XXXXXXXXXXXXXXX` |
| `<SLACK_USER_ID_BOOKKEEPER>` | Slack user ID for the bookkeeper / finance contact who receives the processed disbursement file. | In Slack, click the user's profile → three-dot menu → "Copy member ID". Starts with `U`. | `U0XXXXXXXXX` |
| `<GDRIVE_FOLDER_ID>` | Google Drive folder ID where processed disbursement files are archived. | Open the folder in Google Drive. The URL is `https://drive.google.com/drive/folders/<ID>` — copy the part after `/folders/`. | `1xXXXXXXXXXXXXXXXXXXXXXXXXXX` |

## Custom Field Names

The skill references these Salesforce custom fields. If your org uses different API names, update the references in `skills/recurring-tasks/references/canada-helps-processing.md` and `skills/recurring-tasks/references/benevity-processing.md`.

| Field API name (as used in skill) | What it stores |
|---|---|
| `Opportunity_ID__c` | A human-readable Opportunity reference number written back to the CSV after creation. |
| `Fund__c` | The internal fund / budget bucket the donation is attributed to (e.g. `Operating Fund`). |

## Fund Values

The skill uses these fund codes by default:

| Fund code | When applied |
|---|---|
| `Operating Fund` | Default for all donations. |
| `CORPORATE_GRANT` | Example value applied when a Benevity row has Amount = exactly $1,500.00. Replace with your own fund code (or remove the rule) to match how your org categorizes corporate matching programs. |

## Once Configured

After filling in your values, you can either:

1. **Hardcode them** by editing the skill files directly and replacing every `<PLACEHOLDER>` with your real value.
2. **Keep the placeholders** in the skill files and instead provide the values in a session note or org-level CLAUDE.md so Claude substitutes them at runtime.

Option 2 is preferable if you plan to share the repo or version-control it.

# claude-cowork-nonprofit-workflow

An example Claude Cowork "skill" that processes Canada Helps and Benevity donation disbursement CSVs into Salesforce (NPSP). Published as an anonymized showcase — fork it, adapt the placeholders to your own org, and you have a working disbursement-processing skill.

## What this is

This repo is a single Claude Code skill — `recurring-tasks` — packaged the way a Claude Cowork plugin would ship it. Hand Claude a Canada Helps or Benevity CSV and it will: parse the file, match or create Contacts in Salesforce, create Fundraising Opportunities, write the Opportunity IDs back into the CSV, rename and save the processed file, DM the bookkeeper in Slack, and upload to a Google Drive archive folder.

It is a real workflow used at an unnamed Canadian charity, scrubbed of org-specific identifiers and contributed back as a reference example.

## Why it's interesting (as a Claude Code pattern)

A few design choices worth calling out for anyone learning how to write Claude Code / Cowork skills:

- **Auto-detection of file source from filename and header patterns.** The skill doesn't ask the user to specify "is this Canada Helps or Benevity?" — it inspects the filename prefix (`CharityDataDownload` vs `DonationReport_`) and the row 1 header pattern, and routes to the right playbook automatically. Falls back to asking only when uncertain.
- **"Confirm before mutate" UX.** Before creating a single Salesforce record, the skill parses the file end-to-end and presents a summary (donation count, totals, anonymous count, fund breakdown) for the operator to confirm. Nothing writes to Salesforce until that confirmation lands. This is a generally useful pattern for any skill that performs irreversible mutations.
- **Graceful tool fallbacks.** Slack and Google Drive are listed as "optional" tools. If they're not connected, the skill completes everything it *can* do (Salesforce writes, CSV append, local save) and then tells the operator the exact manual steps remaining — instead of failing the whole run.
- **Session row limits.** Hard cap of 20 Opportunities per session so each batch stays reviewable. Bigger files get processed in explicit batches rather than silently chewing through 200 rows.
- **Split SKILL.md (router) + references/ (detailed playbooks).** The top-level `SKILL.md` is short — it's a router that lists the available workflows, the auto-detection rules, and shared IDs. The detailed step-by-step playbooks live in `references/` and are only loaded into context when a matching workflow fires. This keeps Claude's working context lean and lets you grow the skill (add more platforms) without bloating the loaded prompt.

## What it does

For each disbursement CSV the operator hands it, the skill will:

- Auto-detect Canada Helps vs Benevity from filename and headers
- Parse the CSV and present a summary for confirmation (counts, totals, anonymous donors, companies, fund assignments)
- For each row: detect anonymous donors, match existing Salesforce Contacts (by email → name → postal code priority for Benevity), or create new ones (with NPSP Household Account auto-creation)
- Update Contact addresses when Salesforce is blank, flag when both sides have different values
- Create a Fundraising Opportunity per donation with appropriate Record Type, Campaign, Fund, Stage, Amount, and Close Date
- Append `OPPORTUNITY ID` and `FUND` columns back into the CSV (skipping metadata and totals rows)
- Rename the file using disbursement date / disbursement ID / period ending conventions
- DM the bookkeeper in Slack with a short message and the file attached
- Upload the processed file to a designated Google Drive folder
- Tell you exactly what's left to do manually if any optional tool was unavailable

## How to use

1. Install this skill into a Claude Cowork-enabled Claude Code session (drop the `skills/recurring-tasks/` folder into your plugin's `skills/` directory, or load it as a standalone skill).
2. Fill in the placeholders in `CONFIGURATION.md` with your org's Salesforce, Slack, and Google Drive identifiers.
3. Hand Claude a Canada Helps or Benevity disbursement CSV and ask it to "process the disbursement file" (or "process the Canada Helps file" / "process the Benevity file"). The skill will trigger from those phrasings.

## Adapting for your org

See [CONFIGURATION.md](./CONFIGURATION.md) for the full list of placeholder values to fill in (Salesforce Record Type ID, Anonymous Contact ID, default Campaign ID, bookkeeper Slack user ID, archive Google Drive folder ID), how to find each, and the example fund-code rule you may want to adjust.

## Limits and assumptions

- **Salesforce + NPSP** assumed. The Opportunity / Contact / Campaign / Account / Fund model and the `Opportunity_ID__c` and `Fund__c` custom field names are NPSP-flavored.
- **CSV formats** match Canada Helps (CharityDataDownload) and Benevity (Donations Report) as of 2026-Q2. If either platform changes their export schema, the column mappings in `references/` will need an update.
- **20 rows per session cap.** Larger files are processed in batches by re-running.
- **Salesforce MCP required.** Slack and Google Drive are nice-to-have with manual fallbacks; Salesforce is non-optional (the whole point of the skill is creating SF records).

## Author

Published by [`nocodenonprofit`](https://github.com/nocodenonprofit) as an anonymized showcase. The original was built for an unnamed Canadian charity using Claude Code + Claude Cowork. Feedback and forks welcome.

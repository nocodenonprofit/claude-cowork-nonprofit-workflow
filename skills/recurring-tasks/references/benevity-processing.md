# Benevity Disbursement Processing

Process a Benevity Donations Report CSV: match or create Contacts in Salesforce, create Fundraising Opportunities, append Opportunity ID/Fund columns, send to Bookkeeping via Slack, upload to Google Drive.

## CSV Format

### File Structure
- **Rows 1-11:** Report metadata (charity name, ID, period, disbursement ID)
- **Row 5:** Period Ending (for file naming)
- **Row 8:** Disbursement ID (for file naming)
- **Row 12:** Column headers
- **Row 13+:** Individual donation rows
- **"Totals" row:** Stop processing here
- **After Totals:** Summary rows

### Donation Columns (Row 12 headers)

| Col | Header | Maps To |
|---|---|---|
| A | Company | Reference only (employer) |
| B | Project | Reference only |
| C | Donation Date | Opportunity.CloseDate (extract YYYY-MM-DD from ISO datetime) |
| D | Donor First Name | Contact.FirstName |
| E | Donor Last Name | Contact.LastName |
| F | Email | Contact.Email |
| G | Address | Contact.MailingStreet |
| H | City | Contact.MailingCity |
| I | State/Province | Contact.MailingState |
| J | Postal Code | Contact.MailingPostalCode |
| Q | Source | Reference (Payroll, Corporate Grant, etc.) |
| S | Total Donation to be Acknowledged | Opportunity.Amount |

**Appended columns (donation rows only, NOT metadata/totals):** OPPORTUNITY ID, FUND

## Fund Rules

| Condition | Fund Value |
|---|---|
| Default | `Operating Fund` |
| Amount = exactly $1,500.00 | `CORPORATE_GRANT` |

(The `CORPORATE_GRANT` value is an example of a custom fund code your org may use for a specific corporate matching program. Adjust the trigger amount and code to match your own fund taxonomy.)

## Step-by-Step

### Step 1: Parse CSV
1. Read CSV, extract metadata (Period Ending from row 5, Disbursement ID from row 8)
2. Identify donation rows: start at row 13, stop before "Totals"
3. Present summary: disbursement ID, period, donation count, total, anonymous count, companies, fund assignments
4. Ask the operator to confirm

### Step 2: Process Each Row

**2a. Determine if Anonymous:**
If ALL of First Name, Last Name, Email = "Not shared by donor" → use Anonymous Contact `<CONTACT_ID_ANONYMOUS>`. Query to get AccountId. Skip to 2e.

**2b. Match Existing Contact:**

Priority 1 — Match by email (if not "Not shared by donor"):
```sql
SELECT Id, FirstName, LastName, Email, AccountId, MailingStreet, MailingCity, MailingState, MailingPostalCode
FROM Contact WHERE Email = '[email]' LIMIT 5
```

Priority 2 — Match by name:
```sql
SELECT Id, FirstName, LastName, Email, AccountId, MailingStreet, MailingCity, MailingState, MailingPostalCode
FROM Contact WHERE FirstName = '[first]' AND LastName = '[last]' LIMIT 5
```

Priority 3 — Match by postal code:
```sql
SELECT Id, FirstName, LastName, Email, AccountId, MailingPostalCode
FROM Contact WHERE MailingPostalCode = '[postal]' LIMIT 10
```

If all identifiers are "Not shared by donor" → treat as anonymous.

**2c. Create New Contact (if no match):**
```json
{
  "FirstName": "[name]", "LastName": "[name]",
  "Email": "[omit if 'Not shared by donor']",
  "MailingStreet": "[omit if 'Not shared by donor']",
  "MailingCity": "[omit if 'Not shared by donor']",
  "MailingState": "[omit if 'Not shared by donor']",
  "MailingPostalCode": "[omit if 'Not shared by donor']"
}
```

**2d. Update Contact Address (if needed):**
Same as Canada Helps — compare and update or flag.

**2e. Create Fundraising Opportunity:**
Determine Fund: Amount = 1500.00 → `CORPORATE_GRANT`, otherwise → `Operating Fund`

```json
{
  "Name": "[First] [Last] - Fundraising - [CloseDate]",
  "RecordTypeId": "<RECORD_TYPE_ID_FUNDRAISING>",
  "StageName": "Closed Won",
  "Amount": [amount],
  "CloseDate": "[YYYY-MM-DD]",
  "AccountId": "[from Contact]",
  "CampaignId": "<CAMPAIGN_ID_DEFAULT>",
  "Fund__c": "[per fund rules]"
}
```

Query: `SELECT Opportunity_ID__c FROM Opportunity WHERE Id = '[ID]'`

### Step 3: Progress Report
Include fund breakdown (Operating Fund vs CORPORATE_GRANT count).

### Step 4: Append Columns
Add to header row (12) and donation rows ONLY. NOT to metadata, Totals, or summary rows.

### Step 5: Rename
`Benevity Disbursement [Disbursement ID] [Period Ending YYYY-MM-DD].csv`

### Step 6-8: Save, Slack, Google Drive
Same as Canada Helps. Slack message: "Here is the Benevity disbursement file (ID: [ID], period ending [date]). [X] donations totalling $[total]."

## Edge Cases

| Scenario | Handling |
|---|---|
| All donor fields = "Not shared by donor" | Use anonymous Contact |
| Email = "Not shared" but name is real | Search by name, flag for the operator |
| Only postal code available | Search by postal code, present matches |
| Amounts with commas (e.g. "1,500.00") | Strip commas before parsing |
| Corporate Grant rows | Often all "Not shared" — treat as anonymous |
| Amount = $1,500 | Fund = `CORPORATE_GRANT` |

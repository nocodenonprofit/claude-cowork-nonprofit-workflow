# Canada Helps Disbursement Processing

Process a CanadaHelps CharityDataDownload CSV: match or create Contacts in Salesforce, create Fundraising Opportunities, append Opportunity ID/Fund columns, send to Bookkeeping via Slack, upload to Google Drive.

## CSV Format (CanadaHelps CharityDataDownload)

| Column | Maps To |
|---|---|
| TRANSACTION NUMBER | Reference only |
| DONOR FIRST NAME | Contact.FirstName |
| DONOR LAST NAME | Contact.LastName |
| DONOR EMAIL ADDRESS | Contact.Email |
| DONOR ADDRESS 1 | Contact.MailingStreet (line 1) |
| DONOR ADDRESS 2 | Contact.MailingStreet (line 2, append with newline) |
| DONOR CITY | Contact.MailingCity |
| DONOR PROVINCE/STATE | Contact.MailingState |
| DONOR POSTAL/ZIP CODE | Contact.MailingPostalCode |
| AMOUNT | Opportunity.Amount |
| DONATION DATE | Opportunity.CloseDate (convert YYYY/MM/DD to YYYY-MM-DD) |
| FEE | Reference only |
| DISBURSEMENT DATE | Used for file naming |

**Appended columns:** OPPORTUNITY ID (`Opportunity_ID__c`), FUND (`Operating Fund` default)

## Step-by-Step

### Step 1: Parse CSV
1. Read the CSV
2. Count total rows, identify DISBURSEMENT DATE values
3. Present summary: donation count, dates, total amount, anonymous count
4. Ask the operator to confirm before proceeding

### Step 2: Process Each Row

**2a. Determine if Anonymous:**
If First Name = "Anon" AND Last Name = "Anon" → use Anonymous Contact `<CONTACT_ID_ANONYMOUS>`. Query to get AccountId. Skip to 2e.

**2b. Match Existing Contact:**
If email is NOT "Anon":
```sql
SELECT Id, FirstName, LastName, Email, AccountId, MailingStreet, MailingCity, MailingState, MailingPostalCode
FROM Contact WHERE Email = '[email]' LIMIT 5
```
- 1 match → use it
- Multiple → present to the operator
- None → search by name if email is "Anon" but name is real
- Still none → create new Contact

**2c. Create New Contact (if no match):**
```json
{
  "FirstName": "[name]", "LastName": "[name]",
  "Email": "[email, omit if Anon]",
  "MailingStreet": "[addr, omit if Anon]",
  "MailingCity": "[city, omit if Anon]",
  "MailingState": "[province, omit if Anon]",
  "MailingPostalCode": "[postal, omit if Anon]"
}
```
NPSP auto-creates Household Account. Query new Contact to get AccountId.

**2d. Update Contact Address (if needed):**
Compare CSV address to Salesforce. Update if SF is blank. Flag if both differ.

**2e. Create Fundraising Opportunity:**
```json
{
  "Name": "[First] [Last] - Fundraising - [CloseDate]",
  "RecordTypeId": "<RECORD_TYPE_ID_FUNDRAISING>",
  "StageName": "Closed Won",
  "Amount": [amount],
  "CloseDate": "[YYYY-MM-DD]",
  "AccountId": "[from Contact]",
  "CampaignId": "<CAMPAIGN_ID_DEFAULT>",
  "Fund__c": "Operating Fund"
}
```
For anonymous: Name = "Anonymous - Fundraising - [CloseDate]"

Query created Opportunity: `SELECT Opportunity_ID__c FROM Opportunity WHERE Id = '[ID]'`

### Step 3: Progress Report
Present summary: Opportunities created, Contacts matched/created, anonymous count, errors.

### Step 4: Append Columns
Add OPPORTUNITY ID and FUND columns to the CSV.

### Step 5: Rename
`Canada Helps Disbursement [date or date range].csv`

### Step 6: Save
Save to processed folder.

### Step 7: Slack
DM the bookkeeper (`<SLACK_USER_ID_BOOKKEEPER>`): "Here is the Canada Helps disbursement file for [dates]. [X] donations totalling $[total]."
If Slack unavailable → tell the operator to send manually.

### Step 8: Google Drive
Upload to folder `<GDRIVE_FOLDER_ID>`.
If unavailable → tell the operator to upload manually.

## Edge Cases

| Scenario | Handling |
|---|---|
| First + Last = "Anon" | Use anonymous Contact |
| Email = "Anon" but name is real | Search by name, flag for the operator |
| Address fields = "Anon" | Skip those fields |
| Multiple Contact matches | Present to the operator |
| Duplicate transaction | Check for existing Opp with same Amount + CloseDate on Account |
| Different Campaign/Fund | The operator will specify |

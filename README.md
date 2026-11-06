# AI Job Search Automation

An n8n workflow that turns a daily job search into a ranked shortlist with practical CV adaptation reports.

It retrieves up to 150 LinkedIn job postings through Apify, evaluates new opportunities against a candidate profile using Claude, and selects up to 10 relevant jobs. Each selected job receives a Markdown report in Google Drive and a record in Google Sheets. A final Gmail message summarizes the results.

## What it does

- Searches recent job postings across configurable keywords and locations.
- Filters senior job titles and removes duplicates within the run and against the job tracker.
- Scores technical fit, experience, education, career alignment, and realistic seniority.
- Separates unresolved eligibility requirements from eligible recommendations.
- Selects up to 10 APPLY or CONSIDER opportunities scoring at least 55/100.
- Chooses a base CV and suggests evidence-based edits, relevant keywords, and claims to avoid.
- Uploads one Markdown report per selected job to Google Drive.
- Updates Google Sheets by job URL and includes a link to the uploaded report.
- Sends an email summary after the reports and tracker rows have been saved.

This workflow prepares recommendations for manual review. It does not submit applications or edit the original CV documents.

## Stack

n8n · Apify · Anthropic Claude API · Google Drive · Google Sheets · Gmail

The workflow can run on a self-hosted n8n instance, including Docker.

## Repository

- `workflows/job-automation.public.json` — sanitized n8n workflow export.
- `README.md` — overview and configuration instructions.

## Setup

1. Import `workflows/job-automation.public.json` into n8n.
2. Install the Apify node package used by the export: `@apify/n8n-nodes-apify`.
3. Connect your own Apify, Anthropic, Google Drive, Google Sheets, and Gmail credentials to the corresponding nodes.
4. Configure the placeholders listed below.
5. Prepare your candidate profile, base CVs, and Google Sheet.
6. Review the scoring rules, search settings, model selection, and schedule.
7. Test with a small scrape limit before publishing and enabling scheduled runs.

### Configuration placeholders

| Placeholder | Where to configure it |
| --- | --- |
| `YOUR_PROFILE_FOLDER_ID` | Search files and folders → profile folder |
| `YOUR_APPLICATIONS_FOLDER_ID` | Upload file → destination folder |
| `YOUR_SPREADSHEET_ID` | Both Google Sheets nodes → spreadsheet |
| `jobs` | Both Google Sheets nodes → tab name; change if needed |
| `you@example.com` | Gmail → recipient |

Credentials are intentionally omitted. The exported workflow is inactive.

### Candidate files

Put these in your profile folder:

- One UTF-8 Markdown master profile containing the marker `MASTER PROFILE`.
- Your base CVs as text-readable PDFs. The current extraction branch reads PDF, not DOCX or scanned images.

The master profile should document education, experience, tools, projects, languages, and any verified work authorization. Keep factual claims consistent across the profile and CVs.

The workflow keeps the profile content before `## 🇪🇸` when that heading exists. Use an English-only profile, or place the English section first with this separator before translations. CV filenames are read from Drive rather than hardcoded in the public prompt.

### Google Sheets headers

Create a tab named `jobs` with the following first row. Paste this tab-separated line into cell A1:

```text
date	company	job_title	job_url	score	recommendation	priority	chosen_cv	fileName	status
```

The `fileName` cell contains a hyperlink to the Markdown report. The workflow matches existing records on `job_url`.

### Search and scoring

The supplied search targets AI/ML, quantitative finance, quantum technologies, and photonics roles in Europe, with a scrape limit of 150. Adjust the Apify input for your goals.

The scoring prompt assumes up to two years of candidate experience and includes strict visa and security-clearance rules. Adapt those rules to your actual circumstances before use. Unknown eligibility requirements are routed to REVIEW and excluded from the shortlist.

Both Anthropic nodes retain the model settings from the original export. Select a model available to your account before testing.

### Schedule

The export is configured for 11:00 in `Europe/Madrid`. Change this in the Schedule Trigger and workflow settings if needed.

For local hosting, the computer must be awake and n8n running at the scheduled time. Publishing the workflow to GitHub does not host or run it.

## Outputs

Each selected opportunity produces:

1. A Google Drive Markdown report with the chosen CV, suggested profile text, prioritized edits, supported keywords, and application notes.
2. A Google Sheets row with the job details, score, recommendation, report link, and application status.
3. An entry in the final email summary.

Fewer than 10 jobs may qualify. If a branch returns no items, downstream nodes may not execute, so the current workflow does not guarantee a confirmation email for an empty shortlist. The final email is a success summary, not a separate failure alert.

## Limits and operating costs

- Apify and Anthropic API usage incur costs according to your accounts and usage. Start with a small limit and inspect actual usage before scaling.
- AI scores and CV suggestions require human review, especially for eligibility, experience, and unsupported claims.
- Previously recorded jobs are skipped, but jobs that were scored and not shortlisted can be evaluated again in a later run.
- Re-running the upload stage can create duplicate report files, even though tracker rows are matched by job URL.
- The public export has been checked for JSON validity and preserved connections; it has not been run with fresh credentials.

## Privacy

This repository contains a sanitized workflow template. Personal CVs, candidate data, credential references, private Google resource IDs, and the original email recipient are not included. Keep your profile files and credentials outside the public repository. Candidate and job content is sent to Anthropic during evaluation and report generation.

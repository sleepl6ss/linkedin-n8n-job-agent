# LinkedIn n8n Job Agent

An importable n8n workflow template for finding LinkedIn job posts, deduplicating repeated regional postings with AI, scoring each role against a candidate profile, and sending high-fit opportunities to Telegram.

The template is designed for automation-focused searches such as n8n, AI automation, no-code/low-code workflow tools, Zapier, Make, operations automation, and support automation. You can change the search configuration without editing the workflow logic.

## What The Workflow Does

1. Runs manually or on a schedule.
2. Builds LinkedIn guest search URLs from configurable filters.
3. Fetches LinkedIn search result pages.
4. Extracts job cards: title, company, location, posting date, and job URL.
5. Uses an AI agent to deduplicate the same role posted across multiple regions.
6. Keeps only opportunities not seen in recent workflow runs.
7. Fetches each job detail page and extracts the `About the job` section when available.
8. Scores each job from 1 to 10 against `candidateProfile`.
9. Filters out roles below `minFitScore`.
10. Sends Telegram alerts with role, company, regions, links, fit score, and a short fit note.
11. Optionally hands eligible jobs to a local Easy Apply assistant endpoint. This is disabled by default and included only as an integration stub.

## Node Map

| Area | Nodes | Responsibility |
| --- | --- | --- |
| Triggers | `When clicking 'Execute workflow'`, `Every Hour` | Manual testing and scheduled runs. |
| Configuration | `Workflow Configuration` | Search filters, candidate profile, score threshold, Telegram chat ID, and optional Easy Apply settings. |
| Search | `Build Page Requests`, `Fetch LinkedIn Search Pages` | Build paginated LinkedIn guest search requests and fetch HTML. |
| Extraction | `Extract Job Cards` | Parse LinkedIn guest search cards into structured job items. |
| Deduplication | `Prepare Jobs For AI Deduplication`, `AI Deduplicate Jobs`, `Structured Output Parser`, `Expand AI Deduped Jobs` | Merge repeated same-company/same-role postings across regions. |
| Freshness | `Keep Only Newly Seen Opportunities` | Avoid repeat alerts using workflow static data. |
| Detail Enrichment | `Fetch Job Detail Page`, `Extract About the Job` | Fetch the representative posting and extract the job description. |
| Fit Scoring | `AI Score Job Fit`, `Structured Output Parser (Scoring)`, `Normalize And Filter Fit Scores` | Score roles against the profile and keep only roles above threshold. |
| Telegram | `Format Telegram Alerts`, `Send to Telegram` | Build and send HTML Telegram alerts. |
| Easy Apply Stub | `Prepare Easy Apply Task`, `Send Easy Apply Task`, `Easy Apply Status Webhook`, `Easy Apply Status Configuration`, `Format Easy Apply Status`, `Send Easy Apply Status to Telegram` | Optional integration points for a separate local browser assistant. Disabled until configured. |

## Configuration

Edit the values in `Workflow Configuration` after importing the workflow into n8n.

| Field | Default | What It Controls |
| --- | --- | --- |
| `keywords` | `AI automation` | LinkedIn search query. Change this to search for `n8n`, `Zapier automation`, `Make.com`, `workflow automation`, or another role family. |
| `location` | `Worldwide` | LinkedIn location filter. Use values like `United States`, `United Kingdom`, `Europe`, or a city/country. |
| `postedSeconds` | `86400` | Recency window in seconds. `86400` means last 24 hours. |
| `remoteWorkType` | `2` | LinkedIn remote filter. `2` is remote. Leave blank if you want broader results. |
| `pageSize` | `25` | Results per page. |
| `maxPages` | `8` | Number of search pages to fetch. Higher values increase coverage and requests. |
| `sortBy` | `DD` | LinkedIn sort mode. `DD` means newest first. |
| `candidateProfile` | example profile | Resume/profile text used by the scoring agent. Replace it with your private profile inside n8n. |
| `minFitScore` | `7` | Minimum score required for Telegram alerts and optional Easy Apply handoff. |
| `telegramChatId` | placeholder | Telegram chat ID for alerts. Keep it private. |
| `easyApplyEnabled` | `false` | Enables optional handoff to an external local browser assistant. Keep false unless you built that service. |
| `easyApplyWebhookUrl` | placeholder | URL for the optional Easy Apply handoff endpoint. |
| `easyApplyHandoffSecret` | placeholder | Shared secret used by the optional handoff endpoint. |
| `easyApplyStatusCallbackSecret` | placeholder | Shared secret for status callbacks. |
| `cvPath` | placeholder | Local CV path for optional Easy Apply integrations. Not used by the core Telegram flow. |

## Search Examples

Use one phrase at a time in `keywords`, then run manually to inspect quality before scheduling.

| Goal | Suggested `keywords` |
| --- | --- |
| n8n-focused roles | `n8n` |
| AI automation roles | `AI automation` |
| No-code automation | `no-code automation` |
| Zapier roles | `Zapier automation` |
| Make.com roles | `Make.com automation` |
| Operations automation | `operations automation` |
| Customer support automation | `customer support automation` |
| AI agent workflow roles | `AI agents workflow automation` |
| Integration builder roles | `workflow integration specialist` |

You can also tune `location`, `postedSeconds`, and `minFitScore` to widen or narrow results. For example, use `postedSeconds = 604800` for the last 7 days, or `minFitScore = 6` if you want to inspect more borderline roles.

## Setup

1. Import `workflows/linkedin_remote_jobs_ai_fit_scoring.template.json` into n8n.
2. Add OpenAI credentials to both chat model nodes.
3. Add Telegram credentials to Telegram nodes.
4. Replace `candidateProfile` with your private resume/profile text.
5. Set `telegramChatId` to your private Telegram chat ID.
6. Adjust `keywords`, `location`, `postedSeconds`, and `minFitScore`.
7. Run the workflow manually and inspect the first Telegram alerts.
8. Enable the schedule only after the manual run looks correct.

## Privacy Notes

This repository intentionally contains no real resume, contact details, Telegram chat ID, local file paths, browser profile, private answer database, or secrets.

Do not commit:

- Real `candidateProfile` text.
- Telegram chat IDs or bot tokens.
- OpenAI credentials.
- Local CV paths.
- Easy Apply secrets or tunnel URLs.
- HAR files, execution exports, browser profiles, or private answer databases.

## Troubleshooting

| Problem | What To Check |
| --- | --- |
| No LinkedIn results | Try a broader `keywords` value, increase `maxPages`, or use a wider `location`. |
| Jobs appear but no Telegram alerts | Lower `minFitScore`, check OpenAI credentials, and inspect `Normalize And Filter Fit Scores`. |
| Job descriptions are missing | Some LinkedIn guest pages hide or expire details. The scorer falls back to title/company/location and scores conservatively. |
| Telegram message fails | Confirm Telegram credentials, `telegramChatId`, and HTML parse mode. |
| Duplicate alerts keep appearing | Clear workflow static data only when you intentionally want to re-alert old jobs. |
| Easy Apply does nothing | `easyApplyEnabled` is false by default. This repo does not include the local browser assistant. |

## Safety

LinkedIn pages and job descriptions are external content. Treat them as untrusted input. The workflow uses structured AI outputs and does not automatically apply to jobs in the public template.

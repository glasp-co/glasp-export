# Glasp Export
Automatically export your [Glasp](https://glasp.co) highlights to your favorite tools using GitHub Actions. No server required — just set up once and it runs on a schedule.

## Supported Destinations
| Destination | Status | Guide |
|---|---|---|
| Slack | ✅ Available | [Setup →](#slack) |
| Google Sheets | ✅ Available | [Setup →](#google-sheets) |
| Airtable | 🔜 Coming soon | — |

> Want a destination added? [Open an issue](../../issues) and let us know!

---

## Quick Start

### 1. Create your repo from this template
Click the green **"Use this template"** button at the top of this page, then select **"Create a new repository"**. We recommend making it **private**.

### 2. Add your Glasp Access Token
1. Go to [glasp.co](https://glasp.co) and sign in
2. Open **Settings** → **Access Token**
3. Copy the token
4. In your new repo, go to **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
5. Name: `GLASP_ACCESS_TOKEN` / Value: your token

### 3. Set up a destination
Follow the guide for the destination(s) you want below. You can enable multiple destinations at the same time — just add the secrets for each one.

---

## Destinations

### Slack
Send new highlights to a Slack channel with rich formatting and thumbnails.

#### Setup
1. Go to [api.slack.com/apps](https://api.slack.com/apps) and create a new app (or use an existing one)
2. Enable **Incoming Webhooks**
3. Click **"Add New Webhook to Workspace"** and select a channel
4. Copy the webhook URL
5. Add a repository secret: Name: `SLACK_WEBHOOK_URL` / Value: your webhook URL
6. Go to **Actions** → **Glasp → Slack** → **Run workflow** to test

The workflow runs daily at 09:00 UTC by default.

#### Configuration
Edit `.github/workflows/glasp_to_slack.yml` to customize:

**Schedule:**
```yaml
schedule:
  - cron: "0 9 * * *"    # Daily at 09:00 UTC (default)
  - cron: "0 9 * * 1"    # Every Monday at 09:00 UTC
  - cron: "0 */6 * * *"  # Every 6 hours
```

**Options** (uncomment in the workflow file):
| Variable | Default | Description |
|---|---|---|
| `LOOKBACK_HOURS` | `24` | How far back to fetch highlights |
| `MAX_DOCS` | `5` | Max documents posted per run |
| `MAX_HIGHLIGHTS_PER_DOC` | `10` | Max highlights shown per document |

> **Tip:** If you change the cron schedule, set `LOOKBACK_HOURS` to match. For example, every 6 hours → `LOOKBACK_HOURS: "6"`.

---

### Google Sheets
Append new highlights to a Google Sheet — one row per highlight.

**Output columns:** Timestamp, Document Title, Document URL, Glasp URL, Highlight Text, Note, Tags, Color, Highlighted At

#### Setup

**1. Create a Google Cloud Service Account**
1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (or select an existing one)
3. Enable the **Google Sheets API** (APIs & Services → Enable APIs → search "Google Sheets API")
4. Go to **IAM & Admin** → **Service Accounts** → **Create Service Account**
5. Give it a name, click **Done** (no roles needed)
6. Click on the service account → **Keys** → **Add Key** → **Create new key** → **JSON**
7. Download the JSON file

**2. Share your Google Sheet with the Service Account**
1. Open the JSON file and copy the `client_email` value (e.g., `name@project.iam.gserviceaccount.com`)
2. Open your Google Sheet → **Share** → paste the email → set role to **Editor**

**3. Add secrets to your repo**
1. `GOOGLE_SERVICE_ACCOUNT_JSON` — paste the entire contents of the JSON file
2. `GOOGLE_SHEET_ID` — the ID from your sheet URL: `https://docs.google.com/spreadsheets/d/`**`THIS_PART`**`/edit`

**4. Test it**
Go to **Actions** → **Glasp → Google Sheets** → **Run workflow**

A "Highlights" tab will be created automatically on first run.

#### Configuration
Edit `.github/workflows/glasp_to_sheets.yml` to customize:

**Options** (uncomment in the workflow file):
| Variable | Default | Description |
|---|---|---|
| `LOOKBACK_HOURS` | `24` | How far back to fetch highlights |
| `SHEET_TAB` | `Highlights` | Sheet tab name |

---

## Project Structure
```
├── .github/workflows/
│   ├── glasp_to_slack.yml       # Slack workflow
│   └── glasp_to_sheets.yml      # Google Sheets workflow
├── scripts/
│   ├── glasp_export.py          # Shared: Glasp API client (used by all destinations)
│   ├── glasp_to_slack.py        # Slack: formatting + posting
│   └── glasp_to_sheets.py       # Google Sheets: formatting + appending
├── docs/
│   └── slack-example.png
├── LICENSE
└── README.md
```

## Troubleshooting
**Workflow runs but posts nothing** — Check that `LOOKBACK_HOURS` covers the period since your last highlight. Try increasing it or adding a new highlight in Glasp and re-running.

**401 Unauthorized** — Your `GLASP_ACCESS_TOKEN` may be expired. Generate a new one in Glasp Settings.

**Slack error** — Verify the webhook URL is correct and the Slack app is still installed in your workspace.

**Google Sheets 400 error** — Make sure the `SHEET_TAB` name matches an existing tab in your spreadsheet, or leave it as the default `Highlights` (created automatically on first run).

**Google Sheets 403 error** — Make sure you shared the spreadsheet with the Service Account email as Editor.

## License
MIT
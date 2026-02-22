# Glasp Export
Automatically export your [Glasp](https://glasp.co) highlights to your favorite tools using GitHub Actions. No server required — just set up once and it runs on a schedule.

## Supported Destinations
| Destination | Status | Guide |
|---|---|---|
| Slack | ✅ Available | [Setup →](#slack) |
| Google Sheets | ✅ Available | [Setup →](#google-sheets) |
| Airtable | ✅ Available | [Setup →](#airtable) |
| Notion | ✅ Available | [Setup →](#notion) |

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
5. Give it a name, click **Done** (no roles needed at the project level)
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
| Variable | Default | Description |
|---|---|---|
| `LOOKBACK_HOURS` | `24` | How far back to fetch highlights |
| `SHEET_TAB` | `Highlights` | Sheet tab name |

---

### Airtable
Append new highlights to an Airtable table — one row per highlight.

**Output fields:** Timestamp, Document Title, Document URL, Glasp URL, Highlight Text, Note, Tags, Color, Highlighted At

#### Setup

**1. Create an Airtable Base and Table**
1. Go to [airtable.com](https://airtable.com) and create a new Base
2. Rename the default table to `Highlights`
3. Add the following fields (all as **Single line text**):
   - `Timestamp`
   - `Document Title`
   - `Document URL`
   - `Glasp URL`
   - `Highlight Text`
   - `Note`
   - `Tags`
   - `Color`
   - `Highlighted At`

**2. Create a Personal Access Token**
1. Go to [airtable.com/create/tokens](https://airtable.com/create/tokens)
2. Click **Create new token**
3. Add scopes: `data.records:read` and `data.records:write`
4. Under **Access**, add your Base
5. Copy the token

**3. Get your Base ID**
Open your Base in Airtable — the URL looks like:
`https://airtable.com/appXXXXXXXXXXXXXX/...`
The `appXXXXXXXXXXXXXX` part is your Base ID.

**4. Add secrets to your repo**
1. `AIRTABLE_API_KEY` — your Personal Access Token
2. `AIRTABLE_BASE_ID` — your Base ID (starts with `app`)

**5. Test it**
Go to **Actions** → **Glasp → Airtable** → **Run workflow**

#### Configuration
| Variable | Default | Description |
|---|---|---|
| `LOOKBACK_HOURS` | `24` | How far back to fetch highlights |
| `AIRTABLE_TABLE_NAME` | `Highlights` | Table name |

---

### Notion
Add new highlights to a Notion Database — one page per article, with highlights as blocks inside each page.

**Database properties:** Name, URL, Glasp URL, Tags, Highlighted At, Highlights Count

**Page content:** Each highlight appears as a Quote block, with Notes as Callout blocks (📝) below.

#### Setup

**1. Create a Notion Integration**
1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click **"New integration"**
3. Give it a name (e.g., "Glasp Export"), select your workspace
4. Copy the **Internal Integration Token**

**2. Create a Notion Database**
Create a new Database (Table view) with the following properties:
- `Name` — Title (default)
- `URL` — URL
- `Glasp URL` — URL
- `Tags` — Multi-select
- `Highlighted At` — Date
- `Highlights Count` — Number

**3. Connect the Integration to your Database**
Open the Database → click **···** (top right) → **Connect to** → select your integration

**4. Get your Database ID**
Open the Database as a full page — the URL looks like:
`https://www.notion.so/yourworkspace/`**`DATABASE_ID`**`?v=...`
Copy the 32-character ID before the `?v=`.

**5. Add secrets to your repo**
1. `NOTION_API_KEY` — your Integration Token
2. `NOTION_DATABASE_ID` — your Database ID

**6. Test it**
Go to **Actions** → **Glasp → Notion** → **Run workflow**

#### Configuration
| Variable | Default | Description |
|---|---|---|
| `LOOKBACK_HOURS` | `24` | How far back to fetch highlights |

---

## Project Structure
```
├── .github/workflows/
│   ├── glasp_to_slack.yml       # Slack workflow
│   ├── glasp_to_sheets.yml      # Google Sheets workflow
│   ├── glasp_to_airtable.yml    # Airtable workflow
│   └── glasp_to_notion.yml      # Notion workflow
├── scripts/
│   ├── glasp_export.py          # Shared: Glasp API client
│   ├── glasp_to_slack.py        # Slack: formatting + posting
│   ├── glasp_to_sheets.py       # Google Sheets: formatting + appending
│   ├── glasp_to_airtable.py     # Airtable: formatting + appending
│   └── glasp_to_notion.py       # Notion: formatting + page creation
├── docs/
│   └── slack-example.png
├── LICENSE
└── README.md
```

---

## Troubleshooting

**Workflow runs but posts nothing** — Check that `LOOKBACK_HOURS` covers the period since your last highlight. Try increasing it or adding a new highlight in Glasp and re-running.

**401 Unauthorized** — Your `GLASP_ACCESS_TOKEN` may be expired. Generate a new one in Glasp Settings.

**Slack error** — Verify the webhook URL is correct and the Slack app is still installed in your workspace.

**Google Sheets 400 error** — Make sure the `SHEET_TAB` name matches an existing tab, or leave it as the default `Highlights` (created automatically on first run).

**Google Sheets 403 error** — Make sure you shared the spreadsheet with the Service Account email as Editor.

**Airtable 403 error** — Make sure the Personal Access Token has `data.records:read` and `data.records:write` scopes and that your Base is added under Access.

**Airtable 422 error** — Make sure all field names match exactly (case-sensitive) and that all fields are **Single line text** type.

**Notion 400 error** — Make sure the Database property names match exactly and that your integration is connected to the Database via **Connect to**.

---

## License
MIT
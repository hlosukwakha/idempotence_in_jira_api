# Idempotent REST API Pagination with Jira (Python + Google Colab)

This repository demonstrates how to fetch **Jira issues safely and consistently** from the **Enhanced Search API** using **cursor-based pagination** — ensuring idempotence even when data changes mid-fetch.

> 💡 *Idempotence* means you can rerun your ingestion job without producing duplicates or missing records, even if new issues are created or updated while you’re paging.

---

## 🚀 Features

✅ **Enhanced Search (Cursor-based)** — Uses Jira’s modern `/rest/api/3/search/jql` endpoint  
✅ **Resumable Cursor** — Progress is saved in a `jira_cursor.json` file after every batch  
✅ **Consistent Ordering** — Deterministic sort: `ORDER BY updated ASC, key ASC`  
✅ **Resilient to Updates** — Skips already-processed issues even if Jira data changes  
✅ **Backoff Handling** — Automatic retry on rate limits (`429`)  
✅ **CSV Output** — Writes clean, deduped results to `jira_issues.csv`  
✅ **Colab-Ready** — Works directly in Google Colab with optional Google Drive persistence

---

## 📂 Project Structure

```
├── README.md
├── jira_pagination_colab.ipynb   # Google Colab notebook (copy cells into Colab)
├── jira_cursor.json              # Automatically created incremental cursor
├── jira_issues.csv               # Output file (appended safely between runs)
└── sample_data/
    └── jira_issues_sample.csv    # Example schema for the output
```

---

## ⚙️ Requirements

| Dependency | Version | Notes |
|-------------|----------|-------|
| Python | 3.9+ | Works in Colab’s default runtime |
| Libraries | `requests`, `pandas`, `python-dateutil` | Installed automatically |
| Jira | Cloud | Must support the [Enhanced Search API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issue-search/#api-rest-api-3-search-jql-post) |
| Authentication | API token + email | Atlassian Cloud Basic Auth |

---

## 🧠 How It Works

The notebook uses **cursor-based pagination** to iterate through issues ordered by `updated` and `key`.  
It maintains a **cursor checkpoint** (`jira_cursor.json`) that stores:

```json
{
  "last_updated": "2025-10-01T09:00:00.000+0000",
  "last_issue_id": "10023",
  "nextPageToken": "eyJwYWdlIjoiMiJ9",
  "done": false
}
```

Each subsequent run resumes from that point, skipping already-processed issues.  
If the ingestion stops halfway (e.g., rate limit, runtime timeout), it will **resume exactly where it left off**.

---

## 🪜 Setup (Google Colab)

1. Open a new Colab notebook:  
   [https://colab.research.google.com/](https://colab.research.google.com/)

2. Paste the notebook cells from this repo (`jira_pagination_colab.ipynb`).

3. In the **configuration section**, set your credentials:
   ```python
   JIRA_BASE_URL = "https://yourdomain.atlassian.net"
   JIRA_EMAIL    = "you@example.com"
   JIRA_API_TOKEN= "YOUR_API_TOKEN"
   ```

4. (Optional) Mount Google Drive to persist data between runs:
   ```python
   from google.colab import drive
   drive.mount('/content/drive')
   ```

5. Run all cells.  
   The script will:
   - Fetch issues in batches  
   - Write each batch to `jira_issues.csv`  
   - Update the incremental `jira_cursor.json`  

6. You can stop and restart the notebook safely — it will resume automatically.

---

## 🧩 Example Output Schema

| key | id | updated | created | summary | status | assignee | labels | reporter | project | issue_type | priority | url |
|------|----|----------|----------|----------|---------|-----------|---------|-----------|-----------|------------|------|
| PROJ-1 | 10001 | 2025-10-06T09:12:34.000+0000 | 2025-09-28T14:03:11.000+0000 | User cannot log in (403) | To Do | alice@example.com | incident,login | bob@example.com | PROJ | Bug | High | https://example.atlassian.net/browse/PROJ-1 |
| PROJ-2 | 10002 | 2025-10-07T11:25:10.000+0000 | 2025-10-01T08:22:51.000+0000 | Upgrade Kafka Connect cluster | In Progress | charlie@example.com | platform,kafka | dana@example.com | PROJ | Task | Medium | https://example.atlassian.net/browse/PROJ-2 |

---

## 🛠️ Error Handling & Troubleshooting

| Error | Likely Cause | Fix |
|-------|---------------|-----|
| **400 Bad Request** | Invalid JQL or unsupported field (`ORDER BY id`) | Use `ORDER BY updated ASC, key ASC` |
| **401 / 403 Unauthorized** | Bad token or missing permissions | Regenerate API token and ensure *Browse Projects* access |
| **429 Too Many Requests** | Hit Jira’s rate limit | Script automatically backs off; lower `MAX_RESULTS` |
| **404 Not Found** | Using Server/DC URL | Only Jira Cloud supports `/search/jql` |
| **KeyError / NoneType** | Hidden or empty field | Safe access already handled; you can extend fields list |

---

## 📊 Why This Script Is Idempotent

- ✅ **Stable ordering** — deterministic `(updated, key)` sort  
- ✅ **Monotonic cursor** — only moves forward after a successful write  
- ✅ **Boundary filter** — skips already-processed issues  
- ✅ **Deduplication** — avoids writing the same issue twice  
- ✅ **Resumable runs** — safe to stop/restart at any time  

---

## 📈 Extending the Project

You can easily adapt this framework to:

- Incremental syncs to a data warehouse (use UPSERT/MERGE by issue id)  
- Daily scheduled runs via Airflow, Prefect, or n8n  
- Integration with analytics notebooks or Power BI dashboards  
- Historical backfills using a windowed JQL (e.g., `updated >= "2025/10/01"`)

---

## 🧾 License

MIT License — feel free to reuse and adapt for your own Jira integrations.

---

## 👤 Author

**Mpumelelo Ndlovu (@hlosukwakha)**  
Cloud Support Engineer & Data Architect  
📍 Alkmaar, Netherlands  
🔗 [LinkedIn](https://linkedin.com/in/mpums) | [GitHub](https://github.com/hlosukwakha)

---

### 🏁 Quick Start

```bash
# Run inside Colab
!pip install requests pandas python-dateutil
python jira_pagination_colab.py
```

➡️ Output: `jira_issues.csv` + `jira_cursor.json`

# GitHub Rock · Backend Data

> **Real GitHub data. Clean pipelines. Offline-ready feeds.**

Backend data infrastructure for **GitHub Rock** — discovering real open-source projects, organizing them by platform and category, validating release assets, and publishing cacheable JSON for the app and backend.

---

## ✦ What this powers

| Experience | Available |
| --- | --- |
| Trending | ✓ |
| New Releases | ✓ |
| Most Popular | ✓ |
| Topics | ✓ |
| Platform feeds | ✓ |
| Offline feed cache | ✓ |
| Release validation | ✓ |
| PostgreSQL persistence | Optional |
| Meilisearch indexing | Optional |

**Platforms:** Android · Windows · macOS · Linux

**Topics:** Privacy · Media · Productivity · Networking · Dev Tools

---

## ◈ Data flow

~~~text
GitHub
  │
  ├─ repositories
  ├─ releases
  └─ release assets
        │
        ▼
┌───────────────────────────┐
│  GitHub Rock Data Pipeline │
│                           │
│  discover → verify → rank │
│           → validate      │
└──────────────┬────────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
 cached-data/       optional stores
 JSON feeds         PostgreSQL / Meilisearch
       │
       ▼
 GitHub Rock Backend
       │
       ▼
 GitHub Rock Android
~~~

The generated cache is the portable source for offline and feed workflows. The backend remains the API boundary consumed by GitHub Rock.

---

## ▣ Generated data

~~~text
cached-data/
├── feed/
├── trending/
├── new-releases/
├── most-popular/
└── topics/
    ├── privacy/
    ├── media/
    ├── productivity/
    ├── networking/
    └── dev-tools/
~~~

Each generated JSON document follows a stable envelope:

~~~json
{
  "category": "trending",
  "platform": "android",
  "lastUpdated": "2026-09-20T02:00:00Z",
  "totalCount": 0,
  "repositories": []
}
~~~

The repository list is populated from real GitHub results by the scheduled pipeline. The empty structure above is documentation only; no fake catalog entries are committed.

---

## ⬡ Real release detection

The pipeline looks for **real platform installers** attached to GitHub releases.

| Platform | Release assets |
| --- | --- |
| Android | .apk |
| Windows | .exe, .msi |
| macOS | .dmg, .pkg |
| Linux | .AppImage, .deb, .rpm, .pkg.tar.zst |

Android detection explicitly excludes Alpine Linux packages that also use the .apk extension.

Repositories are filtered and validated before they enter the generated catalog.

---

## ⚡ Automation

### Fetch All Repository Categories
- Daily at **02:00 UTC**
- Manual dispatch supported
- Validates generated JSON
- Commits refreshed cache data

### Publish Feed Offline Cache
- Runs every **3 hours**
- Manual dispatch supported
- Mirrors the backend feed for offline use

### Validation
- Python syntax checks
- Workflow YAML validation
- Release/data validation

No generated data is claimed to be live until an actual workflow has produced it.

---

## 🔐 Credentials

Category fetches support dedicated GitHub Actions tokens:

~~~text
GH_TOKEN_TRENDING
GH_TOKEN_NEW_RELEASES
GH_TOKEN_MOST_POPULAR
GH_TOKEN_TOPICS
~~~

GITHUB_TOKEN can be used as the fallback.

Optional integrations:

~~~text
DATABASE_URL
MEILI_URL
MEILI_MASTER_KEY
~~~

Secrets stay in GitHub Actions or the deployment environment and are never committed to this repository.

---

## ⇄ Feed contract

The feed publisher targets:

~~~text
https://github-rock-backend.onrender.com/v1/feed
~~~

Set FEED_BACKEND_ORIGIN to use another backend origin.

The backend integration must serve real generated data; this repository does not create a second or fake API layer.

---

## 🛠 Local development

Requirements: **Python 3.11+**

~~~bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r scripts/requirements.txt

GITHUB_TOKEN=ghp_... python scripts/fetch_all_categories.py
~~~

Validate the pipeline:

~~~bash
python scripts/validate_releases.py
python -m compileall -q scripts
~~~

---

## 📁 Repository map

~~~text
scripts/
├── fetch_all_categories.py
├── fetch_feed_cache.py
├── validate_releases.py
├── db_writer.py
├── meili_sync.py
└── backfill_downloads.py

.github/workflows/
├── fetch_all_categories_workflow.yml
├── fetch_feed_cache.yml
└── validate.yml

cached-data/
└── generated JSON feeds
~~~

---

## Design principles

- **Real data only** — no fake repositories or placeholder catalog records.
- **GitHub-native** — GitHub releases and metadata are the source.
- **One data layer** — avoid duplicate pipelines and competing sources of truth.
- **Offline-ready** — generated JSON can be consumed without live discovery.
- **Validated output** — malformed or invalid generated data should not be published.
- **Automation first** — scheduled workflows keep maintenance out of the app.
- **Backend boundary** — the Android app talks to the backend/API rather than inventing its own catalog system.

---

## License

Apache 2.0

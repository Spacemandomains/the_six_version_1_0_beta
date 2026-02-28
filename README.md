# AI CPO Agent — The Six SaaS

An AI-powered Chief Product Officer built for SaaS founders. It acts as an always-available executive that translates founder vision into structured product strategy using Google Gemini AI.

## What It Does

The AI CPO monitors your Google Doc for "Dear CPO" messages and automatically generates:

- **Daily Recap** — structured summary of what happened (outcome, decisions, blockers)
- **Daily CPO Brief** — strategic guidance with focus, next action, kill list, and a question for you
- **Customer Recap** — polished external product update (optional, separate doc)
- **Task Tracking** — assign tasks via "Dear CPO, task: ..." with due dates and completion tracking

## Tech Stack

- **Python 3.11** / **FastAPI** with Jinja2 templates
- **PostgreSQL** via SQLAlchemy ORM
- **Google Gemini AI** for content generation
- **Google Docs API** for reading/writing documents
- **APScheduler** for background monitoring

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `GEMINI_API_KEY` | Yes | Google Gemini API key |
| `JOB_SECRET` | Yes | Secret for the fallback job trigger endpoint |
| `GOOGLE_SERVICE_ACCOUNT_JSON` | Yes (Render) | Google Service Account JSON key (see setup below) |

## Local Development

```bash
pip install -r requirements.txt
export DATABASE_URL="postgresql://user:pass@localhost:5432/ai_cpo"
export GEMINI_API_KEY="your-key"
export JOB_SECRET="your-secret"
export GOOGLE_SERVICE_ACCOUNT_JSON='{"type":"service_account",...}'
uvicorn app.main:app --host 0.0.0.0 --port 5000 --reload
```

## Deploy on Render

### 1. Create the Service

- Go to [render.com](https://render.com) and click **New > Blueprint**
- Connect your GitHub repo (`Spacemandomains/the_saas_version_1_0`)
- Render will detect the `render.yaml` and set up a web service + PostgreSQL database automatically

**Or manually:**
- Click **New > Web Service**, connect the repo
- Set **Build Command**: `pip install -r requirements.txt`
- Set **Start Command**: `gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:$PORT`
- Click **New > PostgreSQL** to create a database, then copy the **Internal Database URL** into the `DATABASE_URL` env var

### 2. Set Environment Variables

In your Render web service settings, add:

- `DATABASE_URL` — from your Render PostgreSQL (Internal Connection String)
- `GEMINI_API_KEY` — your Google Gemini API key
- `JOB_SECRET` — any strong random string
- `GOOGLE_SERVICE_ACCOUNT_JSON` — the full JSON key from Google (see below)

### 3. Set Up Google Service Account

This is how the AI CPO reads and writes to your Google Docs on Render:

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (or use an existing one)
3. Enable the **Google Docs API**:
   - Go to **APIs & Services > Library**
   - Search for "Google Docs API" and click **Enable**
4. Create a Service Account:
   - Go to **APIs & Services > Credentials**
   - Click **Create Credentials > Service Account**
   - Give it a name (e.g., "ai-cpo-docs")
   - Click **Done**
5. Create a key:
   - Click on the service account you just created
   - Go to the **Keys** tab
   - Click **Add Key > Create new key > JSON**
   - Download the JSON file
6. Copy the entire JSON content and paste it as the `GOOGLE_SERVICE_ACCOUNT_JSON` environment variable on Render
7. **Share your Google Docs** with the service account:
   - Find the service account email in the JSON file (the `client_email` field, looks like `ai-cpo-docs@your-project.iam.gserviceaccount.com`)
   - Open each Google Doc (source, output, recap) and click **Share**
   - Add the service account email with **Editor** access

### 4. Deploy

Click **Manual Deploy > Deploy latest commit** in Render, or push to your GitHub repo and Render will auto-deploy.

### 5. Create Your Account

Visit your Render URL and sign up. Then configure your Google Doc IDs in the CPO Dashboard settings.

## Project Structure

```
app/
  main.py          - FastAPI entrypoint, API + page routes
  scheduler.py     - APScheduler background monitoring
  cpo_agent.py     - Gemini AI agent orchestration
  daily_job.py     - Daily automation pipeline
  db.py            - SQLAlchemy models and database setup
  auth.py          - API key auth, password hashing
  google_docs.py   - Google Docs API (Service Account + Replit connector)
  tools.py         - Scoring utilities
templates/         - Jinja2 HTML templates
static/            - Static assets
```

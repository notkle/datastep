# DataStep

Guided, step-by-step CSV data cleaning. Human-in-the-loop approve/deny workflow.
All data processed locally in the browser — nothing is uploaded to any server.

## Stack
- **Frontend**: Vanilla HTML/CSS/JS — zero dependencies, zero build step
- **API**: Gemini 2.0 Flash (free tier — 1,500 requests/day)
- **Proxy**: Cloudflare Worker (free tier — 100,000 requests/day)
- **Hosting**: GitHub Pages (free)

**Total cost to run: $0**

---

## Deployment

### Step 1 — Deploy the Cloudflare Worker

1. Go to [workers.cloudflare.com](https://workers.cloudflare.com) and sign up free
2. Create a new Worker (use the online editor)
3. Paste the entire contents of `worker.js`
4. Go to **Settings → Variables → Add variable**
   - Name: `GEMINI_API_KEY`
   - Value: get a free key at [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)
5. Click **Deploy**
6. Copy the worker URL (looks like `https://datastep.yourname.workers.dev`)

### Step 2 — Configure the frontend

Open `index.html` and find these two lines near the top of the `<script>` block:

```js
const WORKER_URL = 'https://your-worker.your-subdomain.workers.dev';
const DEMO_MODE = true;
```

Replace the URL with your worker URL and set `DEMO_MODE = false`.

### Step 3 — Deploy to GitHub Pages

1. Create a new GitHub repository
2. Upload `index.html` to the root (worker.js stays local — never commit your worker code with secrets)
3. Go to **Settings → Pages → Source: main branch / root**
4. Your site will be live at `https://yourusername.github.io/repo-name`

### Optional: Lock down CORS

In `worker.js`, change:
```js
const ALLOWED_ORIGIN = '*';
```
to:
```js
const ALLOWED_ORIGIN = 'https://yourusername.github.io';
```
This prevents anyone else from using your worker endpoint.

---

## How it works

1. User uploads a CSV — parsed entirely in the browser via FileReader API
2. Full column statistics are computed locally (nulls, types, distributions, min/max/mean across ALL rows)
3. Stats summary (~3,000 tokens) is sent to Gemini — never the raw data
4. Gemini returns a structured JSON cleaning step
5. The browser applies the operation locally and renders a diff preview
6. User approves (step committed, next step fetched) or denies (feedback sent, step re-evaluated)
7. On completion, user downloads the cleaned CSV — generated locally

**Cost per session: ~$0.00** on Gemini free tier.

---

## Format modes

- **Excel** — formulas and UI instructions
- **SQL** — standard SQL ALTER/UPDATE statements  
- **Python** — pandas operations

Switch format before uploading. The selected format applies to all code suggestions in that session.

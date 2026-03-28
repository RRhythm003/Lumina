# Lumina Deployment Guide
## Stack: Netlify (Free) + Anthropic API

### What's in this folder

```
lumina.html                    ← The app (API calls go to /api/claude)
netlify.toml                   ← Netlify config (routes + headers)
netlify/functions/claude.js    ← Serverless proxy (holds your API key safely)
```

---

## Step 1 — Get Your Anthropic API Key

1. Go to https://console.anthropic.com
2. Click **API Keys** → **Create Key**
3. Copy the key (starts with `sk-ant-...`)
4. Store it somewhere safe — you'll paste it in Step 3

---

## Step 2 — Deploy to Netlify (2 minutes)

**Option A: Drag and Drop (Fastest)**

1. Go to https://app.netlify.com/drop
2. Drag the entire `lumina-deploy` **folder** onto the page
3. Netlify deploys in ~30 seconds
4. You get a URL like `https://amazing-lumina-abc123.netlify.app`

**Option B: GitHub (Recommended for updates)**

1. Create a GitHub repo
2. Push this entire folder to it
3. Go to https://app.netlify.com → **Add new site** → **Import from Git**
4. Select your repo → Deploy
5. Every `git push` auto-deploys

---

## Step 3 — Add Your API Key (Critical)

Without this step, all AI features return errors.

1. In Netlify: go to **Site Settings** → **Environment Variables**
2. Click **Add a variable**
3. Key: `ANTHROPIC_API_KEY`
4. Value: `sk-ant-...` (your key from Step 1)
5. Click **Save**
6. Go to **Deploys** → **Trigger deploy** → **Deploy site**

That's it. Your API key never touches the browser — it lives only in Netlify's secure env.

---

## Step 4 — Verify Everything Works

Open your Netlify URL. On the Home screen:

| Feature | What to check |
|---|---|
| ✨ Fact of the Day | Should load automatically with a real fact within 3 seconds |
| ❤️ Health → AI Calorie Counter | Type "rice and dal" → tap Analyze → should return macros |
| Both | If either shows "Could not load" → check your API key in Step 3 |

---

## Live Data Architecture

```
Browser (Lumina PWA)
    │
    │  POST /api/claude
    │  { model, messages, system }
    ↓
Netlify Edge (netlify/functions/claude.js)
    │
    │  POST api.anthropic.com/v1/messages
    │  x-api-key: sk-ant-... (from environment)
    ↓
Anthropic Claude Sonnet
    │
    │  { content: [{ text: "Did you know..." }] }
    ↓
Back to browser → renders in Lumina card
```

**Features using Claude API:**
- **Fact of the Day** — fetches once per day on app open, cached in localStorage
- **AI Calorie Counter** — fetches on each meal entry, returns calories + P/C/F/Fiber

**Features that are static (production would add live RSS):**
- News articles — 25 curated articles with real source links
- Stock ticker — static values (live data needs a market API subscription)

---

## Cost Estimate

At 100 DAU (daily active users):
- Fact of the Day: 100 calls/day × ~200 tokens = 20,000 tokens
- Calorie counter: ~50 calls/day × ~200 tokens = 10,000 tokens
- **Total: ~30,000 output tokens/day**
- Claude Sonnet 4: $3 per million output tokens
- **Estimated cost: ~$0.09/day (~$3/month) at 100 DAU**

---

## Custom Domain (Optional)

1. Netlify → **Domain management** → **Add custom domain**
2. Enter `lumina.yourdomain.com`
3. Update your DNS records as shown
4. SSL is automatic and free

---

## Updating the App

If using GitHub: edit `lumina.html` → commit → push → Netlify auto-deploys in ~20 seconds.

If using drag-and-drop: drag the updated folder to Netlify drop again.

---

## Troubleshooting

**"Could not load today's fact"**
→ Check API key is set correctly in Netlify Environment Variables
→ Trigger a redeploy after adding the key

**Fact loads in demo but not on Netlify**
→ Open browser DevTools → Network tab → look for `/api/claude` request
→ If 401: API key wrong or missing
→ If 500: Check Netlify function logs (Netlify → Functions → claude → View logs)

**App not installable as PWA**
→ Must be served over HTTPS (Netlify does this automatically)
→ Open on mobile → browser menu → "Add to Home Screen"

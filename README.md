# LinkedIn AI Auto-Poster

Automatically posts AI-generated LinkedIn content 3× per day, sourced from trending discussions across Reddit, Hacker News, Dev.to, Lobste.rs, Medium, and Product Hunt — powered by your choice of Groq, OpenAI, or Anthropic.

## Live Generator

👉 **[Open the Workflow Generator](https://artenisalija.github.io/linkedin-ai-autoposter/)**

Use the web UI to generate a custom n8n workflow JSON file for any industry and AI provider — no coding required.

---

## How It Works

```
Schedule (3×/day)
  → Fetch Reddit + HN + Dev.to + Lobste.rs (+ any custom sources)
  → Normalize & merge all posts
  → Pick top 6 by score → build prompt
  → AI generates LinkedIn post (Groq / OpenAI / Anthropic)
  → Post to LinkedIn via API
```

---

## Quick Start

### 1. Generate your workflow file

Open the [generator](https://artenisalija.github.io/linkedin-ai-autoposter/) and fill in:

| Field | Description |
|-------|-------------|
| Industry | Controls the AI prompt angle (e.g. "AI & Technology", "Finance") |
| Subreddits | Which Reddit communities to pull from |
| Content sources | Reddit, HN, Dev.to, Lobste.rs, Medium, Product Hunt, or custom |
| AI Provider | Groq (free), OpenAI, or Anthropic — paste your API key |
| LinkedIn token | Your Bearer token (see below) |
| Person ID | Your LinkedIn `sub` ID (see below) |
| Timezone | Your local timezone — post times are applied in this zone |
| Post times | Pick which hours to post each day |

Click **Generate & Download** to get a `.json` file ready to import.

---

### 2. Get your credentials

#### Groq API key (free)
1. Sign up at [console.groq.com](https://console.groq.com)
2. Go to **API Keys** → **Create API Key**

#### OpenAI / Anthropic
- OpenAI: [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
- Anthropic: [console.anthropic.com](https://console.anthropic.com) → API Keys

#### LinkedIn access token + Person ID
1. Go to [linkedin.com/developers](https://www.linkedin.com/developers/) → **Create App**
2. Under **Products**, request **"Share on LinkedIn"** (auto-approved)
3. Under **Auth**, add redirect URL: `http://localhost:3000`
4. Open this URL in your browser (replace `CLIENT_ID`):
   ```
   https://www.linkedin.com/oauth/v2/authorization?response_type=code&client_id=CLIENT_ID&redirect_uri=http://localhost:3000&scope=w_member_social%20openid%20profile
   ```
5. After approving, your browser redirects to `http://localhost:3000/?code=ABC123...` — copy the full URL
6. Exchange the code for a token:
   ```bash
   curl -X POST https://www.linkedin.com/oauth/v2/accessToken \
     --data-urlencode "grant_type=authorization_code" \
     --data-urlencode "code=PASTE_CODE_HERE" \
     --data-urlencode "redirect_uri=http://localhost:3000" \
     --data-urlencode "client_id=YOUR_CLIENT_ID" \
     --data-urlencode "client_secret=YOUR_CLIENT_SECRET"
   ```
7. Copy the `access_token` from the response
8. Get your Person ID:
   ```bash
   curl -H "Authorization: Bearer YOUR_TOKEN" https://api.linkedin.com/v2/userinfo
   ```
   Copy the `sub` field value

> **Note:** LinkedIn tokens expire after 60 days. Repeat steps 4–7 to refresh.

---

### 3. Import into n8n

1. Open n8n → **Workflows** → **Add Workflow**
2. Click the **⋮ menu** (top right) → **Import from file**
3. Select the downloaded `.json` file
4. Click **Save**, then toggle the workflow **Active**

The workflow runs automatically at your chosen times in your timezone.

---

## Template workflow

`ai-linkedin-workflow.json` is a ready-made template for AI & Tech content (Reddit AI subs + HN + Dev.to + Lobste.rs, posting at 8am / 1pm / 6pm UTC).

Replace the placeholders before importing:

| Placeholder | Replace with |
|-------------|-------------|
| `YOUR_GROQ_API_KEY` | Your Groq API key |
| `YOUR_LINKEDIN_ACCESS_TOKEN` | Your LinkedIn Bearer token |
| `YOUR_LINKEDIN_PERSON_ID` | Your LinkedIn `sub` ID |

---

## Adding custom sources

In the generator, scroll to **Content Sources** and click **+ Add Custom Source**. Enter:
- **Name** — label shown in logs
- **URL** — RSS feed or JSON API endpoint
- **Type** — RSS / JSON Array / JSON Object

The workflow normalizes all sources into a common format before merging.

---

## Adjusting the AI prompt

The prompt changes tone by time of day:
- **Morning** — news/trend with expert take + question
- **Afternoon** — actionable tip with concrete example
- **Evening** — motivational growth angle for businesses

To customize, edit the **Build Prompt** node in n8n after importing.

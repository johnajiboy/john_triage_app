# Message Triage Agent

A small AI automation that classifies an incoming message against a strict system prompt and returns structured JSON, which then drives a routing action.

Built for the AI System Prompting and Robust Rulesets activity. Runs entirely on a free API tier. No payment, no card.

## What it does

A message comes in. The AI reads it and returns JSON in this shape:

```json
{
  "category": "SECURITY",
  "priority": "HIGH",
  "reason": "short explanation"
}
```

The page reads the `category` field and picks an action:

| Category | Action |
| --- | --- |
| SALES | Sales response sent |
| REGISTRATION | Registration response sent |
| SUPPORT | Support response sent |
| COMPLAINT | Escalated to a human agent |
| SECURITY | Admin alerted |
| OTHER | General response sent |

There is a checkbox that removes Rules 1, 2 and 5 from the system prompt, so you can see what happens when the ruleset is weaker.

## Get a free API key

1. Go to **aistudio.google.com**
2. Sign in with a Google account
3. Click **Get API key**, then **Create API key**
4. Copy it

No card is required. The free tier allows roughly 1,500 requests a day, which is far more than this demo needs.

## Files

```
index.html         the page
api/classify.js    serverless function that calls the Gemini API
package.json
.env.example
```

The API key lives on the server only. The browser never sees it.

## Run it locally

You need Node 18 or newer.

Create a `.env` file:

```
GEMINI_API_KEY=your-key-here
```

Then:

```bash
npm i -g vercel
vercel dev
```

Open http://localhost:3000

## Put it on GitHub

```bash
git init
git add .
git commit -m "Message triage agent"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/message-triage-agent.git
git push -u origin main
```

`.gitignore` already excludes `.env`, so your key will not be committed.

## Deploy on Vercel

1. Go to vercel.com, click **Add New**, then **Project**
2. Import the GitHub repo
3. Framework preset: **Other**. No build command and no output directory needed.
4. Open **Environment Variables** and add:
   - Name: `GEMINI_API_KEY`
   - Value: your key
5. Click **Deploy**

If you add the key after deploying, redeploy once so the function picks it up.

## Changing the model

The model id is set by an environment variable, so you can change it without touching the code:

```
GEMINI_MODEL=gemini-3.6-flash
```

If not set, it defaults to `gemini-3.5-flash`. Free tier models are the Flash and Flash-Lite ones. Pro models are paid only.

## Other free options

If you prefer a different provider, these also have a permanent free tier with no card:

* **Groq** at console.groq.com, very fast, thousands of requests a day
* **OpenRouter** at openrouter.ai, has free models
* **Cerebras** at cloud.cerebras.ai

All three use an OpenAI-compatible endpoint, so swapping means changing the URL, the header and the response path in `api/classify.js`. The two prompt strings stay exactly the same.

## Notes on the code

`api/classify.js` does two things worth pointing out:

* It strips code fences from the output in case the JSON comes back wrapped, even though Rule 8 tells the model not to do that.
* It checks the category against the allowed list and falls back to OTHER if it is something unexpected. An automation should never route on a value it does not recognise.

Temperature is set to 0 so repeated runs of the same message give the same answer, which matters when you are comparing the strong and weak rulesets.

One deliberate choice: the request does not force JSON mode at the API level, even though Gemini supports that. Forcing it would guarantee valid JSON for structural reasons and the prompt rules would no longer be what is being tested. Leaving it off keeps the experiment honest.

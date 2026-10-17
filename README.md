# 🐝 Hornet Solve

An AI chatbot that helps Sacramento State students with financial aid, transfer
credits, degree planning, advising, clubs, campus resources, IT, and ASI —
personalized to each student.

This repo is a **prototype / foundation** designed to grow into the real product:
scrape Sac State data into a knowledge base, and answer student questions from it
with an AI chatbot as the main page.

## Run it

No build step. Open `index.html` in a browser, or serve the folder:

```bash
# any static server works, e.g.
python -m http.server 8000
# then visit http://localhost:8000
```

## Architecture

```
index.html                 Single-page chat UI
css/styles.css             Styling (Sac State green/gold)
src/
  data/
    schema.js              Data contract — the shape every scraped doc must match
    catalog.js             ⭐ THE KNOWLEDGE BASE — scraper output goes here
    students.js            Student profiles (from SIS/login in production)
  core/
    knowledge.js           Retrieval: finds relevant docs for a question
    chatEngine.js          Builds grounded, personalized answers
  llm/
    provider.js            Pluggable AI provider (mock now, OpenAI/Claude later)
  app.js                   UI wiring
scripts/scrape.md          How to scrape Sac State into catalog.js
```

### The request flow

```
user question
   → knowledge.retrieve()      find top matching docs from catalog.js
   → chatEngine                render a grounded, personalized answer
   → llm/provider              mock returns it as-is; real LLM can rewrite it
   → chat UI
```

### Two seams built for your team

1. **Data seam (`src/data/catalog.js`)** — the entire knowledge base is one
   array of documents. Point your scrapers at it. See `scripts/scrape.md`.
   Nothing else needs to change when the data gets real.

2. **AI seam (`src/llm/provider.js`)** — today a `MockProvider` returns grounded
   HTML with zero hallucination. To make it a true generative chatbot:
   - set `LLM_CONFIG.useReal = true` and add a key/backend, OR
   - keep the mock for structured answers and only call the LLM for open-ended
     questions.
   The chat engine already passes the retrieved docs + student profile to the
   provider, so a real model gets grounded context automatically.

## What works right now (offline, no API key)

- Personalized answers for 2 sample students (switch via the top-right selector)
- Financial aid eligibility computed from the student profile
- Transfer credit mapping + contacts
- Degree checklist with transfer credits marked, progress bar
- Remaining courses + semester-by-semester plan
- Major-change scenarios (CS → CPE/EEE/ME) with "behind on" courses + new grad date
- Advising, department contacts, campus map (Apple + Google Maps links)
- Club recommendations (scored by major/culture/interests, inactive hidden)
- Tutoring/PAL + cultural resources, IT/ServiceNow, ASI benefits
- Source citations (`csus.edu` links) on knowledge-based answers

## Important: data is sample data

Everything in `catalog.js` and `students.js` is **illustrative sample data** for
the demo. Replace it with scraped/real data before relying on any contact,
phone number, room, or award amount.

## Suggested next steps

1. Build the scrapers described in `scripts/scrape.md`; overwrite `catalog.js`.
2. Add real student auth (CAS/SSO) and pull profiles from the SIS.
3. Turn on a real LLM provider for open-ended questions.
4. Add a small backend to hold the API key and cache scraped data.
5. Add a "schedule upload" parser to match club meeting times to free times.
```

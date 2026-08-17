# AI Support Desk

An internal IT support tool with six pages: **Dashboard**, **Raise Ticket**, **Open Tickets**,
**AI Agent**, **About**, and **Settings** — plus sign in / sign up.

## Run it

```bash
pip install -r requirements.txt
cp .env.example .env   # then add your GEMINI_API_KEY
python main.py
```

Open `http://localhost:5000`. The SQLite database (`database.db`) and tables are created
automatically on first run.

## What changed from the original files

**Removed**
- The marketing landing page (`home.html`) and its fabricated stats, testimonials, and
  buzzword copy ("cognitive resolution path", "vector parsing sequences", 99.4%/40% stats,
  a quote from a made-up director). None of that was one of the six real app pages.
- The unused document-upload API and `documents` table — no page in the app used them.
- Duplicated per-page HTML for the header/sidebar (~150 lines repeated six times) and the
  per-page JavaScript that manually highlighted the active sidebar link.
- Hardcoded/fake data everywhere: the dashboard's static ticket counts and "AI Copilot
  Efficiency" percentages, Open Tickets' hardcoded rows, and Raise Ticket's `setTimeout`
  fake AI response.
- A stray code comment claiming the chat was "forwarded to a local Ollama model" — the
  actual backend calls Gemini; the comment was simply wrong.
- The broken `../static/...` relative asset paths (only correct if a page happened to be
  served one level deep) in favor of Flask's `url_for('static', ...)`.

**Added / wired up for real**
- A shared `base.html` layout. Every page's sidebar link is now marked active by comparing
  `request.endpoint` server-side, and a breadcrumb in the header always shows where you are —
  so navigation is consistent and correct instead of copy-pasted per page.
- A `tickets` table and full CRUD API (`/api/tickets`, `/api/tickets/<id>`, `/api/dashboard/stats`).
- Raising a ticket now calls Gemini for a real category/priority/summary/recommendation,
  stores it, and the resolve/escalate buttons actually update the ticket's status.
- Open Tickets lists real tickets from the database with working search/department/status
  filters and a detail view that can mark a ticket In Progress or Resolved.
- The Dashboard's numbers and "recent tickets" list are now live queries, not fixed text.
- Settings actually persists a display-name change, a password change, and two notification
  preferences (`/api/settings`, `/api/settings/profile`, `/api/settings/password`).
- Login and Register now call the real `/api/auth/*` endpoints with error handling and
  redirect on success, instead of a `handleFormValidation` stub that did nothing.
- About page describes the real stack (Flask + SQLite + the Gemini REST API) instead of
  the original's inaccurate "FastAPI" claim and invented uptime/CPU numbers, and shows a
  live queue snapshot instead of static figures.
- Dark mode preference now persists across visits (`localStorage`) instead of resetting.

## Project structure

```
main.py                  Flask app: pages, auth, tickets, dashboard, settings, chat APIs
templates/
  base.html               Shared header/sidebar/breadcrumb layout
  login.html, register.html
  dashboard.html, raise_ticket.html, open_tickets.html, ai_agent.html, about.html, settings.html
static/
  css/style.css           Small shared utilities (scrollbars, active-nav indicator, focus states)
  js/app.js                Theme toggle, sidebar toggle, toasts, auth-aware fetch helper
```

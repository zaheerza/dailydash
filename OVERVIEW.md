# DailyDash — Overview

## What it is

`index.html` is a self-contained single-page morning dashboard called **"Your Morning Brief"**. It requires no server, no build step, and no external dependencies beyond a Google Fonts import and the Anthropic API. Everything runs in the browser.

## How it works

On first load the user is prompted for an Anthropic API key. The key is saved to `localStorage` and is the only credential required. All subsequent AI calls go directly from the browser to `api.anthropic.com` using `claude-haiku-4-5-20251001`.

Content is cached in `localStorage` per calendar day (cache key prefix `ddc_v8_*`) so the page loads instantly on repeat visits and API calls only fire once per day. A "Refresh All" button forces regeneration.

## Layout

Two-column grid (collapses to single column on mobile ≤ 768 px):

| Column | Sections |
|--------|----------|
| Main   | Opening quote · News Digest · Canada · Tech & AI · Today's Learning + Quiz |
| Sidebar | Markets · Brain Food · World at a Glance · Toronto Events · Today's Discovery · Your Tilts · Rabbit Hole |

## Sections

### Opening Quote
Claude generates one of three formats each day — a poem excerpt, a Quranic verse with context, or a quote from a thinker — weighted toward the user's active interests.

### News Digest
Pulls up to 10 RSS feeds concurrently (CBC Toronto/Canada/World/Business, CP24, National Post, BlogTO, Globe & Mail, News24, Daily Maverick), pools up to 30 articles, then asks Claude to select 5 and write a 2-sentence synthesis per item. Mix is fixed: 2 Canada/Toronto, 1 South Africa, 2 global.

### Canada
Dedicated Canadian section from Toronto Star, Globe & Mail, Maclean's, and iPolitics. Claude selects 3 real items + generates 1 FT-style analysis item. Falls back gracefully if feeds are unreachable.

### Tech & AI
Pulls DeepLearning.AI ("The Batch") and Turing Post RSS feeds. Claude selects 3 items spanning research, application, and industry news.

### Today's Learning
Claude teaches a concept, historical event, or idea (≤ 200 words) that builds on previous sessions. Learning history (last 30 topics) is stored in state so each day continues where the last left off. A "go deeper" button fetches a 150-word expansion inline.

### Quiz
Generated from the user's learning history (up to 6 past topics, shuffled). Three multiple-choice questions testing applied understanding across different subjects. Score is persisted to state. Cached daily.

### Markets
Fetches live quotes from Yahoo Finance via CORS proxies (corsproxy.io → allorigins → codetabs). Displays:
- **Indices**: S&P 500, TSX, FTSE 100, JSE (South Africa)
- **FX**: USD/CAD, USD/ZAR
- **Commodities**: Gold, Silver, Platinum, WTI Oil, Copper

### Brain Food
Claude writes a 2–3 sentence Farnam Street-style mental model or thinking-tool observation tied to the user's active interests.

### World at a Glance (Pulse)
Fetches live headlines from CBC World, News24, and BBC World RSS, then asks Claude to synthesise one crisp situational sentence per region: Africa, Middle East, Europe, Asia, Americas, Global/Climate. Refreshes on every load (not cached).

### Toronto Events
Tries the Luma public discover API for real Toronto events. Falls back to Claude generating 3 plausible AI/tech/startup events if the API is unavailable.

### Today's Discovery
Calls the Wikipedia Random Summary API and displays the article title and extract with a toggle. Not cached — different article on each load.

### Rabbit Hole
Claude suggests one specific thing to explore (Wikipedia article, YouTube video, documentary, podcast, or essay) tied to active interests. Clicking opens a Google search.

## User Personalisation

**Interests** (stored in state): `geopolitics · science · history · philosophy · technology · economics · literature · environment`

Active interests (default: geopolitics, science, history) are toggleable tags. They influence the quote, brain food, learning topic, rabbit hole, and news synthesis prompts. The page also tracks:
- Visit count and last visit timestamp
- Learning history (last 30 topics with tag and body excerpt)
- Quiz scores (last 30 sessions)

## RSS / CORS Strategy

All RSS fetches try three CORS proxies in order with a 6-second timeout each:
1. `corsproxy.io` — returns raw XML
2. `api.allorigins.win` — wraps response in `{ contents }`
3. `api.rss2json.com` — rate-limited JSON conversion

Market data uses the same proxy cascade against Yahoo Finance's `/v7/finance/quote` endpoint.

## State & Storage

All state lives in two `localStorage` namespaces:

| Key | Contents |
|-----|----------|
| `mypage_state` | apiKey, interests, activeInterests, learnHistory, visitCount, lastVisit, quizScores |
| `ddc_v8_<section>` | Cached HTML or object for that section, tagged with today's date string |

Cache version is hardcoded as `v8` — bumping it in the source busts all cached content across devices.

## Design

- Font: Inter (Google Fonts), weights 400 and 600
- Palette: near-black ink (`#0a0a0a`), white paper, grey mid/faint tones
- Cards (learn, quiz, rabbit hole, setup box) use an inverted dark-on-black scheme
- No JavaScript framework, no bundler — plain DOM manipulation
- Responsive via a single `@media (max-width: 768px)` breakpoint

## Target User

The content mix (CBC, National Post, News24, Daily Maverick, USD/ZAR, JSE) points to a user based in **Toronto, Canada** with ties to **South Africa**.

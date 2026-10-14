# DeepEnk

> An AI-powered universal commerce decision engine — one natural-language query, the best deal across every platform.

DeepEnk turns a plain-English request like *"Gaming laptop under ₹70k"* or
*"cheapest biryani near me"* into a ranked, explained recommendation drawn from
shopping, food, ride, travel and hospitality providers at once. A natural-language
intent engine classifies the query, fan-out fetchers pull live options from
multiple platforms in parallel, and a GPT-4 reasoning layer summarises the best
overall pick, the best value, and why — finishing with a universal checkout link.

## Why

Buying anything online means hopping between Amazon, Flipkart, Swiggy, Uber,
MakeMyTrip and a dozen others, comparing prices and offers by hand. DeepEnk
collapses that into a single query: it understands *what* you want, searches
*everywhere* that can supply it, and hands back a decision instead of a list of
tabs.

## Features

- **Natural-language intent extraction** — parses category, budget, features and
  domain (shopping / food / rides / travel / hospitality) from a free-text query.
- **Multi-platform fan-out search** — queries Amazon, Flipkart, Myntra, Swiggy,
  Zomato, Uber, Ola, MakeMyTrip, OYO and more in parallel via RapidAPI, with a
  scraper fallback.
- **GPT-4 reasoning layer** — generates "best overall" and "best value" picks,
  comparison insights, sentiment analysis and coupon validation.
- **Price-drop prediction & watch** — monitors tracked items and pushes real-time
  alerts over WebSocket (Socket.IO) when a target price is hit.
- **Universal checkout** — one-click redirection to the winning provider with
  offers pre-applied, plus click-through analytics.
- **Caching & auth** — in-memory TTL cache (NodeCache) in front of external APIs,
  Google OAuth 2.0 + JWT sessions, and MongoDB-backed search/click history.

## Architecture

DeepEnk is a React / React Native frontend over an Express + Socket.IO backend,
with an "Intelligence Core" (SearchEngine, AIService, ProductService) sitting
between the API gateway and the external provider ecosystem. State lives in
MongoDB (users, searches, clicks) with a NodeCache layer for hot results.

A full set of diagrams — request sequence, component graph, data flow, auth flow,
database ERD, price-drop state machine, route map and the WebSocket event flow —
is maintained in [`deepenk-diagrams.md`](./deepenk-diagrams.md).

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React.js (SPA), React Native |
| Backend | Node.js, Express, Socket.IO |
| AI | OpenAI GPT-4 (summaries, comparison, sentiment, prediction) |
| Data | MongoDB, NodeCache (TTL cache), Supabase (auth/profiles) |
| Auth | Google OAuth 2.0 (Passport.js), JWT (HS256, 7-day expiry) |
| Integrations | RapidAPI connectors for Amazon, Flipkart, Myntra, Swiggy, Zomato, Uber, Ola, MakeMyTrip, OYO, ONDC |

## Getting started

### Prerequisites

- Node.js 18+ and npm
- A MongoDB connection string
- API keys: OpenAI, RapidAPI, Google OAuth client ID/secret

### Setup

```bash
# 1. clone
git clone https://github.com/RaghavDabra/DeepEnk.git
cd DeepEnk

# 2. install dependencies
npm install

# 3. configure environment
cp .env.example .env     # then fill in the values below

# 4. run
npm run dev              # starts the Express + Socket.IO server
```

### Environment variables

```env
MONGODB_URI=
OPENAI_API_KEY=
RAPIDAPI_KEY=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
JWT_SECRET=
```

> Exact scripts and variable names may vary with the current source layout — check
> `package.json` and `.env.example` in the repository root.

## License

Released under the [MIT License](./LICENSE).

## Author

Built by [Raghav Dabra](https://github.com/RaghavDabra).

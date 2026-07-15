# EeshaEats

**What should I eat right now?**

EeshaEats is a food decision engine that tells you exactly what to order at nearby restaurants based on your calorie and macro targets, budget, and location. Not a tracker. Not a gym bro app. Not a meal planner. Just an answer.

Live at **[eeshaeats.vercel.app](https://eeshaeats.vercel.app)**



## The Problem

Every time my girlfriend and I want to eat out, it turns into a 20-minute debate. She'd pull up MyFitnessPal, I'd Google nutrition info, we'd do mental math on whether something fit her macros, give up, and order whatever. EeshaEats solves that.



## How It Works

1. Enter your stats or macro targets (or use Simple mode for calories only)
2. Pick a category — Meals, Treats, or Drinks
3. Drop in your ZIP code, search radius, and optional budget
4. Get a ranked list of real nearby menu items with plain-English explanations

Each result is scored 0–100 based on how closely it matches your targets across calories, protein, carbs, fat, price, and distance. The scoring weights shift by category — calories matter more for Treats, protein matters more for Meals.


## Features

- **4 calibration paths** — guided TDEE calculator, per-meal manual entry, daily totals auto-split, and simple calorie-only mode
- **Fit Score (0–100)** — weighted scoring engine with category-specific weights
- **Plain-English reasoning** — every result explains why it scored the way it did
- **Budget filtering** — items 50%+ over budget are excluded entirely
- **Distance scoring** — closer restaurants score higher
- **Dark/light mode** — white + pastel green default, black + amber dark mode
- **Nutrition disclaimers** — customizable items show variation notes
- **Get directions** — one tap opens Google Maps to the restaurant
- **localStorage persistence** — calibration inputs survive page refresh



## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14 + TypeScript + Tailwind CSS |
| Backend | Python + FastAPI |
| Geocoding | Zippopotam.us API (ZIP → coordinates) |
| Data | CSV files (chains, menu items, locations) |
| Frontend hosting | Vercel |
| Backend hosting | Railway |


# Nerd Points
## Project Structure 

```
eeshaeats/
├── CLAUDE.md                  # Full product spec and logic reference
├── backend/
│   ├── main.py                # FastAPI app, endpoints, geocoding
│   ├── calibration.py         # TDEE calculator + macro targets (4 paths)
│   ├── scoring.py             # Scoring engine + penalty bands
│   ├── reasoning.py           # Plain-English explanation generator
│   ├── filters.py             # Pre-filter logic
│   ├── models.py              # Pydantic data models
│   ├── requirements.txt
│   └── data/
│       ├── chains.csv         # Chain restaurant metadata
│       ├── menu_items.csv     # Nutrition data (per chain item)
│       └── locations.csv      # Physical locations with lat/lng
└── frontend/
├── pages/
│   ├── index.tsx          # Calibration page
│   └── results.tsx        # Results page
├── components/
│   ├── CalibrationForm.tsx
│   ├── ResultCard.tsx
│   ├── ScoreBadge.tsx
│   ├── ReasoningLayer.tsx
│   └── ThemeToggle.tsx
└── styles/
└── globals.css        # CSS variables, dual theme system
```


## Scoring Engine

Scores are calculated as:
Score = 100 − calorie_penalty − protein_penalty − carb_penalty − fat_penalty − price_penalty − distance_penalty

Category weights:

| Factor | Meals | Treats | Alcohol |
|---|---|---|---|
| Calories | 40 | 65 | 65 |
| Protein | 28 | — | — |
| Carbs | 10 | 10 | 10 |
| Fat | 7 | 10 | 10 |
| Price | 10 | 10 | 10 |
| Distance | 5 | 5 | 5 |

Penalty bands scale proportionally. Items priced 50%+ over budget are excluded entirely. Scores are capped at 100.



## Calibration Paths

| Path | Description |
|---|---|
| Auto-calculate | Full TDEE calculator using Mifflin-St Jeor. Enter age, sex, height, weight, goal, activity level. |
| My targets | Enter per-meal macro targets directly. |
| My daily totals | Enter daily totals + meals per day. System divides evenly. |
| Simple mode | Calories only. Ranks by calories, budget, and distance. |



## Data Schema

**chains.csv** — `chain_id, chain_name`

**menu_items.csv** — `item_id, chain_id, category, subcategory, item_name, calories, protein_g, carbs_g, fat_g, price_avg, source_url, source_name, source_type, last_updated, status, notes`

**locations.csv** — `location_id, chain_id, address, zip_code, lat, long, active`

Nutrition is stored once per chain item, not per location. Distance is calculated from user ZIP to location coordinates via the Zippopotam.us geocoding API.

Data status values: `verified / manual / incomplete / needs_review`



## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Returns data load status |
| POST | `/calibrate` | Takes user inputs, returns per-meal macro targets |
| POST | `/search` | Takes targets + search params, returns ranked results |



## Running Locally

**Backend**
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

**Frontend**
```bash
cd frontend
npm install
```

Create a `.env.local` file in the frontend folder:
NEXT_PUBLIC_API_URL=http://localhost:8000

```bash
npm run dev
```

Open `localhost:3000`.



## Current Coverage

RDU area, North Carolina — 7 chains, 37 menu items, 21 locations.

Chains: McDonald's, Chipotle, Subway, Starbucks, Chili's, Applebee's, Dunkin'

Expanding coverage is the main ongoing work. If you want to contribute restaurant data, see the schema above.



## Roadmap

- [ ] Expand restaurant and location coverage beyond RDU
- [ ] RAG-based menu ingestion pipeline
- [ ] Supabase migration from CSV files
- [ ] OpenAI natural language search ("something high protein under $10")
- [ ] User accounts and saved targets
- [ ] Mobile app



## License

All rights reserved. © 2026 EeshaEats.

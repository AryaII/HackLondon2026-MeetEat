# MeetEat

Multi-person collaborative group dining recommender. Each participant sets a
location, budget, cuisine preferences, dietary needs, and travel constraints;
MeetEat geocodes everyone, estimates travel times, and ranks restaurants with a
composite consensus score (geographic fairness, cuisine match, quality, and
occasion fit). The top picks get an AI-written explanation of why they suit the
group.

## Prerequisites

- Node.js 18+
- A [Google Maps Platform](https://developers.google.com/maps) API key (for `MapVisualization`)
- A GLM (Z.AI) API key

## Run locally

1. Install dependencies:
   ```sh
   npm install
   ```
2. Copy [.env.example](.env.example) to `.env.local` and fill in your keys:

   | Variable | Used by | Notes |
   | --- | --- | --- |
   | `VITE_GLM_API_KEY` | [src/utils/aiClient.ts](src/utils/aiClient.ts) | Browser-side call to Z.AI's OpenAI-compatible chat-completions API (`https://api.z.ai/api/paas/v4`, model `glm-5.1`). Must use the `VITE_` prefix — Vite only exposes prefixed vars to client code via `import.meta.env`. |
   | `VITE_GOOGLE_MAPS_PLATFORM_KEY` | [src/components/MapVisualization.tsx](src/components/MapVisualization.tsx) | Renders the participant/restaurant map. The app runs fine without it; the map shows a setup guide instead. |

3. Run the dev server:
   ```sh
   npm run dev
   ```
   Opens at http://localhost:3000.

## Available scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the Vite dev server on port 3000 |
| `npm run build` | Type-check and build a production bundle to `dist/` |
| `npm run preview` | Preview the production build locally |
| `npm run lint` | Type-check only (`tsc --noEmit`) — run this before committing |
| `npm run clean` | Remove `dist/` and the generated `server.js` |

## Project structure

```
src/
├── App.tsx                 # Screen state machine (landing → room → loading → results → winner)
├── components/
│   ├── LandingScreen.tsx   # Entry screen — start a demo or build a group from scratch
│   ├── RoomScreen.tsx      # Add/edit participants, pick occasion & options
│   ├── ParticipantModal.tsx# Add/edit a single participant's preferences
│   ├── LoadingScreen.tsx   # Analysis animation while recommendations are computed
│   ├── ResultsScreen.tsx   # Top-3 ranked restaurants with score breakdowns & AI reasons
│   ├── WinnerScreen.tsx    # Final pick after the group votes
│   └── MapVisualization.tsx# Google Maps view of participants & candidate restaurants
├── data/restaurants.ts     # Mock London restaurant dataset + postcode → lat/lng resolution
├── utils/
│   ├── scoring.ts          # Haversine distance, travel-time estimates, consensus scoring
│   ├── aiClient.ts         # GLM (Z.AI) client that writes natural-language recommendation reasons
│   └── i18n.ts             # English/Chinese translation strings
└── types.ts                # Shared TypeScript types (Participant, Restaurant, ScoringBreakdown, …)
```

## How recommendations work

1. **Hard filtering** ([scoring.ts](src/utils/scoring.ts)) drops restaurants that
   violate anyone's travel-time limit, budget, minimum rating, disliked cuisine,
   or hard dietary/religious requirements.
2. **Composite scoring** ranks the rest on four weighted components: geographic
   fairness (commute equality), cuisine match, restaurant quality, and occasion
   suitability — each producing a templated English reason as a safe fallback.
3. **AI enrichment** ([aiClient.ts](src/utils/aiClient.ts)) sends that scoring
   breakdown to GLM (`glm-5.1` via Z.AI's OpenAI-compatible API) to write a more
   natural explanation grounded in the same facts. If `VITE_GLM_API_KEY` is
   missing or the request fails, the UI silently keeps the templated reason —
   the AI step never blocks the results screen.

[View MeetEat Prototype](https://htmlpreview.github.io/?https://github.com/AryaII/HackLondon2026-MeetEat/blob/main/meeteat-prototype.html)

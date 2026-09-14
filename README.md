# JABO — Smart Routes. Better Journeys.

Premium Bangladesh-focused smart mobility web app + PWA. Plan journeys across bus, metro, railway, launch, and walking using **local DEMO/SEED data** plus optional free OSM/Nominatim/OSRM providers.

Paid APIs are **never required**. Missing `TRAFFIC_API_KEY`, `AI_API_KEY`, Google OAuth, or Redis does not crash the app.

## Architecture

| Layer | Stack | Default port |
| --- | --- | --- |
| Frontend PWA | Next.js 16, TypeScript, Tailwind CSS 4, TanStack Query, Zustand, MapLibre GL | 3000 |
| API | NestJS, REST, Socket.IO | 4000 |
| Database | PostgreSQL + PostGIS | 5432 |
| Cache | Redis if `REDIS_URL` is set, otherwise in-memory | optional |

Official branding uses the uploaded JABO logo at `apps/web/public/brand/jabo-logo.png` without recoloring.

## Local development (zero paid services)

### 1. Start PostgreSQL (PostGIS)

```bash
docker compose up -d postgres
```

### 2. API

```bash
cd apps/api
copy .env.example .env   # Windows: already includes local defaults
npm install
npx prisma generate
npx prisma db push
npm run prisma:seed
npm run start:dev
```

API: http://localhost:4000/health

Seeded accounts (change in production):

- Admin: `admin@jabo.app` / `Admin1234!`
- Rider: `demo@jabo.app` / `Demo1234!`

### 3. Web

```bash
cd apps/web
copy .env.example .env.local
npm install
npm run dev
```

Open http://localhost:3000

Splash (logo + tagline + MINHAZ STUDIO) shows once per browser session.

Guest From/To search works without login. Try **Farmgate** → **Motijheel**.

## Environment

See `apps/api/.env.example` and `apps/web/.env.example`.

Required for a real install: `DATABASE_URL`, `JWT_SECRET`, `JWT_REFRESH_SECRET`.

Optional with fallbacks:

- `REDIS_URL` → in-memory cache
- `TRAFFIC_API_KEY` → “Real-time traffic data unavailable.”
- `AI_API_KEY` → local rule-based route assistant
- `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` → email/password still works
- Nominatim / OSRM public endpoints → local geocoding & estimated geometry

Never commit production secrets. Never put keys in the frontend.

## Data honesty

Transport records are labelled **DEMO / SEED / ESTIMATED / LIVE / UNAVAILABLE**.

Live vehicle GPS is **not faked**. Until an operator posts to `POST /vehicles/:id/location`, the UI shows:

> Live vehicle location is currently unavailable.

## Deployment (free tiers)

1. **Database** — Neon PostgreSQL (enable PostGIS). Run `npx prisma db push` and `npm run prisma:seed` from `apps/api`.
2. **API** — Render (`render.yaml`). Set `DATABASE_URL`, `JWT_SECRET`, `JWT_REFRESH_SECRET`, `FRONTEND_URL`.
3. **Web** — Vercel, root directory `apps/web`. Set `NEXT_PUBLIC_API_URL` to the Render URL.
4. **Redis** — Upstash only if you want shared cache; skip otherwise.
5. **GitHub** — push this repository.

### Production API start

```bash
cd apps/api
npx prisma generate
npx prisma db push
npm run prisma:seed
npm run build
npm run start:prod
```

### Production web

```bash
cd apps/web
npm run build
npm run start
```

## Main product flow

Home → From / To → Plan journey → compare ranked alternatives → active trip steps → optional stop alert → trip history.

## Admin

Sign in as admin → `/admin` for users, operators, vehicles, routes, stops, places, fares, live vehicles, reports, verification, alerts, CMS, analytics, settings.

## PWA

Installable via Web App Manifest + `public/sw.js`. Offline: app shell, cached pages, static DEMO transport. Live GPS/traffic/ETA stay unavailable offline.

## Real-time Traffic and GPS

JABO now supports two real live-data paths:

1. **Real-time traffic overlay:** configure `NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN` in the web app. JABO uses the Mapbox Traffic v1 vector tileset and styles its `traffic` layer by the live `congestion` field. Mapbox documents Traffic v1 as a continuously updating traffic dataset, with speed/density updates approximately every 8 minutes.
2. **Traffic-aware route ETA:** configure `TRAFFIC_API_KEY` or `GOOGLE_MAPS_API_KEY` in the API. The backend calls Google Routes API with `TRAFFIC_AWARE`, returning live traffic-aware duration, static duration, delay, and a congestion classification.
3. **Real vehicle GPS:** an authenticated ADMIN/OPERATOR opens `/operator/tracking` on the vehicle driver's phone, selects the assigned vehicle, grants browser geolocation, and starts GPS sharing. Coordinates are stored in `vehicle_locations` and broadcast through Socket.IO to live maps.

No GPS coordinates are simulated. If no operator device is publishing, the UI explicitly reports that live GPS is unavailable.

### Required live-data configuration

Web `.env.local`:

`NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN=pk.your_public_mapbox_token`

API `.env`:

`TRAFFIC_API_KEY=your_google_routes_api_key`

The Google Cloud project must enable the Routes API. Do not commit secrets. The Mapbox browser token should be a public `pk...` token with the minimum required access.

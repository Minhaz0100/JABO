# JABO — Final Realtime Feature Fixes

This package keeps the existing JABO feature set and fixes the main missing realtime pieces without replacing the existing transport/planner architecture.

## Fixed

### 1. Real-time traffic
- Backend uses Google Routes API with `TRAFFIC_AWARE` when `GOOGLE_MAPS_API_KEY` or `TRAFFIC_API_KEY` is configured.
- Planner requests live origin → destination traffic data.
- Live delay/congestion is applied to road-based journey legs before route scoring, so fastest-route ranking can change with current traffic.
- Route responses expose `trafficImpact` and per-route traffic delay/congestion.
- Home route cards show live road delay when available.
- Map traffic overlay uses Mapbox Traffic v1 when `NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN` is configured.
- Provider/API failures return `UNAVAILABLE` instead of breaking journey planning.

### 2. Real GPS vehicle tracking
- Authenticated ADMIN/OPERATOR users can select an assigned vehicle.
- Browser/device `navigator.geolocation.watchPosition()` sends actual coordinates to the backend.
- Backend validates coordinates and operator ownership before saving.
- GPS locations are broadcast through Socket.IO.
- Live vehicle map displays the latest position per vehicle.
- Stale vehicle positions older than two minutes are not treated as live.
- No simulated/random GPS coordinates are generated.

### 3. Additional fixes
- Trip start/complete endpoints now enforce trip ownership.
- Active-trip stop alert now calculates distance to the destination and notifies the user within approximately 500 m.
- Live vehicle API returns only the latest position per vehicle.
- Operator/Admin navigation includes the Live GPS page.
- Admin traffic configuration status recognizes both traffic environment variable names.

## Required configuration

API `.env`:

```env
TRAFFIC_PROVIDER=google-routes
GOOGLE_MAPS_API_KEY=your_google_routes_api_key
# or
TRAFFIC_API_KEY=your_google_routes_api_key
```

Web `.env.local`:

```env
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_WS_URL=http://localhost:4000
NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN=pk.your_public_mapbox_token
```

The Google Cloud project must have the Routes API enabled. The Mapbox token is only used in the browser for the traffic map overlay.

## GPS test

1. Start the API and web app.
2. Sign in as an ADMIN or OPERATOR.
3. Assign a vehicle to an operator from Admin → Vehicles.
4. Open `/operator/tracking` on the operator's phone.
5. Select the assigned vehicle.
6. Allow location permission and press **Start Live GPS**.
7. Open `/admin/live` or Explore on another device/browser.
8. The vehicle marker should update from the phone's real GPS coordinates.

## Validation note

The final source and ZIP were integrity-checked. A full dependency installation/build could not be completed in the packaging environment because `npm ci` timed out, so this package does **not** claim a successful production build from this environment.

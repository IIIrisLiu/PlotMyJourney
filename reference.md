# PlotMyJourney Reference

## Execution checklist

```
Task Progress:
- [ ] Resolve hotel + all attractions (google-maps-official: search_places, resolve_names)
- [ ] Verify hours, holidays, and ticketing (Fetch/Web Search MCP)
- [ ] Identify hard anchors (pre-booked slots, last entry times)
- [ ] Check weather/disruptions for target date
- [ ] Compute route legs (DRIVE: compute_routes; TRANSIT/WALK: Routes API via fetch-layer)
- [ ] Sequence attractions (anchors, hours, weather, geography)
- [ ] Insert 12:00–1:30 PM lunch near current/preceding stop
- [ ] Add 10-minute J-Type buffer on every transit leg
- [ ] Confirm closed loop: Hotel → … → Hotel
- [ ] Emit mandatory output format (Sequence, Timeline, Transit, Pro-Tips)
```

## Routes API: TRANSIT and WALK (fetch-layer)

When `travelMode` is `TRANSIT` or `WALK`, POST directly to the Routes API. **Do not** include `routingPreference` in the body.

**Endpoint:** `https://routes.googleapis.com/v2:computeRoutes`

**Headers:**
- `Content-Type: application/json`
- `X-Goog-Api-Key: <GOOGLE_MAPS_API_KEY>`
- `X-Goog-FieldMask: routes.duration,routes.distanceMeters,routes.legs.steps.transitDetails,routes.legs.steps.navigationInstruction,routes.legs.duration`

**Example payload (TRANSIT):**

```json
{
  "origin": {
    "location": {
      "latLng": { "latitude": 35.6719, "longitude": 139.7650 }
    }
  },
  "destination": {
    "location": {
      "latLng": { "latitude": 35.7101, "longitude": 139.8107 }
    }
  },
  "travelMode": "TRANSIT",
  "departureTime": "2026-07-01T09:00:00+09:00",
  "languageCode": "en-US",
  "units": "METRIC"
}
```

**Example payload (WALK):** Same structure with `"travelMode": "WALK"`.

Parse `routes[0].duration` and `routes[0].distanceMeters`; add the 10-minute J-Type buffer to displayed transit time.

## Venue validation prompts (Fetch/Web Search)

For each attraction, search for:
- Official site hours for the travel date (weekday vs weekend, holidays)
- Last admission / last entry time (not just closing time)
- Ticket or reservation requirements and whether the user already has a slot

Treat confirmed pre-booked slots as immovable anchors in the Master Timeline.

## Weather reshuffle heuristics

| Signal | Action |
|--------|--------|
| Rain in afternoon | Outdoor stops → morning; indoor → rainy window |
| Heavy congestion / closure | Recompute affected legs; prefer alternate mode or reorder if times still fit |
| Venue early closure | Drop or swap; re-anchor remaining stops around hard bookings |

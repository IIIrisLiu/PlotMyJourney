# PlotMyJourney (Plotter)
### Autonomous J-Type Concierge Agent for Stress-Free Travel

> *"Not navigation — autonomous re-planning when the day breaks."*

---

## The Problem

For structured, planning-oriented travelers, itinerary planning is a fragmented, high-anxiety process long before the trip even begins. The tools that exist today each solve only a slice of the problem: Google Maps calculates raw distances but contains no scheduling logic; social media platforms surface attractive spots but ignore geographical efficiency entirely; and general-purpose LLMs, while conversational, hallucinate travel times and lack real-time awareness of the world outside their training data. No single tool bridges personalized scheduling logic with live spatial and environmental awareness — and that gap has real consequences.

The deeper failure becomes most visible when something goes wrong mid-trip. When a sudden downpour rolls in at 2 PM or a road closure forces a reroute, travelers are left manually juggling three or four apps simultaneously — re-mapping locations, recalculating timelines, and rebuilding plans from scratch — all while standing in the rain. For travelers who derive comfort and genuine enjoyment from a well-structured itinerary, this kind of forced improvisation doesn't just create inconvenience; it destroys the emotional serenity the trip was designed to provide.

---

## The Solution

**PlotMyJourney (Plotter)** is an autonomous concierge agent built to handle single-day, multi-destination itinerary optimization with dynamic, human-in-the-loop adaptation. Rather than acting as a passive chatbot, Plotter functions as an executive planner — holding the user's full context, reasoning about constraints, and taking initiative when conditions change.

The agent's reasoning is grounded in real-world data from the start. Plotter uses the Google Maps MCP for place discovery and geocoding, while computing routes either through the Maps MCP or via direct Google Routes API calls through a fetch-layer MCP, depending on the travel mode. This ensures that routing data is derived from live infrastructure rather than statistical pattern completion from training data.

---

## Architecture

```
User Request
     ↓
LLM Agent (The Brain)
Spatial-temporal reasoning + empathetic communication
     ↓
Google Maps MCP (The Eyes)
Live geocoding + real-time place data
     ↓
Travel Skill Evaluation (The Soul)
Closed-loop routing + J-Type buffers + weather guardrails
     ↓
Optimized Itinerary Output
     ↑
Environmental Disruption (re-plan trigger loop)
```

**Three Pillars:**
- **The Brain** — Executes complex spatial-temporal reasoning across destinations and time constraints
- **The Eyes (Google Maps MCP)** — Continuously supplies real-time, ground-truth geographical data
- **The Soul (Travel Skill)** — Enforces deterministic planning constraints the model cannot override

---

## Key Concepts Demonstrated

| Concept | Implementation |
|---|---|
| **MCP Server** | Google Maps Official MCP + Fetch MCP for real-time routing and data |
| **Agent Skills** | Travel Skill with J-Type buffer rules, closed-loop routing, weather guardrails, tool routing rules |
| **Autonomous Reasoning** | Multi-step re-planning: disruption detection → constraint evaluation → sequence rebuild |
| **Security** | API keys managed via environment variables, never hardcoded in committed files |

---

## Travel Skill Rules

The Travel Skill enforces deterministic constraints the agent cannot override:

- **Loop Boundary Principle** — Every itinerary must form a perfect closed loop, starting and returning to the user's hotel
- **J-Type Buffer Rule** — Mandatory +10 min buffer on every transit leg, explicitly displayed in the timeline
- **Meal Integration Principle** — Lunch locked to 12:00–13:30, constrained to the neighborhood of the current attraction
- **Autonomous Real-Time Validation** — Fetch/Web Search MCP must verify operational hours and ticketing availability before finalizing any sequence
- **Weather-Adaptive Guardrails** — Rain trigger reshuffles outdoor attractions to morning; indoor attractions fill rainy slots
- **Disruption Trigger** — Road closures or transit delays trigger immediate rerouting with justified sequence changes

---

## MCP Configuration

Copy `mcp_config.example.json` and add your API key in your local MCP config (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "google-maps-official": {
      "url": "https://mapstools.googleapis.com/mcp",
      "headers": {
        "X-Goog-Api-Key": "YOUR_GOOGLE_MAPS_API_KEY"
      }
    },
    "fetch-layer": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

**Prerequisites:**
- [Cursor IDE](https://cursor.com) or any MCP-compatible agent environment
- [uv](https://github.com/astral-sh/uv) — `pip install uv`
- Node.js + npx
- Google Maps API Key ([Get one here](https://console.cloud.google.com/apis/credentials)) with Maps Grounding Lite API enabled

**Skill Setup:**

The Travel Skill is at `.agents/skills/PlotMyJourney/SKILL.md`. In Cursor, it auto-triggers when you mention trips, routes, multiple destinations, or itinerary changes. Invoke explicitly with `@plot-my-journey`.

---

## Security

API keys are never hardcoded in any committed file. `mcp_config.example.json` contains only placeholder values. Your real key stays in `~/.cursor/mcp.json` — outside the project directory and never committed to version control.

---

## Demo Scenarios

Three sequential scenarios validate the framework against a full-day Tokyo trip from a Ginza hotel:

**Scenario 1 — Plan A (Baseline Optimal Routing)**
Input: Full-day itinerary request for Asakusa Temple, Ueno Park, and Tokyo Skytree from Ginza.
Plotter generates a closed-loop, weather-aware schedule with all J-Type buffers applied and venue hours validated via MCP.

**Scenario 2 — Plan B (Rain Trigger)**
Input: Heavy rain begins at 1:30 PM mid-itinerary.
Plotter activates weather-adaptive reasoning, identifies Ueno Park as high-exposure risk, restructures the remaining sequence to prioritize indoor alternatives, and re-validates closing time constraints — without dropping any destination.

**Scenario 3 — Plan C (Transit Disruption)**
Input: Tokyo Metro Ginza Line delayed 30 minutes at 3:30 PM.
Plotter evaluates the delay against downstream hard constraints (Sensō-ji 17:00 closure), bypasses the affected metro line, switches to alternative surface transit, and recomputes the remaining itinerary.

---

## Challenges & Solutions

**Challenge:** WALK and TRANSIT routing consistently triggered `400 INVALID_ARGUMENT` errors. The MCP layer was unconditionally injecting a `routing_preference` parameter incompatible with Google's Routes API for non-driving modes.

**Solution:** The Travel Skill was redesigned to enforce explicit tool-routing separation:
- **Location search & resolution** → always via `google-maps-official` (`search_places`, `resolve_names`)
- **DRIVE routing** → `google-maps-official.compute_routes`
- **TRANSIT / WALK routing** → bypasses MCP routing layer; directly invokes Routes API via `fetch-layer` with no `routingPreference` field

This eliminates the faulty parameter injection at its source rather than relying on downstream error handling.

---

## Future Work

- **Richer real-time validation** — Dynamic ticket inventory, last-minute reservation drops, venue capacity constraints
- **Calendar MCP integration** — Push finalized itineraries directly into Google Calendar or Apple Calendar as structured events
- **Travel Personality Slider** — Dial between Extreme Planner and Loose Explorer to dynamically adjust buffer times and sequencing strictness

---

*Built for the AI Agents: Intensive Vibe Coding Capstone Project (Kaggle × Google, 2026)*

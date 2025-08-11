# 🍪 Cookiemon: Design Spec v1.0

## Summary
**Cookiemon** is a digital pet whose entire life is stored inside a single, human-readable browser cookie. It cannot die (unless the cookie is deleted), but can become **stale** or **embittered** through neglect. It remembers only its **first times** — forming a curated memory of emotional milestones.

---

## Cookie Data Schema

```json
{
  "name": "Crumbsy",
  "species": "snickerdoodles",
  "color": "#BADA55",
  "initializationDay": 1754000000000,
  "personality": "sassy",
  "lastFed": 1754020000000,
  "lastPlayed": 1754004000000,
  "memories": [
    {
      "timestamp": 1754010000000,
      "event": "firstFed",
      "detail": "https://en.wikipedia.org/wiki/HTTP_cookie"
    },
    {
      "timestamp": 1754003000000,
      "event": "firstPlay",
      "detail": "mahjong"
    }
  ]
}
```

- Total size target: < 4KB
- Stored as a single JSON string in a cookie named `cookiemon`

---

## Derived State (Not Stored in Cookie)

These are computed at runtime using timestamps and logic:

| Field       | Derived From                                |
|-------------|----------------------------------------------|
| `age`       | `Date.now() - initializationDay`            |
| `hunger`    | `Date.now() - lastFed`                      |
| `mood`      | Time since lastFed/lastPlayed + personality + memories |
| `energy`    | Inversely proportional to inactivity        |
| `status`    | Interpreted from time-based neglect         |

---

## Memory System

### Memory Rules
- Only first-time experiences are stored
- No duplicates
- Stored in `memories` array as `{ timestamp, event, detail }`

### Example Memory Events

| Event              | Trigger Condition                             | Example Detail                |
|--------------------|-----------------------------------------------|-------------------------------|
| `firstWord`        | First message sent by user                    | "Hey Crumbsy!"                |
| `firstFed`         | First feed interaction                        | "https://en.wikipedia.org/wiki/HTTP_cookie"|
| `firstPlay`        | First play interaction                        | "mahjong"         |
| `firstStale`       | First time pet is left alone >6h              | "You left me alone 7h..."     |
| `firstEmbittered`  | First time pet is neglected >24h              | "I realized I'm on my own."   |
| `firstLaugh`       | First time pet’s mood becomes “happy”         | "You made me giggle"          |
| `firstMilestone`   | Reaching 1 day.                               | "I turned 1 day old!"         |
| `firstOutburst`    | First grumpy/cranky mood                      | "I threw a crumb"             |
| `firstForgiveness` | First emotional recovery after embitterment   | "I forgave you. We’re okay."  |

---

## Status Logic

| Status        | Time Since Last Update     | Description                              |
|---------------|----------------------------|------------------------------------------|
| `fresh`       | < 6 hours                  | Cookiemon is happy and energetic         |
| `stale`       | 6–24 hours                 | Cookiemon is bored, slower responses     |
| `embittered`  | > 89 hours                 | Cookiemon is resentful, grumpy, sarcastic|
| `forgotten` (meta) | if cookie deleted     | Cookiemon ceases to exist                |

---

## Personality Types (Assigned Randomly at Creation)

| Type     | Traits                                         |
|----------|------------------------------------------------|
| Sassy    | Snarky replies, hates being ignored           |
| Sweet    | Forgives easily, loves attention              |
| Chill    | Doesn’t mind neglect, low-maintenance         |
| Dramatic | Overreacts to small things, big mood swings   |
| Grumpy   | Hard to please, rewards consistency           |

Each type affects **mood responses, dialogue tone, and memory reactions**.

---

## Utilities (To Implement)

- `initializePet()` – Creates and saves the Cookiemon cookie
- `loadPet()` – Loads from cookie and calculates derived state
- `savePet()` – Saves pet object to cookie
- `addMemory(event, detail)` – Adds a memory if not already logged
- `getMood()` – Returns dynamic mood based on time/personality
- `getStatus()` – Returns fresh/stale/embittered based on neglect
- `debugPanel()` – Optional visualizer for ~nerds~ bakers

---

## Developer Tips

- Keep cookie **under 4096 bytes**
- Use `encodeURIComponent(JSON.stringify(obj))` for storage
- Be sure to `.find()` or `.some()` to avoid memory duplicates
- Let ~nerds~ bakers view/edit cookie if they want — part of the charm

---

## Feeding & Traveling via the Web

Cookiemon interacts with the broader web in two meaningful ways:

### Feeding via Referrer

If the user visits the main Cookiemon page (`cookiemon.html`) from another website, Cookiemon treats that referrer as food.

**Logic:**
- Use `document.referrer` to detect external visits
- Trigger feeding based on hostname
- Example: "Nourished by wikipedia.org"

**Memory Example:**
```json
{ "timestamp": 1723490000000, "event": "firstWebFeed", "detail": "https://en.wikipedia.org/wiki/HTTP_cookie" }
```

### Traveling via Care Points

Special `cookiemon-travel.html` pages embedded in external sites act as "travel pings" that log Cookiemon's journey across the web.

**How it works:**
- Page loads and reads the `cookiemon` cookie
- Adds a `visitedSite_*` memory if not already present
- Can be embedded on any site as an invisible `<iframe>`

**Memory Example:**
```json
{ "timestamp": 1723500000000, "event": "visitedSite_1rg.space", "detail": "Visited 1rg.space" }
```

These visits contribute to Cookiemon's sense of exploration and can be used to trigger unique dialogue, achievements, or emotional responses.

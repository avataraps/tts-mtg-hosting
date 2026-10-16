# TTS MTG Lobby Beacon: Project Brief

## Goal

Make it easy to find public Magic: The Gathering games in Tabletop Simulator (TTS).

TTS has a built-in server browser, but it's generic and hard to filter for MTG. This project adds two things:

1. **Lobby Beacon**: a standalone scripted TTS object that a host drops onto any MTG table (Malixx's MTG Deluxe Table, Oops I Baked a Pie's tables, etc.). While the game runs, it reports the table's status to a backend.
2. **Listing service**: a backend plus a simple public web page (and optionally a Discord bot) that shows live tables with filters.

Players still join through TTS's own server browser (by searching the server name) or a Steam invite. TTS Lua cannot join or switch servers, so the listing only points people to the right server.

## Hard constraints

- **Standalone, original code only.** Don't copy scripts or assets from other Workshop mods (Malixx, Omes, Oops I Baked a Pie, rikrassen, Amuzet, etc.). The beacon is its own Workshop item and must work on any table.
- **No Wizards of the Coast IP** in the beacon: no card images, set symbols or logos.
- **TTS networking:** `WebRequest` (get/post/put/custom) only runs on the game host's machine. Lua runs on the host, so this works naturally, but don't design anything that needs a client to make requests.
- **Lua scripts are stored inside the save/object JSON.** The beacon's script lives on the object itself (object script, using `self`), not in the Global script, so it doesn't conflict with the table's own Global script.

## Part 1: Beacon object (Lua)

### Host-facing UI on the object
Use XML UI or `self.createButton` / `self.createInput`, whichever is more robust:
- **Server name** text field. The host types the exact name they gave their TTS server, so players can search for it. Don't assume Lua can read the server name; verify against the API docs before relying on it.
- **Format** selector: Commander, Standard, Modern, Pioneer, Pauper, Legacy, Draft/Cube, Other.
- **Power level / bracket** selector for Commander (1–5 plus "casual / any").
- **Max seats**: 2–8.
- **Language**: free text or a short list.
- Optional **note**, max ~140 chars, e.g. "new players welcome", "no infinite combos".
- **Go public / Stop listing** toggle, plus a small status line ("Listed ✓ last ping 12s ago" or an error).

### Behaviour
- When listing starts, generate a random session token (keep it in object state). The backend issues a listing id on the first POST.
- Send a **heartbeat** every ~60s with `Wait.time` (repeating): seats filled (count seated players via `Player.getPlayers()` / seated colors), max seats, format, power level, language, note, server name, host display name (`steam_name` of the host player), and beacon version.
- On stop, on `onDestroy`, and where possible on unload, send a delete. Don't rely on it, since the backend must also expire stale listings.
- Persist settings with `onSave` / `script_state` so a reloaded table remembers them.
- Handle request failures gracefully: show the error on the object, back off, never spam chat.
- Collect no Steam IDs or other personal data beyond the host display name the TTS server browser already shows.

### Deliverables
- `beacon/beacon.ttslua`: the object script.
- `beacon/beacon.xml` if using XML UI.
- `beacon/README.md`: how to create the physical object in TTS (e.g. a custom token or tile), attach the script, save it as a Saved Object, and upload it to the Workshop. Also how a host spawns it onto any table.

## Part 2: Backend and listing page

**Stack:** open decision. Ask the user before choosing. Reasonable options are Cloudflare Workers + D1/KV, or Node (Fastify/Hono) + SQLite on a small VPS. Keep it cheap and simple.

### API (JSON)
- `POST /listings`: create. Returns `{id, token}`.
- `PUT /listings/:id`: heartbeat/update. Requires the token.
- `DELETE /listings/:id`: requires the token.
- `GET /listings`: public. Filters: `format`, `power`, `language`, `has_open_seat`.

### Rules
- Listings expire automatically if no heartbeat arrives for ~3 minutes.
- Validate and length-limit every field; strip control characters; reject links in notes or render them inert.
- Rate-limit by IP and token.
- Basic abuse handling: a profanity filter on free-text fields and an admin way to remove a listing.

### Listing page
- One mobile-friendly page that polls `GET /listings`. Show cards with server name (with a copy button), format, power level, seats `3/4`, language, note, and age.
- Include short instructions for joining: open TTS → Multiplayer → Server Browser → search the server name.

### Optional, later
- Discord bot or webhook that posts and updates live tables in a channel.

## Testing

Claude Code can't run TTS. The user will load the beacon in TTS and report results.

- For faster iteration, TTS exposes a local External Editor API (the official Lua editor plugins use it) for pushing scripts into a running game. A small push script is optional.
- Backend: unit tests for validation, expiry and token checks, plus a local mock script that simulates several beacons sending heartbeats.

## Suggested order
1. Confirm the backend stack with the user.
2. Backend API with expiry, plus tests and a mock beacon.
3. Beacon Lua against a local/dev backend. The user tests it in TTS.
4. Listing page.
5. Deploy, then Workshop upload instructions.

## References
- TTS Lua API: https://api.tabletopsimulator.com/
- WebRequest: https://api.tabletopsimulator.com/webrequest/manager/

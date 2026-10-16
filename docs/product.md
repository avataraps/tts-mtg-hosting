# Product Overview

> Source: `SPEC.md` (project brief). No code exists yet; this reflects intended
> scope. Items marked _(needs product-owner confirmation)_ are inferred.

## Product Purpose
TTS MTG Lobby Beacon makes public Magic: The Gathering games in Tabletop
Simulator (TTS) easy to find. TTS's built-in server browser is generic and
hard to filter for MTG. Hosts can't easily advertise format, power level or
open seats, and players can't easily find a matching table.

The product has two parts. The **Lobby Beacon** is a standalone scripted TTS
object that a host drops onto any MTG table; while the game runs it reports
the table's status to a backend. The **Listing service** is a backend and
public web page that shows live tables with filters. Players still join
through TTS's own server browser (by searching the server name) or a Steam
invite, because TTS Lua cannot join or switch servers. The listing only points
people to the right server.

## Target Users
- **Hosts.** Players running a public MTG game in TTS. They want to advertise
  their table (format, bracket, seats, language, house rules) with minimal
  setup, on whatever table mod they already use.
- **Players looking for a game.** They want to quickly find an open table
  that matches their format and power level, from desktop or mobile, then join
  via the TTS server browser.
- **Admins/moderators.** They need to remove abusive listings and keep the
  public page clean.
- **Discord communities** _(optional, later)_. They want live tables posted
  to a channel.

## Key Features
1. **Lobby Beacon object.** A Workshop item with its own object script (no
   Global script conflicts). It works on any MTG table and holds host-facing
   UI for server name, format, Commander bracket, max seats, language and note.
2. **Go public / Stop listing.** The host toggles the listing and sees a
   status line (last ping or error).
3. **Heartbeats.** About every 60s the beacon reports seats filled, settings,
   host display name and beacon version. Settings persist across reloads via
   `script_state`.
4. **Listing API.** Create/update/delete listings with token auth, plus a
   public filtered `GET /listings`. Stale listings expire after about 3 minutes
   without a heartbeat.
5. **Public listing page.** A mobile-friendly page of live table cards that
   shows server name with a copy button, format, power, seats, language, note
   and age. It also has join instructions.
6. **Abuse handling.** Field validation and length limits, rate limiting,
   inert links, a profanity filter, and admin removal.
7. **Discord bot/webhook** _(optional, later)_. Posts and updates live tables.

## Business Objectives
_(needs product-owner confirmation; this is a community tool, not a
commercial product)_
- Make it fast for MTG players in TTS to find a public game at the right
  format and power level.
- Get hosts to adopt the beacon across popular table mods, with no
  per-table integration work.
- Run at very low hosting cost and with little operational effort.

## Success Metrics
_(all targets TBD with product owner)_
- Number of concurrently listed tables, and unique hosts per week.
- Share of listings with an open seat that fill up (seats-filled trend).
- Listing-page visits and server-name copy clicks.
- Beacon Workshop subscriptions.
- Stale listings shown (should be near zero once expiry works).
- Monthly hosting cost.

## Product Principles
1. **Standalone and original.** No code or assets copied from other
   Workshop mods. The beacon must work on any table.
2. **No Wizards of the Coast IP.** No card images, set symbols or logos in
   the beacon.
3. **Host-only networking.** Only the game host makes requests, because
   `WebRequest` runs only on the host. Nothing depends on clients.
4. **Minimal personal data.** No Steam IDs. The only identifier collected is
   the host display name, which the TTS server browser already shows.
5. **Self-healing listings.** Never rely on delete-on-exit. The backend
   expires stale listings.
6. **Quiet and resilient.** Failed requests show on the object and back
   off. They never spam chat.
7. **Cheap and simple.** Prefer the smallest stack that does the job.

## Monitoring & Visibility
Planned only: the beacon's on-object status line (last ping or error) for
hosts, and an admin removal path. Production observability (logs, metrics,
alerting) depends on the backend stack, which is still an open decision
(Cloudflare Workers + D1/KV vs. Node + SQLite on a VPS). _(needs
product-owner confirmation)_

## Future Vision
Ship the core loop first: beacon → API → listing page. Then deploy and
publish the beacon to the Workshop.

### Potential Enhancements
- Discord bot or webhook that posts and updates live tables (in the brief as
  optional/later).
- A push script for TTS's External Editor API to speed up beacon iteration.
- _(speculative, needs roadmap confirmation)_ More filters or formats on
  the listing page, based on how hosts use it.

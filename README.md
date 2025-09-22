Town Survival Online — Full Design Document
Title
Town Survival Online
One-line pitch
A browser-based 3D town (Three.js) where players explore businesses populated from a JSON feed — and survive recurring 18-minute zombie outbreaks that sweep the streets.
Table of contents
1. Executive summary
2. Gameplay design
3. World & level design (map sketch + LOD)
4. Business data model (JSON schema + examples) 5. Technical architecture
6. Zombie event system (timers, AI, balancing)
7. UI / UX wireframes
8. Multiplayer & networking
9. Tools for content authors / admin
10. Art & asset pipeline
11. Monetization and analytics
12. Roadmap & milestones
13. Appendix: code snippets and algorithms
1. Executive summary
Goal: deliver a lightweight, multiplayer-capable, browser 3D experience built with Three.js where businesses are dynamically injected from JSON, interiors are explorable, and emergent fun is created by a repeating timed zombie event every 18 minutes.
Target platforms: Desktop browsers (primary), mobile browsers (optimised fallback).
Core user flows: explore town → enter business → view business info → roam streets → survive zombie
outbreak → earn rewards.
Success metrics: average session length, retention (DAU/MAU), sponsor conversion rate for businesses.
   1

2. Gameplay design Player controls
• Third-person camera (orbit + collision) with WASD or arrow keys + mouse for camera. • Sprint (Shift), Jump (Space), Interact (E), Use item (F).
• Enter/exit building triggers when player crosses a door collider.
Objectives
• Primary: Explore town + discover businesses.
• Secondary: Survive zombie outbreaks, collect rewards, leaderboard progression.
Progression & rewards
• Survive an outbreak → XP and "survivor coins".
• Discovering businesses (first-visit bonus).
• Repeat visits increase business "rating" (engagement metric).
Interaction in businesses
• Info board UI panel with business name, description, hours, website link, and promotional image.
• Optional short quiz or coupon (sponsor-specific) to increase engagement.
Difficulty & balancing
• Zombie count scales with number of active players (if multiplayer) and town size.
• Outbreak frequency: Fixed cadence — every 18 minutes. Duration: default 2.5 minutes
(configurable).
3. World & level design Town layout (high-level)
• Central plaza with park & clocktower.
• Commercial avenue (cafes, shops).
• Residential blocks and office quarter.
• Industrial edge and small subway/metro entrance.
ASCII map sketch (top-down simplified)
N
W [Park] E
||
[House]--[Main St]--[Shops]--[Office]
|| [Subway] [Plaza]
  2

Enterable building design
• Each building has a ground-floor interior prefab and 0..N floors (optional).
• Interiors are lightweight: a few interactable props + info board.
• Use LOD: high detail when close / inside, lower detail for exterior and far away.
Performance strategies
• Merge static geometry where possible; GPU instancing for repeating props.
• Use texture atlases for UI and small props.
• Culling and visible-chunk streaming: load interiors only when player is near or door opened.
4. Business data model (JSON schema) JSON schema (draft)
  {
    "businesses": [
{
"id": "string",
"name": "string",
"type": "string",
"description": "string",
"location": { "x": number, "y": number, "z": number }, "buildingModel": "string (glb)",
"interiorModel": "string (glb, optional)",
"logo": "string (png)",
"website": "string (url, optional)",
"hours": "string (optional)",
"sponsorId": "string (optional)",
"promotion": { "type": "coupon" | "message", "value": "string" }
} ]
}
Example entry
  {
    "id": "joes-coffee",
    "name": "Joe's Coffee",
    "type": "Cafe",
    "description": "Local roastery serving espresso and pastries.",
    "location": { "x": 12.4, "y": 0, "z": -5.2 },
    "buildingModel": "models/cafe_exterior.glb",
    "interiorModel": "models/cafe_interior.glb",
    "logo": "assets/logos/joescoffee.png",
    "website": "https://joescoffee.example",
    "hours": "Mon-Sun 7:00-20:00",
     3

     "promotion": { "type": "coupon", "value": "10% OFF" }
  }
Notes
• Location coordinates align with the world coordinate system. A small content-authoring tool will help place building models and determine 'entrance' colliders.
5. Technical architecture Stack
• Frontend: Three.js, React (for UI overlay), Zustand or Redux (state), Ammo.js or Cannon.js for physics (optional), pathfinding (yuka or custom navmesh logic).
• Backend (optional/multiplayer): Node.js + WebSocket (socket.io) or WebRTC SFU for scale. • Storage: JSON hosted on static storage (S3/Cloud) or a small CMS for business owners.
Frontend responsibilities
• Scene loader: loads base town glTF + building instances.
• Business loader: fetch JSON and instantiate building colliders and interactive UI triggers. • Event scheduler: local timer plus server sync for outbreaks.
• AI manager: spawns & updates zombies.
Data flow
1. On load, client fetches businesses.json and town_manifest.json .
2. Client loads minimal geometry and populates building markers.
3. When nearing a building, client fetches or loads the interior GLB.
4. Every outbreak is synchronized by server tick (if multiplayer) or calculated locally based on
server-provided epoch anchor to avoid skew.
6. Zombie event system Timing
• Fixed cadence: spawn event every 18 minutes (1080 seconds).
• Each event lasts configurable default 150 seconds (2.5 min).
• A countdown HUD shows time to next event (in-game, displayed in minutes:seconds).
Synchronization
• If multiplayer: server emits zombieEventStart with timestamp; clients sync to server time. This prevents clients from spawning on different schedules.
• If single-player: local timer runs from game start; consider storing last-event timestamp to persist across sessions.
      4

Spawn rules
• Spawns at randomized points across the town spawnZones[] .
• Spawn count = baseCount + floor(activePlayers / playersPerExtra) * scaleFactor. • Example: baseCount = 6, playersPerExtra = 3, scaleFactor = 2.
Zombie AI
• Lightweight state machine: Idle (wander) → Investigate → Chase → Attack.
• Navigation: simple navmesh pathfinding (A* on navmesh) or steering behaviours using
waypoints.
• Performance: limit active pathfinding requests per frame; use umbrella steering when many
zombies clustered.
Player effects & combat
• Zombies deal damage on contact; allow stun tool (e.g., flare, noise maker) to temporarily freeze zombies.
• Health regen disabled during event; medkits or safe shelters available.
Rewards
• Survive wave: survivors earn coins and XP. Bonus for saving NPCs or hitting objectives (e.g., holding plaza).
7. UI / UX wireframes Main HUD
• Top-left: mini-map (optional) showing player position and business markers. • Top-right: countdown to next zombie event.
• Bottom-center: interaction hint ("Press E to view business info").
• Bottom-left: health & inventory.
Business info panel
• Title, logo, description, CTA button (Visit website), promotion/coupon.
8. Multiplayer & networking (optional) Modes
• Shared session (small world): up to 32 players per shard.
• Instanced servers for events (zombie outbreaks) with stateful server logic.
Networking design
• Authoritative server for events & zombie AI positions (to prevent cheating). • Client-side prediction for player movement; server reconciliation.
   5

• Use spatial partitioning (zones) to limit network updates to nearby players.
 9. Tools for content authors / admin Business CMS (web admin)
• Upload GLB for exterior/interior, logo, business metadata, promotions.
• Place building in town via a simple 2D map editor (drag & drop) or by entering coordinates. • Preview how info board will look.
Admin controls
• Trigger or cancel zombie events manually.
• Hot-swap business JSON entries without restart.
10. Art & asset pipeline
• Use glTF 2.0 (GLB) for all 3D assets.
• Textures: power-of-two textures, atlasing for small props.
• Separate exterior & interior models for streaming.
• Naming conventions: building_<id>_ext.glb , building_<id>_int.glb .
11. Monetization & analytics
• Sponsored businesses: paid placements with clickthrough tracking.
• Cosmetic store: avatar skins, emotes.
• Analytics (mixpanel / amplitude): track visits per business, average time in interior, conversion
(website clicks), outbreak survival rates.
12. Roadmap & milestones (suggested)
M0 — Prototype (4 weeks) - Basic Three.js scene, player controller, simple building with info board,
JSON loader. - Local zombie event (spawn/wander) and HUD countdown.
M1 — MVP (8–12 weeks) - Multiple enterable buildings, interior streaming, CMS for JSON. - Basic AI with pathfinding, rewards, and persistence of business data. - Desktop polish and performance optimisations.
M2 — Multiplayer + Monetization (12–20 weeks) - Node.js server, WebSocket synchronization, sponsor portal, analytics integration.
      6

13. Appendix: code snippets & algorithms 1) JSON loader (simple, client-side)
  // fetch businesses.json and create markers
  async function loadBusinesses(url) {
    const resp = await fetch(url);
    const data = await resp.json();
    data.businesses.forEach(b => {
      // create a trigger at b.location
      createBusinessTrigger(b);
    });
}
2) Event scheduler (client)
  // Anchor time from server to avoid drift
  let serverEpoch = Date.now(); // replace with server-provided epoch
  const EVENT_INTERVAL = 18 * 60 * 1000; // 18 minutes in ms
  const EVENT_DURATION = 150 * 1000; // 150s
  function timeToNextEvent() {
    const now = Date.now();
    const elapsed = (now - serverEpoch) % EVENT_INTERVAL;
    return EVENT_INTERVAL - elapsed;
}
  function startLocalTimers() {
    setInterval(() => {
      if (timeToNextEvent() < 1000) {
        startZombieEvent();
}
}, 1000);
}
3) Simple zombie state machine (pseudocode)
  State: Idle -> Wander
  If player in detectionRadius: -> Chase
  If close enough: -> Attack
  If stunned: -> Stunned (timer)
  If player lost for > T: -> Wander
4) Navmesh note
• Generate navmesh with external tool (Recast/Detour) from the town geometry. Load navmesh into client and use A* to find paths for zombies.
   7

 Final notes & next steps
I created a full, editable design document with: map sketch, JSON schema and examples, event timing, spawn/AI rules, architecture and roadmap.
Next recommended immediate steps 1. Build a small prototype: single building, JSON loader, player movement, one zombie type. 2. Implement event timer & HUD so the 18-minute cadence is visible. 3. Create authoring tool for placing businesses and generating JSON.
If you'd like, I can now: - Produce the React + Three.js starter repo scaffold (single-file example + instructions). - Generate a sample businesses.json with 20 entries placed around the ASCII map. - Create a simple CMS wireframe for business owners.
Tell me which of the above you'd like next and I’ll add it to the document or produce code samples.
  8

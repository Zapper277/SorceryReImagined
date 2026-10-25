# HEXBOUND: Codebase Guide

An original dark-urban-fantasy action RPG (sorcerers vs. curses, powered by Hex), written in server-authoritative Luau.
All names, Lineages, techniques, curses and maps are original.

## 1. Where each file goes in Studio

| File | Location | Type |
|---|---|---|
| `CombatFrameData.lua` | ReplicatedStorage/Shared/CombatFrameData | ModuleScript |
| `TechniqueData.lua` | ReplicatedStorage/Shared/TechniqueData | ModuleScript |
| `LineageData.lua` | ReplicatedStorage/Shared/LineageData | ModuleScript |
| `ItemData.lua` | ReplicatedStorage/Shared/ItemData | ModuleScript |
| `MissionData.lua` | ReplicatedStorage/Shared/MissionData | ModuleScript |
| `PlayerDataService.lua` | ServerScriptService/Services/PlayerDataService | ModuleScript |
| `CombatService.lua` | ServerScriptService/Services/CombatService | ModuleScript |
| `HexService.lua` | ServerScriptService/Services/HexService | ModuleScript |
| `TechniqueRegistry.lua` | ServerScriptService/Services/TechniqueRegistry | ModuleScript |
| `ProgressionService.lua` | ServerScriptService/Services/ProgressionService | ModuleScript |
| `InventoryService.lua` | ServerScriptService/Services/InventoryService | ModuleScript |
| `RollService.lua` | ServerScriptService/Services/RollService | ModuleScript |
| `TradeService.lua` | ServerScriptService/Services/TradeService | ModuleScript |
| `CurseAI.lua` | ServerScriptService/Services/CurseAI | ModuleScript |
| `MapBuilder.lua` | ServerScriptService/Services/MapBuilder | ModuleScript |
| `MissionService.lua` | ServerScriptService/Services/MissionService | ModuleScript |
| `DomainService.lua` | ServerScriptService/Services/DomainService | ModuleScript |
| `ServerBootstrap.server.lua` | ServerScriptService/ServerBootstrap | Script |
| `CombatController.client.lua` | StarterPlayer/StarterPlayerScripts/CombatController | LocalScript |
| `HUDController.client.lua` | StarterPlayer/StarterPlayerScripts/HUDController | LocalScript |

If you use Rojo, these suffixes (`.server.lua`, `.client.lua`) already map correctly.

**Studio settings:** Game Settings → Security → *Enable Studio Access to API Services* (for saving data). Without it,
PlayerDataService falls back to in-memory profiles and prints a warning. Start from an **empty Baseplate**; the hub is
built at the world origin.

## 2. Controls

| Input | Action |
|---|---|
| LMB / R2 | M1 chain (4 hits) |
| F (hold) / L2 | Block. The first 8 frames are a parry. |
| Z X C V / X Y B D-pad-up | Techniques 1–4 |
| G | Domain (needs full Hex and the Lineage's level requirement) |
| B | Inventory (I is reserved for Roblox camera zoom) |
| K | Stats |
| T | Trade |
| M | Abandon the current mission |
| Proximity prompts in hub | Mission Board, Lineage Shrine |

## 3. Systems at a glance

- **Combat**: frame-data state machine with an M1 chain, block, parry, guard break, hitstun and knockback. NPCs use the same code path. Factions prevent friendly fire; PvP works only in the hub sparring ring.
- **Hex**: earned by fighting, spent on techniques. Spending down to 0 triggers a 3 s burnout. Regenerates only out of combat.
- **Techniques**: data-driven, in 4 kinds (Melee / Projectile / Dash / Burst). Hex and cooldown are committed at startup.
- **Lineages**: 6 total across 5 rarities, each with 4 techniques, stat bonuses and a Domain. Rolled with weighted odds and hard pity.
- **Domains**: barrier arena that traps everyone inside. Caster gets damage and Hex regen; enemies inside take more damage plus unblockable pulses. Overlapping Domains clash. Burnout afterwards.
- **Progression**: XP curve `100·L^1.5`, level cap 100, 3 stat points per level, Grades 4→Special.
- **Inventory**: consumables, accessories (2 slots), materials. Items have uids, so non-stackable items stay unique.
- **Trading**: lock offers → ready → 3 s countdown → re-check everything → swap in one step → save both profiles.
- **Missions**: 15 s party lobby (up to 4), separate instanced map, 5 mission types, kill XP, completion rewards and drops.
- **Saving**: DataStore with session locking, autosave, save on leave and on shutdown.

## 4. Maps: what was built, and what wasn't

`MapBuilder` procedurally builds **playable greybox blockouts** for the hub and every mission type. They have the right
scale, routes, cover, sightlines, lighting cues and objective placements, so you can play and balance every mission
today. They are **not** finished art. The intended pipeline is: play-test the blockout → lock the layout → an
environment artist dresses it (meshes, decals, terrain) using the blockout as the reference.

These are original layouts. I don't have the original game's map files, and copying them wouldn't be appropriate anyway.

| Theme | Mission type | Layout idea | Gameplay intent |
|---|---|---|---|
| **Sanctum (Hub)** | Social | Walled courtyard, mission board (N), Lineage shrine (E), sparring ring (W) | Everything reachable within a few seconds; PvP limited to the ring |
| **Derelict Subway** | Exorcism | 240-stud platform hall, two track trenches, a stalled 4-car train, end service tunnels | Mid-range fights on the platform, close-range fights in the tunnels; the train is cover |
| **Abandoned School** | Purge Nest | Hallway with 6 classrooms, gym (E), rooftop library reached by outdoor stairs (W) | Nests in a classroom, the gym and the library, so the party has to move between floors |
| **Night Market** | Escort | Winding street lined with stalls under lanterns; 3 open plazas; green extraction pad | Stalls block sightlines so ambushes come from the sides; the civilian only moves while a sorcerer is close |
| **Flooded Shrine** | Warding | Sunken courtyard, Ward Stone on a central dais, 3 causeways from red gates | You can see which gate each wave comes from; stone lanterns give cover on the dais |
| **Rooftop Cathedral** | Calamity (boss) | 110-stud central roof arena, bell tower landmark, 6 outer rooftops joined by beams | Large boss arena; outer roofs are for repositioning and add spawns; falling costs time |

Each map has an invisible kill floor that returns fallen players to spawn. The rebuilt layout varies with
`MapBuilder:Build(theme, origin, seed)`. To add a theme, add a function to `Themes` that returns
`playerSpawns`, `curseSpawns` and its objective points.

## 5. Assumptions I made (the spec didn't cover these)

1. All frame data, damage, rarity odds, pity thresholds, XP curve and rewards are placeholder balance values.
2. Technique Hex and cooldown are spent at cast start, with no refund if you're interrupted.
3. Dying in a mission removes you from it with no rewards. Completion rewards are full, not split.
4. A roll always replaces your Lineage, and never gives the one you already have.
5. Domains need full Hex and a per-Lineage minimum level (20–40); a Domain clash compares level + rarity + random(0–15).
6. Trades need level 5; soulbound items (`tradeable = false`) can't be traded.
7. Each new character starts with 50% Hex.

## 6. Known limitations and next steps

- **Animations and VFX**: animation IDs are placeholders (`rbxassetid://0`). Effects are Highlight flashes and colour correction.
  Upload animations and fill in `ANIMS` in `CombatController`. Projectiles need a client-side visual for the
  `CombatFX "Technique"` event (it already sends origin, direction and speed).
- **StreamingEnabled**: if it's on, call `player:RequestStreamAroundAsync(spawnPos)` before the mission teleport.
- **Lag compensation**: hitboxes use current server positions. Rewinding positions to when the attacker swung is the next combat upgrade.
- **Parties**: there are lobbies, but no persistent party or friend-invite system.
- **Cross-server trading** isn't supported (same server only). Session locks prevent duplication across servers.
- **UI** is a functional greybox meant to be reskinned.
- **Testing**: everything compiles and passes `luau-analyze` (the only warnings are Roblox globals). It has **not** been
  play-tested in Studio. Expect tuning passes, especially on map collision and AI pathing in the school map.

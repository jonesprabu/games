# Block Craft — design notes

A single-file 3D voxel sandbox (`craft.html`) built with [Three.js](https://threejs.org/)
(loaded from a CDN via import map, so it runs as a static page on GitHub Pages).

## Controls

| Action | Key |
| --- | --- |
| Move | `W` `A` `S` `D` |
| Look | Mouse (click to capture) |
| Jump / swim up / fly up | `Space` |
| Sneak / fly down | `Shift` |
| Sprint | `Ctrl` (hold) |
| Break block | Left click (hold in survival until it breaks) |
| Place block / eat | Right click |
| Select hotbar item | `1`–`9` or mouse wheel |
| Toggle fly (creative) | `F` or double-tap `Space` |
| Inventory / crafting | `E` |
| Pause / save / menu | `Esc` |

## Phase 1 (done) — single player

- **Two modes:** Creative (fly, unlimited blocks) and Survival (health, hunger,
  breath/drowning, fall damage, day–night cycle, passive + hostile mobs).
- **Rich world:** ~19 block types, procedural terrain (Perlin fBm), biomes
  (grass / sand beaches / snowy peaks), trees, caves, and ores
  (coal, iron, gold, diamond) that get rarer with depth.
- **Building:** raycast block break/place with per-block hardness, block drops,
  and a small crafting menu (logs→planks, planks→sticks, sand→glass, etc.).
- **Persistence:** worlds save to `localStorage` (seed + player state + only the
  blocks you changed) and reload via **Continue**.

## Architecture (kept modular for Phase 2)

The code is split into clear systems so a network layer can wrap the authoritative
world without rewriting gameplay:

- `World` — chunk storage, procedural generation, and the `edits` map (the only
  state that needs to be synced/persisted).
- Mesher — turns chunk voxels into face-culled Three.js geometry.
- `Player` — physics, AABB collision, camera.
- Inventory / Mobs / Game loop — input, day-night, save/load, UI.

`window.BlockCraft` exposes `{ game, World, startWorld, saveGame }` for debugging.

## Phase 2 (planned) — multiplayer

The intended approach for playing with friends:

1. **Authoritative server** (Node + WebSocket) that owns the `World` (seed +
   edits) and the list of players/mobs. The same `World`/generation code can be
   shared between client and server.
2. **Client → server:** join, player position/look, block break/place.
3. **Server → clients:** world seed + edits on join, then broadcast block
   changes, other players' positions, and mob updates.
4. Render remote players as simple avatars (reuse the mob model), add a name tag
   and a basic chat box.

Phase 1 already isolates the syncable state (`World.edits`, player transform,
inventory), so Phase 2 is mostly adding the transport + reconciliation layer.

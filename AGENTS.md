# Neon Rave — repo notes

Build-free static site: everything is in `index.html` (one inline `<style>` and one inline
`<script>`) plus `331music-disco-dancing-energy-603091.mp3`. There is no package.json, bundler,
backend or database, and the game is entirely client-side — **no external credentials are needed**.

## Run and verify

```
docker compose -f docker-compose.base44.yml up -d
curl -s localhost:3000/ | grep -o '<title>Neon Rave</title>'
```

`nginx:alpine` serves the repo bind-mounted read-only at `/usr/share/nginx/html` on host port 3000
(see `.base44/nginx.conf`). Nothing is compiled, so an edit to `index.html` is served immediately —
but there is no watcher or reload tool, so call `reload_preview` (or refresh the browser) to see it.

## Game flow

Each player answers 20 questions in sets of five. While a player is answering, their car on the
circuit creeps forward by itself (`auto[]` feeding `placeCar()`) — that is the auto-drive animation.
A correct answer adds 1 to the HUD `score` **and 1 life**; a wrong answer skips that question
(`qDone` still advances) and earns no life. Everyone starts with 3 lives.

After every 5th question that player's question box is hidden and their drive panel (`#d1` / `#d2`)
takes over: a 3D road (`.road3d` / `.roadGlow` trapezoids via `clip-path`, `#fin1`/`#fin2` at the
finish depth) runs to the finish line on its own while the player steers (panel ◀ LEFT / RIGHT ▶
buttons; ArrowLeft/ArrowRight also work for player 1). `placePickup()` maps a pickup's depth `z`
(1 = horizon, 0 = the car) to top/left/scale, so coins and blocks shrink and drift toward the
vanishing point. Coins add 1 to `score`; a block costs 1 life, and a block hit with 0 lives ends
that drive immediately. Question progress lives in `qDone` (separate from the HUD `score`), the
`nextDrive` checkpoint advances by 5, and the next five questions load when the drive ends. 20
questions wins. Both players run this independently.

## Quirks worth knowing

- **`index.html` must stay a single document.** Git history (commits `b7876cd`, `3c5a005`) contains
  copies where a second, byte-identical document was concatenated directly after the first
  `</script>` (`</script>e html><html lang="en">…`, truncated at EOF). That duplicated every `id`,
  put a second fixed start overlay on top of the game, and left the visible START RACE button with
  no listener — the game looked stuck. Today's file is truncated at the first document's
  `</script>` and closed with `</body></html>`.
- The `<audio>` element points at an **external** URL
  (`https://home.rohansweb.co.uk/Neonrace/331music-disco-dancing-energy-603091.mp3`); the mp3 in the
  repo root is a byte-for-byte fallback and is served locally at
  `/331music-disco-dancing-energy-603091.mp3`. Playback also needs a user gesture, so music only
  starts after START RACE.
- Game state is in-memory only: a page refresh resets scores. `document.getElementById` is used
  throughout, so duplicate ids silently break the game (see the first point).

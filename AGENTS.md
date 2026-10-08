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

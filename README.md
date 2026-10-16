# triptrail-render

Renders [TripTrail](https://github.com/Fangyuan025/triptrail) route animations
headlessly on GitHub Actions, so no phone or small server has to.

## Why this works

TripTrail's `renderOffline()` steps frames deterministically (`paintFrame(t)` +
`mapFrameSettled()`) rather than driving off `requestAnimationFrame`, so it does
not need a live, visible tab, the usual blocker for headless video capture. It
only takes that path if WebCodecs can encode H.264, which holds on
`ubuntu-latest` with no GPU (SwiftShader). Expect 11 to 20 minutes per render:
every frame is rasterised in software, so cost scales with canvas area.

`patch-triptrail.mjs` adjusts the clone before serving it (frame settle time,
camera smoothing, pacing). Each patch asserts its anchor text, so an upstream
change fails the build instead of rendering unpatched.

## Running one

Actions → **Render route** → Run workflow, with a route like:

```json
{"title":"Example","aspect":"16:9","style":"dark","stops":[
  {"name":"Lisbon","lng":-9.1393,"lat":38.7223,"mode":"plane"},
  {"name":"Madrid","lng":-3.7038,"lat":40.4168,"mode":"car"}]}
```

Or from a server, `POST /repos/<owner>/triptrail-render/dispatches` with
`{"event_type":"render-route","client_payload":{"route":{…},"ref":"job-42"}}`
using a token with Contents read/write on this repo.

`mode` is how you *arrived* at that stop, so the first one's is ignored. Globe
view turns on when any leg is a flight. Optional fields: `subtitle`, `accent`
(hex), `style` (`dark`, `eclipse`, `voyager`, `positron`, `fiord`, `liberty`,
`satellite`), `pace`, `globe`.

## Collecting the result

Each run publishes a release `render-<run>` holding `route.mp4`, named by the
dispatch's `ref` (a lowercase slug) or by its tag. Pull it from the Releases
API and delete the release once collected.

**This repo is public, and so are its releases and run logs.** The scripts keep
place names and coordinates out of the logs and never put the route's title on
a release, but the video itself shows the route until its release is deleted.

## Third-party code

TripTrail is MIT, © 2026 Fangyuan Lin. It is cloned at render time and served
locally; the author's own deployment is never automated against.

# Architecture

## Design principle

Prefer dual access:

- keep the official COOLFLY app/cloud functioning
- add independent local access for Home Assistant and archival

Avoid hardware replacement unless the stock path proves impractical. If hardware modification is eventually required, preserve compatibility with the official service where feasible.

## Proposed layers

### 1. Protocol client

A standalone Python library should own:

- authentication/session setup
- feeder discovery/wake behavior
- P2P/STUN negotiation
- command framing
- event / recording enumeration
- media retrieval
- metadata parsing

Keeping this separate from Home Assistant will make packet/protocol development easier and allow reuse outside HA.

### 2. Media pipeline

Desired flow:

```
COOLFLY feeder
   |
local/cloud-assisted protocol
   |
Python client
   |
clip reconstruction / remux / transcode if needed
   |
Home Assistant media storage
```

Target local layout:

```
/media/birdfeeder/YYYY-MM-DD/<timestamp>.<ext>
```

Prefer remuxing rather than transcoding when the camera already produces a common codec.

### 3. Home Assistant integration

Potential entities/features:

- `camera.coolfly_live`
- latest-event camera/image
- last visit timestamp
- last clip path/URL
- battery sensor
- storage status
- connectivity / signal sensor
- event count
- species label/confidence if exposed by COOLFLY
- services/actions to refresh events, fetch a clip, or request live view

### 4. Cloud coexistence

Cloud may remain useful for:

- login/authentication
- device discovery
- push notifications
- AI/species classification
- remote access
- vendor firmware updates

The integration should use the narrowest dependency necessary rather than trying to replace the entire COOLFLY ecosystem.

## Unknowns

- exact P2P implementation/vendor
- local authentication requirements
- media codec and framing
- recording enumeration commands
- whether SD playback is direct local UDP
- whether sessions can be established without COOLFLY cloud
- whether simultaneous vendor-app and HA access is supported

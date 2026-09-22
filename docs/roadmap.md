# Roadmap

## Phase 0 — Evidence capture

- [x] Identify feeder LAN address and MAC.
- [x] Verify aggressive sleep/wake behavior.
- [x] Observe feeder ARP for COOLFLY iPhone.
- [x] Capture iPhone traffic through Apple RVI.
- [x] Confirm direct local UDP path during Live View.
- [x] Identify STUN-formatted negotiation.
- [x] Identify sustained feeder-to-phone media-like UDP stream.
- [ ] Capture clean canonical Live View PCAP.
- [ ] Capture microSD playback PCAP.
- [ ] Compare Live View vs SD playback handshakes and payloads.

## Phase 1 — Protocol identification

- [ ] Extract STUN transactions and attributes.
- [ ] Identify stable device/session identifiers.
- [ ] Fingerprint the P2P SDK/protocol if possible.
- [ ] Analyze UDP media packet headers.
- [ ] Check payloads for H.264/H.265 NAL units or encryption.
- [ ] Reverse-engineer minimum command/session state machine.

## Phase 2 — Recording access

- [ ] Discover SD-card recording-list command.
- [ ] Enumerate recordings.
- [ ] Request a specific recording.
- [ ] Reassemble a retrieved stream.
- [ ] Produce a valid playable local video file.

## Phase 3 — Standalone client

- [ ] Create Python package/client.
- [ ] Device discovery/session management.
- [ ] Recording enumeration API.
- [ ] Clip download API.
- [ ] Live-stream API if practical.
- [ ] Tests using sanitized fixtures/packet excerpts.

## Phase 4 — Home Assistant

- [ ] Custom integration scaffold.
- [ ] Config flow.
- [ ] Coordinator/client lifecycle.
- [ ] Sensors and diagnostics.
- [ ] Media download/storage.
- [ ] Latest-event camera/media entity.
- [ ] Optional live camera entity.
- [ ] HACS-ready packaging/docs.

## Phase 5 — Hardening

- [ ] Redact sensitive diagnostics.
- [ ] Handle device sleep/wake reliably.
- [ ] Handle network changes / dynamic IP.
- [ ] Verify coexistence with COOLFLY app.
- [ ] Document supported hardware/firmware.
- [ ] CI, linting, tests, releases.

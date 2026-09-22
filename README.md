# ha-coolfly

Reverse-engineering and Home Assistant integration work for HIXX / COOLFLY smart bird feeders and related COOLFLY devices.

## Current status

Research / protocol-discovery stage. No Home Assistant integration is functional yet.

Confirmed so far:

- The feeder is reachable on the local LAN.
- COOLFLY on iPhone establishes a direct UDP path to the feeder for Live View.
- The initial exchange is STUN-formatted and appears to negotiate local P2P connectivity.
- After negotiation, one feeder UDP port becomes the sustained high-volume path to the phone.
- Live View traffic consists of many repeated ~522/524-byte UDP payloads plus smaller control/fragment packets.
- The vendor cloud may still be involved in authentication, discovery, metadata, or NAT traversal; the local media path does not imply the cloud is unnecessary.

## Goal

Preserve the vendor app/cloud experience while adding independent local access where possible.

Primary objectives:

1. Discover enough of the COOLFLY protocol to access the feeder locally.
2. Enumerate and retrieve microSD recordings.
3. Save bird clips into Home Assistant/local storage.
4. Expose useful entities such as:
   - latest bird event / clip
   - last visit timestamp
   - camera/live view, if practical
   - battery / storage / connectivity metadata where available
   - species/event metadata if obtainable
5. Keep the official COOLFLY app usable in parallel.

## Repository layout

- `docs/research-log.md` — chronological discoveries and packet-capture notes.
- `docs/architecture.md` — intended integration architecture and design principles.
- `docs/roadmap.md` — reverse-engineering and HA implementation milestones.

## Safety / privacy

Packet captures may contain device identifiers, session credentials, network addresses, or other sensitive data. Raw captures should not be committed without sanitization.

## Project stage

Expect protocol notes, exploratory tooling, and incomplete code until the wire protocol is understood.

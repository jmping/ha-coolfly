# Research log

## Device / environment

Observed device:

- Product family: HIXX smart bird feeder
- App: COOLFLY
- Android package reported during research: `com.hixx.coolfly`
- Feeder LAN IP during testing: `192.168.68.136`
- Feeder MAC: `b4:61:e9:ee:68:23`
- MAC OUI maps to Sichuan AI-Link Technology Co., Ltd.
- iPhone running COOLFLY during capture: `192.168.68.123`

The AI-Link OUI suggests generic/OEM Wi-Fi camera hardware, but does not by itself identify the protocol stack or platform.

## Network reconnaissance

### TCP scan

A scan of the common TCP ports while the device was idle returned all scanned ports filtered/no-response.

The feeder appears to use aggressive power saving. Ping latency while idle varied from roughly tens of milliseconds to more than one second.

### Live View wake behavior

Opening Live View in COOLFLY wakes the device. Once established, ICMP latency often falls into the ~7–25 ms range.

Some ping tests showed duplicate replies with TTL values of both 255 and 64. The cause is not yet established. Possibilities include multiple network stacks/processors, proxy/bridge behavior, or another network peculiarity. This should not be treated as proof of any one architecture.

### ARP observations

During app use, the feeder actively ARPs for the iPhone:

```
who-has 192.168.68.123 tell 192.168.68.136
```

This was the first strong indication that COOLFLY may use a direct LAN path between phone and feeder.

A normal Mac-side packet capture could only see broadcasts/ARP because the Mac was not in the switched unicast path.

## iPhone Remote Virtual Interface capture

Using Apple's RVI with the connected iPhone made the direct phone/feeder traffic visible.

Example RVI setup:

```bash
xcrun xctrace list devices
rvictl -s <IPHONE_UDID>
sudo tcpdump -i rvi0 -nn -vv host 192.168.68.136
```

The RVI interface reports that promiscuous mode is unsupported; that warning is expected and does not prevent capture.

## Live View protocol observations

A Live View session showed the phone initiating UDP toward the feeder and the feeder replying over multiple candidate UDP ports.

One captured session used:

- iPhone: `192.168.68.123:52729`
- feeder candidate/media ports: `37375`, `37927`, `57804`

Earlier sessions used different ephemeral ports, e.g. phone `65112` and feeder `50622`, `38194`, `34246`. Ports therefore appear session-dependent.

### STUN

The beginning of the exchange contains the STUN magic cookie:

```
21 12 a4 42
```

Packets include STUN-style Binding requests/responses and attributes consistent with connectivity checks / NAT traversal. This supports a P2P negotiation model.

Readable fragments observed in STUN-style packets included strings such as:

```
qeT2:6ppu
6ppu:qeT2
```

Their exact semantics are not yet known. They may be ICE-style usernames, session identifiers, or ephemeral credentials.

Do not assume a specific P2P vendor (TUTK, etc.) solely from STUN.

### Sustained media path

After negotiation, one feeder UDP port dominates the traffic.

In the captured Live View session:

```
192.168.68.136:37375 -> 192.168.68.123:52729
```

This path carried a dense stream of UDP packets, especially:

- 524-byte UDP payloads
- 522-byte UDP payloads
- frequent 182-byte payloads
- assorted smaller/larger packets

The pattern strongly suggests an application-layer media transport that fragments video/audio/control data into fixed or semi-fixed chunks.

The other candidate feeder ports continued emitting occasional ~88-byte packets, likely connectivity checks or alternate candidate maintenance.

This establishes that Live View uses a direct local UDP data path once connectivity is negotiated.

It does **not** yet establish:

- whether video payloads are encrypted
- whether the stream is H.264, H.265, or another codec
- whether the P2P protocol is proprietary or based on a known OEM SDK
- whether COOLFLY cloud authentication is required before local negotiation
- whether SD-card playback uses the same local transport

## Packet-capture caveat

One Live View PCAP-NG file later produced:

```
pcap_loop: block in pcapng dump file has a length of 262146 that is not a multiple of 4
```

The beginning and substantial portions remained readable, but future captures should be cleanly stopped with Ctrl-C and packet totals allowed to print before disconnecting the phone.

Raw PCAPs should not be committed publicly without review/sanitization.

## Next experiment: microSD playback

Highest-value next test:

1. Start a fresh RVI capture with Live View closed.
2. Wait ~5 seconds.
3. Open an existing microSD recording in COOLFLY.
4. Play it for ~15 seconds.
5. Stop playback.
6. Wait ~5 seconds.
7. Stop tcpdump cleanly.

Suggested capture:

```bash
sudo tcpdump -i rvi0 -nn -s0 \
  -w ~/Desktop/hixx-sd.pcap \
  host 192.168.68.136
```

Then inspect feeder-to-phone UDP traffic:

```bash
tcpdump -nn -r ~/Desktop/hixx-sd.pcap \
  'udp and src host 192.168.68.136' |
tail -80
```

If SD playback produces a similar negotiated direct UDP stream, the likely implementation target becomes:

```
wake / authenticate device
        |
negotiate P2P session
        |
enumerate SD recordings
        |
request recording
        |
receive / decode media transport
        |
save clip locally
        |
expose through Home Assistant
```

## Longer-term reverse-engineering paths

If LAN packet analysis stalls:

- Inspect COOLFLY Android APK.
- Search decompiled code/native libraries for:
  - STUN / ICE
  - P2P SDK names
  - device IDs / session tokens
  - recording list commands
  - playback commands
  - H.264 / H.265 / codec handling
  - cloud API base URLs
  - MQTT / WebRTC / RTSP references
- Inspect the feeder microSD card read-only for:
  - video files
  - indexes
  - SQLite databases
  - metadata
  - config/log files
  - encryption markers

## Key design preference

The desired end state is additive, not destructive: retain official COOLFLY functionality and any useful cloud/AI features while adding independent local access and archiving.

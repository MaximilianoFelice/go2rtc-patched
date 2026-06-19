# go2rtc-patched

Patched builds of [go2rtc](https://github.com/AlexxIT/go2rtc) for personal use in
[raspi-server](https://github.com/MaximilianoFelice/raspi-server) (a private k3s
home-cluster).

## Why

The upstream releases of `go2rtc` 1.9.10 (bundled inside Frigate 0.17.x) and
1.9.14 carry three latent bugs in the Nest WebRTC client that, under specific
conditions, make a Nest doorbell stream go into a permanent broken state
("Could not find codec parameters for stream X (Video: h264, none): unspecified
size") with no automatic recovery.

These bugs are described and fixed by upstream PR
[#2194](https://github.com/AlexxIT/go2rtc/pull/2194):

1. **`http.Client{Timeout: time.Second * 5000}` (= 5 000 seconds, ~83 min)** in
   six callsites of `pkg/nest/api.go`. A single hung HTTP call to Google keeps
   a goroutine + sockets alive for 83 minutes.
2. **`rtcConn` never calls `pc.Close()` on the three error paths**
   (`CreateCompleteOffer`, `ExchangeSDP`, `SetAnswer`) in `pkg/nest/client.go`,
   leaking a full `*webrtc.PeerConnection` on every failed reconnect.
3. **`ExtendStream` uses a one-shot `*time.Timer`** instead of a recurring loop,
   so the WebRTC media session is extended once and then expires, forcing a
   reconnect every ~5 minutes.

That PR is open since 2026-03-31 but not yet merged at the time of this build.
This repository simply provides binaries of go2rtc with the PR cherry-picked.

## Releases

Each release is named `v<go2rtc-tag>-patch<N>` and ships pre-built static
binaries.

The build script and exact patch applied are documented in the release notes.

## Disclaimer

Binaries here are for personal use only. They are unsigned, built from a
non-canonical source tree and are not affiliated with the upstream go2rtc
maintainers. Use at your own risk.

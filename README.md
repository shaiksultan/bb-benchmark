# bb-benchmark

Short name for the study **Fixed and Mobile Broadband Benchmarking and Operator Strategy for Fiber, 5G, and LEO Networks**.

This repository documents the central measurement server and the four access probes used to compare:

- fixed fiber
- 5G fixed wireless
- mobile 5G
- LEO satellite (Starlink)

## Files in this repo

| File | What it is |
| --- | --- |
| [Data/data.zip](Data/data.zip) | Raw probe logs, 4–6 September 2026: fiber, fixed wireless, mobile, and LEO |
| [topup-24h.zip](topup-24h.zip) | 24-hour follow-up scripts. The same files are also in [topup-24h/](topup-24h/) |
| [Data/network-kpi-timestamp.xlsx](Data/network-kpi-timestamp.xlsx) | Network KPIs with timestamps, after the 1.5×IQR screen |
| [Data/network-kpi-outliers-availability.xlsx](Data/network-kpi-outliers-availability.xlsx) | Outlier candidates and the availability log, including failed pings |
| [Data/application-qoe.xlsx](Data/application-qoe.xlsx) | Application timing for social, messaging, calling, and Teams |
| [Data/youtube-netflix-ott.xlsx](Data/youtube-netflix-ott.xlsx) | YouTube and Netflix playback runs |
| [Data/video-quality-uvq.xlsx](Data/video-quality-uvq.xlsx) | GoChat and BOTIM video scores from Google UVQ |

`sample-data/` inside the top-up folder shows the column layout. Those rows are placeholders, not measurements.

## KPIs the scripts record

Network, written by the probe scripts:

- iperf3 download and upload
- Round-trip time: average, P50, P95, P99, and round-trip time while iperf3 is running (top-up only)
- UDP jitter and UDP packet loss
- Ping loss and probe success. Every failed ping stays in the file.
- DNS response time
- TCP connect, TLS handshake, and time to first byte. TLS is the time after TCP, from curl.
- Voice-pattern delay, jitter, loss, and an estimated MOS from delay and loss. This is not a listening score.
- Gaming-pattern delay, jitter, and loss
- Ookla download, upload, ping, and server id
- SIPp call setup toward the lab server

Application and video, stored in the workbooks rather than in the probe script:

- Facebook, Instagram, X, TikTok, WhatsApp, BOTIM, GoChat, Teams join, and application YouTube timing
- YouTube playback from the player’s Stats for Nerds panel, kept separate from the application YouTube timing
- Netflix playback and the player-reported VMAF score
- GoChat and BOTIM Google UVQ scores

The top-up script does not delete outliers. Publish the unfiltered rows. A 1.5×IQR screen, if used, is a later check and is applied per network and per KPI.

## Install the top-up

Match the lab server on every probe: Ubuntu 24.04.3, iperf 3.16, curl 8.5.0, Python 3.12, SIPp 3.7.2, Ookla Speedtest CLI 1.2.0.84. Cloud VMs need iperf 3.16 only.

```bash
unzip topup-24h.zip
cd topup-24h
chmod +x *.sh
./INSTALL_CLIENT.sh          # each probe
./INSTALL_SERVER.sh lab      # lab server
./INSTALL_SERVER.sh cloud    # UAE VM and Europe VM
./write_versions.sh          # must print VERSIONS OK
```

Set `LAB_HOST`, `UAE_HOST`, and `EU_HOST` in `topup.conf`. Set `LINK` and `DEVICE` in each file under `paths/`. Open TCP and UDP 5201–5204 and ICMP on all three servers, and UDP 5061–5064 on the lab server.

```bash
./START_LAB_SERVER.sh
./START_CLOUD_SERVER.sh
./RUN_TOPUP.sh paths/fiber.env
./RUN_TOPUP.sh paths/fwa.env
./RUN_TOPUP.sh paths/mobile.env
./RUN_TOPUP.sh paths/leo.env
```

Timeouts are at the bottom of `topup.conf`. The full procedure, ports, and schedule are in [the top-up setup guide](docs/06-topup-setup.md).

The September workbooks and the later fiber run are also under `Data/`.

## What the platform does

Each access network runs the same client software against one central server. Throughput, delay, loss, and a synthetic voice setup are measured to that server, so the far end of the test is the same. Public speed tests and public web pages are recorded as a separate, customer-facing check. They are not the primary comparison.

## Read this first

| Document | What it answers |
| --- | --- |
| [Architecture](docs/01-architecture.md) | What runs where, and which ports each access network uses |
| [Setup](docs/04-setup.md) | How to start the central server, then install, check, and run each client |
| [Methodology](docs/02-methodology.md) | What each KPI is, how often it runs, and which timeouts the code sets |
| [September 2026 campaign](docs/03-campaign-2026-09.md) | What the 4–6 September network files show |
| [Application QoE, YouTube, Netflix, UVQ](docs/05-application-qoe.md) | Application timing, OTT playback, and Google UVQ |
| [Reply for the VP](docs/VP-REPLY.md) | Network and application answers for the white paper |
| [24-hour top-up setup](docs/06-topup-setup.md) | How to install, configure, and run the follow-up campaign |

## Campaign in one paragraph

From 4 to 6 September 2026 the probes wrote raw samples to the central server path. Fiber was the stable reference: about 713 Mbps down, 188 Mbps up, 11 ms round trip, and 99.8% probe success. LEO delivered about 240 Mbps down and 12 Mbps up, with about 48 ms round trip. The mobile 5G probe delivered about 328 Mbps down, on only nine download tests and an 18-hour window. 5G fixed wireless looked near 940 Mbps on public speed tests, while the path to the central server succeeded on only 57.9% of minute-by-minute probes. Those two fixed-wireless results must not be averaged.

Application timing, the separate YouTube and Netflix runs, and the Google UVQ scores for GoChat and BOTIM are in the application note. The network paragraph above is the probe campaign. The two campaigns are not one average.

# bb-benchmark

Short name for the study **Fixed and Mobile Broadband Benchmarking and Operator Strategy for Fiber, 5G, and LEO Networks**.

This repository documents the central measurement server and the four access probes used to compare:

- fixed fiber
- 5G fixed wireless
- mobile 5G
- LEO satellite (Starlink)

The workbooks used for the manuscript are in `Data/`. They cover the network KPI tables, the outlier and availability audit, application QoE, YouTube and Netflix, and the Google UVQ scores for GoChat and BOTIM. The raw probe logs are there too: `Data/data.zip` for 4–6 September, and `Data/new_data.zip` for the later fiber run.

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

## Campaign in one paragraph

From 4 to 6 September 2026 the probes wrote raw samples to the central server path. Fiber was the stable reference: about 713 Mbps down, 188 Mbps up, 11 ms round trip, and 99.8% probe success. LEO delivered about 240 Mbps down and 12 Mbps up, with about 48 ms round trip. The mobile 5G probe delivered about 328 Mbps down, on only nine download tests and an 18-hour window. 5G fixed wireless looked near 940 Mbps on public speed tests, while the path to the central server succeeded on only 57.9% of minute-by-minute probes. Those two fixed-wireless results must not be averaged.

Application timing, the separate YouTube and Netflix runs, and the Google UVQ scores for GoChat and BOTIM are in the application note. The network paragraph above is the probe campaign. The two campaigns are not one average.

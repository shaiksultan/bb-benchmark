# bb-benchmark

Short name for the study **Fixed and Mobile Broadband Benchmarking and Operator Strategy for Fiber, 5G, and LEO Networks**.

This repository documents the central measurement server and the four access probes used to compare:

- fixed fiber
- 5G fixed wireless
- mobile 5G
- LEO satellite (Starlink)

It is a documentation project. It does not contain probe logs, result spreadsheets, or the lab server address.

## What the platform does

Each access network runs the same client software against one central server. Throughput, delay, loss, and a synthetic voice setup are measured to that server, so the far end of the test is the same. Public speed tests and public web pages are recorded as a separate, customer-facing check. They are not the primary comparison.

## Read this first

| Document | What it answers |
| --- | --- |
| [Architecture](docs/01-architecture.md) | What runs where, and which ports each access network uses |
| [Setup](docs/04-setup.md) | How to start the central server, then install, check, and run each client |
| [Methodology](docs/02-methodology.md) | What each KPI is, how often it runs, and which timeouts the code sets |
| [September 2026 campaign](docs/03-campaign-2026-09.md) | What the 4–6 September result files actually show |
| [Reply for the VP](docs/VP-REPLY.md) | A note that can be sent as the answer to the methodology questions |

## Campaign in one paragraph

From 4 to 6 September 2026 the probes wrote raw samples to the central server path. Fiber was the stable reference: about 713 Mbps down, 188 Mbps up, 11 ms round trip, and 99.8% probe success. LEO delivered about 240 Mbps down and 12 Mbps up, with about 48 ms round trip. The mobile 5G probe delivered about 328 Mbps down, on only nine download tests and an 18-hour window. 5G fixed wireless looked near 940 Mbps on public speed tests, while the path to the central server succeeded on only 57.9% of minute-by-minute probes. Those two fixed-wireless results must not be averaged.

Figures above are medians from the raw September files, with no outlier removal. Detail, sample counts, and the limits on each figure are in the campaign note.

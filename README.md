# bb-benchmark

Short name for the study **Fixed and Mobile Broadband Benchmarking and Operator Strategy for Fiber, 5G, and LEO Networks**.

This repository holds the method notes **and** the campaign data for comparing:

- fixed fiber
- 5G fixed wireless (FWA)
- mobile 5G
- LEO satellite (Starlink)

## What the platform does

Each access network runs the same client software against one central server. Throughput, delay, loss, and a synthetic voice setup are measured to that server, so the far end of the test is the same. Public speed tests and public web pages are recorded as a separate, customer-facing check. They are not the primary comparison.

## Data folder

All campaign inputs live under [`data/`](data/):

| File / folder | Description |
| --- | --- |
| [`data/data.zip`](data/data.zip) | Packaged raw KPI CSVs (Fiber, 5G FWA, Mobile 5G, Starlink) |
| [`data/e_Fiber/`](data/e_Fiber/) | Extracted Fiber probe CSVs |
| [`data/e_5G_FWA/`](data/e_5G_FWA/) | Extracted 5G FWA probe CSVs |
| [`data/e_Mobile_5G/`](data/e_Mobile_5G/) | Extracted Mobile 5G probe CSVs |
| [`data/Starlink/`](data/Starlink/) | Extracted LEO / Starlink probe CSVs |
| [`data/Final_Benchmark_Timestamp_Data.xlsx`](data/Final_Benchmark_Timestamp_Data.xlsx) | Timestamped network KPI workbook |
| [`data/Application_QOE_All.xlsx`](data/Application_QOE_All.xlsx) | Application QoE (all samples; no outlier sheet required for sharing) |
| [`data/youtube_netflix_raw_Data_video_quality.xlsx`](data/youtube_netflix_raw_Data_video_quality.xlsx) | YouTube / Netflix OTT raw playback & video-quality fields |
| [`data/GochatBotim_Google_UVQ.xlsx`](data/GochatBotim_Google_UVQ.xlsx) | GoChat / BOTIM Google UVQ 1.5 perceptual video scores |

Typical CSV types inside each path folder: `samples.csv`, `reliability.csv`, `speedtest_samples.csv`, `browser_samples.csv`, `sip_sim_samples.csv`, `sip_stats.csv`.

## Read this first

| Document | What it answers |
| --- | --- |
| [Architecture](docs/01-architecture.md) | What runs where, and which ports each access network uses |
| [Setup](docs/04-setup.md) | How to start the central server, then install, check, and run each client |
| [Methodology](docs/02-methodology.md) | What each KPI is, how often it runs, and which timeouts the code sets |
| [September 2026 campaign](docs/03-campaign-2026-09.md) | What the 4–6 September network files show |
| [Application QoE, YouTube, Netflix, UVQ](docs/05-application-qoe.md) | Application timing, OTT playback, and Google UVQ |

## Campaign in one paragraph

From 4 to 6 September 2026 the probes wrote raw samples to the central server path. Fiber was the stable reference: about 713 Mbps down, 188 Mbps up, 11 ms round trip, and 99.8% probe success. LEO delivered about 240 Mbps down and 12 Mbps up, with about 48 ms round trip. The mobile 5G probe delivered about 328 Mbps down, on only nine download tests and an 18-hour window. 5G fixed wireless looked near 940 Mbps on public speed tests, while the path to the central server succeeded on only 57.9% of minute-by-minute probes. Those two fixed-wireless results must not be averaged.

Application timing, the separate YouTube and Netflix runs, and the Google UVQ scores for GoChat and BOTIM are in the application note and the workbooks under `data/`. The network paragraph above is the probe campaign. The two campaigns are not one average.

## Planned next step

A short **24–48 hour synchronized top-up** across all four paths (scripts ready), with row-level raw retention (`record_id` + UTC timestamp), consistent tool settings, and clear separation of automated network KPIs from manual OTT / application runs.

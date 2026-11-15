# Architecture

## Role of the central server

The central measurement server is the common far end for the tests that must be compared across access networks.

Clients do not test against each other. Each client opens its own test to the central server:

- TCP and UDP throughput, using iperf3
- ICMP round trip and the availability probe
- synthetic SIP call setup, using SIPp

Two measurements do **not** end on the central server, by design:

- Ookla Speedtest CLI picks a public Ookla server. That server can change by network and by hour.
- Web timing is taken against two public sites, the operator site and a news site. Those timings include the public internet beyond the access link.

```text
Fiber probe --------+
5G FWA probe -------+--> central server
Mobile 5G probe ----+     iperf3  5201-5204
LEO probe ----------+     SIPp    5061-5064
                         ICMP

Fiber / FWA / Mobile / LEO
        |
        +--> public Ookla servers          (secondary speed check)
        +--> operator site and news site   (web timing)
        +--> 1.1.1.1                        (DNS timing)
```

iperf3 serves one active test per process. Four listeners are required so the four probes can test at the same time instead of receiving "server is busy".

| Access | iperf3 TCP/UDP | SIP UDP |
| --- | --- | --- |
| LEO (Starlink) | 5201 | 5061 |
| Fiber | 5202 | 5062 |
| 5G fixed wireless | 5203 | 5063 |
| Mobile 5G | 5204 | 5064 |

The client package documents those listeners. The server-side start scripts named in the execution guide (`start_4_iperf_servers.sh`, `start_sip_sim_server.sh`) are not in the client folder that was reviewed. On 19 October 2026 the port listing on the central host stopped at a sudo password prompt, so this note does not claim which ports were listening that day.

## Central host

Captured 19 October 2026, 15:55 +04, clock synchronized to NTP, timezone Asia/Dubai.

| Item | Value |
| --- | --- |
| Role | Central measurement server |
| Hostname | netanalyze |
| OS | Ubuntu 24.04.3 LTS, kernel 6.8.0-110-generic |
| iperf3 | 3.16 |
| SIPp | 3.7.2 |
| Python | 3.12.3 |
| curl | 8.5.0 |
| TLS | OpenSSL 3.0.13 |
| Web server present | Apache 2.4.58. The benchmark does not use it. nginx is not installed. |

CPU, memory, and NIC inventory for this host were not captured. The hardware tables below are the probe machines, not the central server.

## Probe hosts

The four access networks were not measured from identical machines. Tool versions also differ. The test method is the same; the binaries are not.

| Access | Machine | OS used to run the probe | iperf3 | SIPp | Python | Ookla CLI |
| --- | --- | --- | --- | --- | --- | --- |
| Fiber | Dell PowerEdge R740, 2× Xeon Gold 6140 (72 threads), 62 GiB RAM, 1 GbE and 10 GbE NICs | Ubuntu 24.04.4 LTS | 3.16 | 3.7.2 | 3.12.3 | 1.2.0.84 |
| 5G fixed wireless | Dell PowerEdge R740xd, 2× Xeon Gold 5120 (56 threads), 30 GiB RAM, 1 GbE NICs only | Ubuntu 22.04.5 LTS | 3.9 | 3.6.1 | 3.10.12 | 1.2.0.84 |
| LEO | Dell PowerEdge R740, 2× Xeon Gold 6140 (72 threads), 62 GiB RAM, 1 GbE NICs only | Ubuntu 22.04 LTS | 3.9 | 3.6.1 | 3.10.12 | 1.2.0.84 |
| Mobile 5G | Lenovo laptop, 12th-gen Intel Core i5, 16 GB RAM, Windows 10 Enterprise 22H2 | WSL2 Ubuntu 26.04.1 | 3.20 | 3.7.7 | 3.14.4 | 1.2.0.84 |

`fping` was not installed on the probe hosts. Availability uses the system `ping`.

## Access equipment and the application devices

Recorded with the campaign notes:

- Fiber CPE: Arcadyan eLife Connect C1BA. The note says this path used 5 GHz Wi-Fi.
- 5G fixed wireless CPE: Pegatron MG36AX.
- Mobile 5G handset referenced for throughput: Samsung Galaxy S25.
- LEO router: Starlink. The note says this path used 5 GHz Wi-Fi.
- Application video tests (YouTube and Netflix) were run on a MacBook Air M2. That work is not part of the central server.

The rack probes have Ethernet adapters. The notes do not record, for every run, whether the probe host was wired to the CPE or crossed a Wi-Fi hop, or the signal level on that hop. That gap should stay open in any external write-up rather than be filled in.

## What each client runs

One network profile is chosen per laptop or probe: Starlink, fiber, 5G fixed wireless, or mobile 5G. The profile selects the port pair above and points iperf3, ping, and SIP at the central server.

During a long run the client starts two processes:

1. `reliability_monitor.py` — one ICMP probe on its own timer, for the whole run.
2. `benchmark_48h.py` — round trip, DNS, web timing, TCP throughput, UDP, a voice-like UDP pattern, and a gaming-like UDP pattern. These share one process, so they run one after another.
3. The network's `*_full.sh` wrapper adds, on their own timers, Ookla, a headless Chromium page load, and the SIPp caller.

Results stay on the client, under `results/<network>/`:

| File | Contents |
| --- | --- |
| `samples.csv` | KPI rows from the main benchmark |
| `reliability.csv` | one row per availability probe |
| `speedtest_samples.csv` | Ookla |
| `browser_samples.csv` | Chromium page timing |
| `sip_sim_samples.csv` | synthetic SIP attempts |

The September 5G fixed-wireless folder has no `browser_samples.csv`.

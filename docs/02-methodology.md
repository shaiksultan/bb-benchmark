# Methodology

The description below is what the client code does. Where the September 2026 files show a different interval than the comments in the config, the campaign note uses the files. The code is the method. The files are the evidence of what ran.

## Two throughput methods

**Primary: iperf3 to the central server.** Download is iperf3 reverse mode (`-R`), so the server sends and the client receives, for 60 seconds. Upload is the same test without `-R`, also 60 seconds, after a 10-second pause. The result is `bits_per_second` from the JSON summary, converted to Mbps. Download prefers `sum_received`. Upload prefers `sum_sent`.

**Secondary: Ookla Speedtest CLI.** The official `speedtest` binary, JSON output, bandwidth converted from bytes/s to Mbps. The selected public server name, location, and id are stored with each row. Ookla is a check on what a customer speed test would see. It is not the comparison across fiber, 5G, and LEO, because the far-end server is not under our control.

A quick check mode shortens the iperf3 tests to 10 seconds. The long run uses 60 seconds.

## KPI catalogue

| ID | What is recorded | How |
| --- | --- | --- |
| KPI-001 | Download throughput | iperf3 TCP, 60 s, reverse |
| KPI-002 | Upload throughput | iperf3 TCP, 60 s |
| KPI-003 | RTT average, P50, P95, P99 | 20 ICMP echoes, 0.2 s apart |
| KPI-004 | UDP jitter | iperf3 UDP, 60 s, offered at 10 Mbps, client toward server |
| KPI-005 | Loss | ICMP loss from the same 20 pings, and UDP loss from the 10 Mbps test. Both use this id. |
| KPI-006 | DNS response time | 5 queries for the operator hostname to 1.1.1.1, `dig +tries=1 +time=3`, mean of replies |
| KPI-007 | Time to first byte | curl, 5 repeats, operator site and news site |
| KPI-008 | TCP connect time | same curl timing, in milliseconds |
| KPI-009 | TLS handshake | curl app-connect minus TCP connect |
| KPI-027 | Browser load, operator site | headless Chromium, `wait_until=load`, then 1.5 s |
| KPI-028 | Navigation success | a second visit to the operator site until DOM content is loaded, scored 100 or 0 |
| KPI-029 | Browser load, news site | same method as KPI-027 |
| KPI-030 | Voice-pattern RTT P95 | 50 ICMP echoes, 0.1 s apart |
| KPI-031 | Voice-pattern jitter | iperf3 UDP, 150 s, 100 kbps, 200-byte payload, client toward server |
| KPI-032 | Voice-pattern loss | loss from that UDP test |
| KPI-033 | Estimated voice MOS | E-model-style formula below. Not a listening test. |
| KPI-034 | Gaming-pattern RTT P95 and P99 | 100 ICMP echoes, 0.1 s apart |
| KPI-035 | Gaming-pattern jitter | iperf3 UDP, 300 s, 256 kbps, client toward server |
| KPI-036 | Gaming-pattern loss | loss from that UDP test |
| SIP-SIM | Synthetic call completion | SIPp built-in caller scenario. Not a real IMS or handset call. |

UDP jitter and loss are measured in the client-to-server direction. The code does not set reverse mode on UDP tests. A download jitter figure was not collected.

KPI-028 is a page navigation, not a login, search, or purchase.

### Estimated voice MOS

Used only to compare paths. It is not ITU-T P.863 and it is not a MOS from listeners.

- one-way delay is approximated as ping P95 / 2
- delay impairment `Id = 0.024 × one-way + 0.11 × max(0, one-way − 177.3)`
- loss impairment `Ie = 0` when loss is 0, otherwise `30 × log10(1 + 15 × loss/100)`
- `R` is `93.2 − Id − Ie`, limited to 0…100
- `MOS = 1 + 0.035×R + R×(R−60)×(100−R)×7×10⁻⁶`, limited to 1…4.5

On a clean, low-delay path this formula sits near 4.4 for almost every sample. It will not separate two good paths. It drops when the UDP loss term rises.

### Synthetic SIP

Each attempt is SIPp's built-in caller: INVITE, 100, 180, 200, ACK, BYE. The long-run settings are 20 calls at 1 call per second. The checker uses 3 calls. The client records attempts, completions, failures, and success rate.

The config has a 3-second hold. `sip_sim_client.py` accepts that setting and does not pass it to SIPp, so the built-in scenario's own pause is what runs.

A failed SIP row means this lab scenario did not complete. It does not by itself mean customer voice failed on that access network.

## Timers the code was written with

The config file shipped with the client says:

| Test | Configured interval |
| --- | --- |
| Availability | 60 s |
| RTT, DNS, TCP, TLS, TTFB | 5 min |
| UDP jitter and loss | 15 min |
| Voice pattern | 15 min |
| SIP simulation | 15 min |
| Browser page load | 15 min, 3 loads per site |
| Gaming pattern | 30 min |
| Ookla | 60 min |
| iperf3 download and upload | 9,000 s (2 h 30 min) |
| Planned duration | 48 h |

The comment next to the iperf3 setting says "every 30 minutes". The value is 9,000 seconds, which is 2 hours 30 minutes. The execution guide also says 30 minutes. An older README says reliability every 5 seconds, browser and SIP hourly or every few hours. Those documents disagree with each other.

The September files settle the question of what ran. Fiber and 5G fixed wireless used a 30-minute iperf3 interval. Mobile 5G used 9,000 seconds. LEO used 2 hours. Ookla was hourly on fiber and mobile 5G, and every 2 hours on LEO. Availability was every 60 seconds on every probe. See the campaign note.

Because download, upload, UDP, voice, and gaming share one Python process, a scheduled interval is a minimum gap. A 5-minute gaming UDP test delays whatever is next. The availability probe is a separate process and keeps its own 60-second timer.

## Timeouts that are set in the code

An earlier note said timeouts were left to tool defaults. That is not what the code does.

| Step | Timeout in the client |
| --- | --- |
| Availability ping | wait 2 s for the reply; the process is killed if the command runs longer than 4 s |
| RTT ping group | wait 2 s per echo; process limit about 30 s for the group |
| DNS | 3 s per query, 1 try; process limit 10 s |
| Web curl | curl `--max-time 30`; process limit 40 s. HTTP status 200–499 is recorded as pass. |
| iperf3 | test length plus 30 s |
| Ookla | 180 s |
| Chromium navigation | 45 s, then a 1.5 s settle |
| SIPp | SIPp `-timeout 60s`; process limit is the larger of 120 s and 5 s per call |
| Preflight SIP | 20 s |

If the availability command itself exceeds 4 seconds, that process raises an error and stops. It does not write a failed probe in that case. Ordinary ping failures (no reply within 2 s) are written and kept.

## Availability

Every 60 seconds the probe sends one ICMP echo to the central server.

- success is a zero exit from `ping`
- the percentage is successful probes divided by all probes so far, including failures
- an outage is counted when a run of failures ends and the next probe succeeds
- an outage still in progress at the end of the file is not added to the outage count

This is probe success to the lab server. It is not a contractual availability SLA, and it does not identify whether a failure was the access network, the path to the lab, the lab host, or the probe.

## What the collector does not do

- It does not remove outliers. A later spreadsheet step used a 1.5×IQR rule. That rule is not in this software, and the cleaned counts in the draft note do not add back to the original count (71,826 + 5,502 = 77,328, not 77,333). Reconcile those three numbers before they are published.
- It does not measure YouTube, Netflix, or a handset video MOS. Those belong to the separate application campaign.
- It does not record Wi-Fi RSSI, channel, or which interface the probe used.

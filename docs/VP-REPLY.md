# Note to VP

Subject: Central measurement server — method and September results for fiber, 5G, and LEO

This is the answer on the central measurement server built for fixed and mobile broadband benchmarking across fiber, 5G fixed wireless, mobile 5G, and LEO.

The short internal name for the work is **bb-benchmark**.

## What we built

We built one central measurement server and pointed four access probes at it: fiber, 5G fixed wireless, mobile 5G, and Starlink. Throughput, delay, loss, and a synthetic call setup are measured to that same server, on a dedicated port per access network, so the comparison does not depend on which public speed-test server a tool happens to choose.

Public Ookla tests and page loads to the operator site and a news site are collected as well. Those describe what a customer session looks like on the open internet. They are a cross-check, not the primary benchmark.

The server is an Ubuntu 24.04 host with iperf3 3.16 and SIPp 3.7.2, NTP synchronized, timezone Asia/Dubai. Each probe runs the same client: a 60-second availability ping, a 5-minute light cycle (round trip, DNS, web timing), plus TCP throughput, UDP, a voice-pattern test, Ookla, a headless browser, and synthetic SIP. The software records every sample, including failed probes. It does not delete outliers. Any 1.5×IQR cleaning was a later spreadsheet step, not part of the server.

## What I would want you to use

All figures are from the raw 4–6 September files, with no outlier removal. Values are medians.

| Access | Controlled download | Controlled upload | Round trip | Probe success to the lab server |
| --- | ---: | ---: | ---: | ---: |
| Fiber | 713 Mbps (82 tests) | 188 Mbps | 11 ms | 99.8% over ~40 h |
| LEO | 240 Mbps (26 tests) | 12 Mbps | 48 ms | 99.7% over ~46 h |
| Mobile 5G | 328 Mbps (9 tests) | 73 Mbps | 30 ms | 100% over ~18 h |
| 5G fixed wireless | not a single capacity number | — | 21 ms when the ping succeeded | 57.9% over ~46 h |

Fiber is the reference path: stable throughput to the lab, low delay, and probe success of 99.8%.

LEO, to the same server, is about one third of fiber download, about 12 Mbps upload, and about 48 ms round trip. Upload and delay are the gap that matters for an operator comparison, more than the download headline.

Mobile 5G on this probe is about 328 Mbps down and 73 Mbps up. That rests on nine download tests in an 18-hour window. I would not set a strategy number on that sample.

5G fixed wireless is the result that must not be simplified. Public Ookla downloads on that probe stayed near **940 Mbps** (37 tests, all between 926 and 949). The path to our server did not: only **57.9%** of the minute-by-minute probes succeeded, and 35 of 60 "successful" iperf3 downloads transferred under 1 Mbps. The downloads that did transfer (24 tests) had a median of 717 Mbps, with a maximum of 744. Ookla and the lab path are telling different stories. I recommend we explain that path before any fixed-wireless capacity number from the lab server is used outside the team.

## Direct answers to the methodology points

**When.** The files, in UTC: fiber samples from 4 Sep 18:57 to 6 Sep 11:40; fixed wireless from 4 Sep 13:30 to 6 Sep 11:41; LEO from 4 Sep 13:00 to 6 Sep 11:41; mobile 5G from 5 Sep 02:59 to 5 Sep 20:58. The plan was 48 hours. Mobile ran about 18 hours. The draft start times do not all match these files; the ends do, for fixed wireless, LEO, and mobile. Please quote the file windows.

**Equipment.** Fiber CPE Arcadyan eLife Connect C1BA, 5G fixed-wireless CPE Pegatron MG36AX, mobile handset referenced separately as a Samsung Galaxy S25, LEO via the Starlink router. Probes were Dell PowerEdge servers for fiber, fixed wireless, and LEO, and a Lenovo laptop under Windows with Ubuntu on WSL2 for mobile 5G. Tool versions are not identical across those hosts (iperf3 3.9, 3.16, and 3.20). The notes say 5 GHz Wi-Fi on the fiber and Starlink access. They do not record signal level, or prove Ethernet versus Wi-Fi for every probe. I have left that as an open item rather than fill it in.

**How often.** Availability every 60 seconds on all four probes, one ICMP echo, failed probes kept. Round trip, DNS, and web timing every 5 minutes. TCP tests were 60 seconds in each direction. The throughput **interval was not the same**: every 30 minutes on fiber and fixed wireless, every 2 hours 30 minutes on mobile 5G, every 2 hours on LEO. Ookla was hourly on fiber and mobile, and every 2 hours on LEO. A sentence that says every network was tested every 60–120 minutes would not be accurate.

**Timeouts.** They are set in the client. Ping waits 2 seconds. DNS allows 3 seconds and one try. Web requests allow 30 seconds. iperf3 is allowed the test length plus 30 seconds. Ookla is allowed 180 seconds. The browser allows 45 seconds. SIP allows 60 seconds. Failed pings stay in the availability file. This is probe success, not a contractual SLA, and the files do not by themselves name the cause of each failed probe.

**Ookla versus iperf3.** They are different tests. iperf3 ends on our server. Ookla ends on whatever public server the client selected: always e& Sharjah for fiber, a mix of e& sites for fixed wireless and mobile, and mostly du and Melbicom for LEO. On fiber, Ookla download (median 899 Mbps) was higher than iperf3 (median 713). On LEO, Ookla (median 351) was also higher than iperf3 (median 240). The draft pair "fixed wireless Ookla 940 versus iperf3 1,310" should be corrected: Ookla at about 940 Mbps is supported; 1,310 Mbps does not appear in the server files. The same is true of the draft mobile pair 1,542 / 1,590 Mbps. Those values are not in the probe files. If they came from a handset speed test, they need to be labeled as a handset test.

**Cleaning.** The manuscript population is the timestamp workbook in `Data/`: 77,333 KPI records before the screen and 71,831 after it (fiber 21,881, fixed wireless 19,697, mobile 9,471, LEO 20,782). The difference is 5,502. The 49,702 figure is an earlier count and should be replaced. The 1.5×IQR rule was applied separately for each network and KPI. It was not part of the collection scripts. The outlier workbook keeps the availability events that are not all present in the cleaned sheets. For 5G fixed wireless those events are 1,562 passes and 1,138 failures, 57.85%. The lab attributes that period to data-package exhaustion, and a fiber router problem to some excluded fiber rows. The published 0% packet loss and 100% fixed-wireless availability are the selected valid rows, not a statement that the full probe log had no failures. The raw events should stay in the outlier file.

**Voice.** The MOS near 4.4 on fiber, mobile, and LEO is a formula from delay and loss, for comparison only. It is not a listening score and it is not Google UVQ. Synthetic SIP completed only from the LEO probe (87% of 3,646 calls). Fiber, fixed wireless, and mobile completed none. That points to the lab listeners on those three ports, not to a customer voice outage.

**TCP, TLS, and TTFB.** The client measures these with curl. TCP time is `time_connect`. TTFB is `time_starttransfer`. TLS is `time_appconnect` minus `time_connect`. TCP and TTFB are cumulative. TLS is the extra time after TCP. Curl follows redirects. A fixed-wireless example in the timestamp file is 64.610 ms TCP and 72.817 ms TLS. Those are separate stages. They are not added into one handshake of 0.41 ms.

**Application QoE, YouTube, Netflix, and UVQ.** These are in `docs/05-application-qoe.md` and were not collected by the central server. The application workbook matches the manuscript averages: mobile is shortest on Facebook (1.793 s), Instagram (1.608 s), both call setups, and Teams (5.054 s). Fiber is shortest on WhatsApp (1.039 s). The application campaign has 236 retained samples, not 300. YouTube OTT startup is 1.361, 1.494, 1.399, and 1.657 s, with no rebuffering and no dropped frames. Fiber stays at 1080p60 on all five runs. Mobile reaches 1080p on one of five. Netflix is 720p AV1 on every run, VMAF 89.0 to 89.8, fiber delivery 165.7 Mb/s, LEO delivery 18.2 Mb/s. Google UVQ on the recorded GoChat and BOTIM videos is about 3.68 and 3.58 on fiber, 3.72 and 3.50 on LEO, and 3.76 and 3.60 on mobile. The fixed-wireless UVQ rows are the same files and the same scores as mobile. Do not present them as an independent fixed-wireless video result until the recording assignment is checked.

**Web page timing.** On fiber and mobile 5G, every automated load of the operator site came back with the title "Request Rejected". Those timings are a block page, not the homepage. I would not use them as a customer page-load KPI.

## What this supports for operator strategy

The platform is fit for a like-for-like comparison of fiber, mobile 5G, and LEO **to our own server**, provided we keep the unequal test counts visible: fiber has 82 download tests, LEO 26, mobile 9.

It supports a clear LEO statement from this campaign: to the same server, LEO download was about 240 Mbps and upload about 12 Mbps, with delay around 48 ms, against fiber at about 713 Mbps, 188 Mbps, and 11 ms.

It does not yet support a fixed-wireless capacity claim from the lab server. Public speed tests on that probe were consistently near 940 Mbps, and the lab path was not. Those results should be presented side by side, with the investigation of the lab path called out, rather than collapsed into one speed.

I can walk through the port plan, the KPI list, and these tables whenever useful. The raw files and the server address are held separately from this write-up.

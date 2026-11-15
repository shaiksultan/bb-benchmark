# Client and server setup

Do the server once. Then repeat the client steps on each access probe: LEO, fiber, 5G fixed wireless, and mobile 5G. Use the same client package on every probe, and point all four at the same central server.

The package does not include `start_4_iperf_servers.sh` or `start_sip_sim_server.sh`. The commands below are the listeners those guides describe. The client side is the scripts that are in the package: `SETUP_WSL_CLIENT.sh`, `CONFIGURE_CLIENT.sh`, `VERIFY_CLIENT.sh`, `PREFLIGHT_SERVER.sh`, `RUN_CHECK.sh`, and `RUN_48H.sh`.

Replace `<SERVER>` with the central server address. Do not use `localhost` for a real campaign.

## 1. Central server

On the central host, as a user who can install packages and open the firewall:

```bash
sudo apt update
sudo apt install -y iperf3 sip-tester
```

Confirm the clock is synchronized. The September hosts used NTP and timezone Asia/Dubai. Probe timestamps are only comparable when every host agrees on the time.

```bash
timedatectl
```

Start one iperf3 listener per access network. One iperf3 process serves one test at a time, so four probes need four listeners.

```bash
nohup iperf3 -s -p 5201 > starlink_iperf.log 2>&1 &
nohup iperf3 -s -p 5202 > fiber_iperf.log 2>&1 &
nohup iperf3 -s -p 5203 > fwa_iperf.log 2>&1 &
nohup iperf3 -s -p 5204 > mobile5g_iperf.log 2>&1 &
```

Start one SIPp answering scenario per access network. The client calls the built-in caller (`sipp -sn uac`). It completes only if this built-in answerer is listening on the matching port.

```bash
nohup sipp -sn uas -p 5061 > starlink_sip.log 2>&1 &
nohup sipp -sn uas -p 5062 > fiber_sip.log 2>&1 &
nohup sipp -sn uas -p 5063 > fwa_sip.log 2>&1 &
nohup sipp -sn uas -p 5064 > mobile5g_sip.log 2>&1 &
```

| Access | iperf3 | SIP |
| --- | --- | --- |
| LEO (Starlink) | 5201 TCP and UDP | 5061 UDP |
| Fiber | 5202 TCP and UDP | 5062 UDP |
| 5G fixed wireless | 5203 TCP and UDP | 5063 UDP |
| Mobile 5G | 5204 TCP and UDP | 5064 UDP |

Open those ports, and allow ICMP, because availability and round-trip both use ping.

```bash
sudo ufw allow 5201:5204/tcp
sudo ufw allow 5201:5204/udp
sudo ufw allow 5061:5064/udp
```

Check that all eight listeners are up before any client starts:

```bash
ss -lntup | grep -E ':5201|:5202|:5203|:5204'
ss -lunp  | grep -E ':5061|:5062|:5063|:5064'
```

Leave the listeners running for the whole campaign. In the September files, synthetic SIP completed only on port 5061. The other three ports failed every call. Confirm all four SIP listeners answer before the long run, using the client preflight in section 3.

## 2. Each client

The client is Ubuntu, or Ubuntu under WSL2. Copy the same extracted package onto the probe and open a shell in that folder.

```bash
chmod +x *.sh *.py
./SETUP_WSL_CLIENT.sh
```

That installer:

1. Installs iperf3, ping, dig, curl, Python, SIPp, and the libraries Chromium needs.
2. Removes the Debian `speedtest-cli` package if it is present, because that is not the Ookla client.
3. Installs the official Ookla `speedtest` package.
4. Creates `.venv` in the package folder and installs Playwright there.
5. Downloads Chromium into `.playwright-browsers` in the same folder.
6. Launches Chromium once to prove it starts.

`client_install.sh` is the shorter installer. It does not create the virtual environment and it does not install Ookla. Use `SETUP_WSL_CLIENT.sh` for a campaign client.

Choose which access network this machine represents, and type the central server address. The same address must be entered on all four probes. The script rewrites `IPERF_SERVER`, `PING_TARGET`, and `SIP_SERVER` in `benchmark.conf`, keeps a dated backup of that file, and writes `.client_profile`.

```bash
./CONFIGURE_CLIENT.sh
```

| Choice | Network | Scripts it selects |
| --- | --- | --- |
| 1 | LEO (Starlink) | `01_starlink_full.sh` |
| 2 | Fiber | `02_eand_fiber_full.sh` |
| 3 | 5G fixed wireless | `03_eand_5g_fwa_full.sh` |
| 4 | Mobile 5G | `04_eand_mobile_5g_full.sh` |

Before a long run, read the intervals in `benchmark.conf` and set the values you intend to publish. The file shipped with the package sets iperf3 to `9000` seconds, which is 2 hours 30 minutes, while the comment on that line says 30 minutes. The September probes did not all use the same throughput interval. Set one interval, put that same file on every probe, and record it with the results.

Keep the probe on AC power, and turn off sleep and hibernation. Keep Windows Update, OneDrive, and other large background transfers off for the run.

## 3. Prove the client can reach the server

On each probe:

```bash
./VERIFY_CLIENT.sh
./PREFLIGHT_SERVER.sh
./RUN_CHECK.sh
```

`VERIFY_CLIENT.sh` checks the local tools: Python in `.venv`, iperf3, SIPp, Ookla, Chromium, DNS, and HTTPS. It also prints the profile and server from `.client_profile`.

`PREFLIGHT_SERVER.sh` checks the path to the central server:

- three pings
- a 2-second iperf3 test on this network's port
- one SIPp call on this network's SIP port, with a 20-second timeout
- one HTTPS fetch of the operator site and one of the news site

`RUN_CHECK.sh` runs the short functional test: round trip, DNS, web timing, a 10-second download, a 10-second upload, a short UDP test, one Ookla test, one browser pass, and three synthetic SIP calls.

Start the 48-hour run only after preflight prints `IPERF: OK` and `SIP: OK` on every probe. A SIP failure here is the same failure that left fiber, fixed wireless, and mobile 5G with zero completed calls in September.

## 4. Start the campaign

Start the four probes close together, after the server listeners are up.

```bash
./RUN_48H.sh
```

That writes a pid file and a console log under `results/<network>/`, then starts the full client. The full client starts the main benchmark and the availability probe, and on their own timers adds Ookla, the browser, and SIP.

Watch one probe:

```bash
tail -f results/<network>/client_48h_console.log
tail -f results/<network>/benchmark.log
tail -f results/<network>/reliability.log
tail -f results/<network>/speedtest.log
tail -f results/<network>/browser.log
tail -f results/<network>/sip_sim.log
```

Result files in that same folder:

| File | Contents |
| --- | --- |
| `samples.csv` | throughput, delay, loss, DNS, web timing, voice and gaming patterns |
| `reliability.csv` | one row per availability probe |
| `speedtest_samples.csv` | Ookla |
| `browser_samples.csv` | Chromium page timing |
| `sip_sim_samples.csv` | synthetic SIP attempts |

Stop one probe:

```bash
kill "$(cat results/<network>/client_48h.pid)"
pkill -f benchmark_48h.py
pkill -f reliability_monitor.py
pkill -f ookla_test.py
pkill -f browser_test.py
pkill -f sip_sim_client.py
```

Run those `pkill` commands on the probe you are stopping. They match by script name and will stop every copy of those scripts on that machine.

## 5. Order of operations

1. Install iperf3 and SIPp on the central server.
2. Start listeners on 5201–5204 and 5061–5064, and confirm them with `ss`.
3. Install the client package on each probe with `SETUP_WSL_CLIENT.sh`.
4. Run `CONFIGURE_CLIENT.sh` with the same server address and a different network on each probe.
5. Make the intervals in `benchmark.conf` identical on all four probes.
6. Pass `VERIFY_CLIENT.sh`, `PREFLIGHT_SERVER.sh`, and `RUN_CHECK.sh` on each probe.
7. Start `RUN_48H.sh` on all four while the server listeners are still up.

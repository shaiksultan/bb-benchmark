# Application QoE, YouTube, Netflix, and Google UVQ

These figures are calculated from the workbooks in the public repository [bb-benchmark-data](https://github.com/shaiksultan/bb-benchmark-data). Application timing has no statistical outlier removal. Network voice MOS is a separate estimate from UDP delay and loss. Google UVQ is an offline score of recorded video, not a listening test.

| Workbook | What it holds |
| --- | --- |
| `data/application-qoe.xlsx` | Startup, first content, call setup, WhatsApp audio receipt, Teams join, and the separate application-campaign YouTube startup |
| `data/youtube-netflix-ott.xlsx` | Five YouTube and five Netflix playback runs per access path |
| `data/video-quality-uvq.xlsx` | Google UVQ and the lab composite score for GoChat and BOTIM recordings |

The network KPI workbooks in the same folder are the probe campaign. They do not contain these application times.

## How the application tests were run

The lab measured Facebook, Instagram, X, TikTok, WhatsApp, GoChat, BOTIM, Microsoft Teams, and YouTube on each access path: fiber, 5G fixed wireless, mobile 5G, and LEO.

Recorded actions are first content or feed load, call setup, WhatsApp audio receipt, Teams join, and YouTube startup. Most cells are 5 runs. Mobile and LEO Instagram use 18 runs, and mobile and LEO TikTok use 20, because later repetition sheets replaced the first 5-run set. The workbook retains 236 samples. That is the application population behind the averages below, not a count of 300.

GoChat and BOTIM were about one-minute video calls. Party B recorded the received video on a Samsung Galaxy S25. Those files were scored offline with Google UVQ. The lab composite also uses BRISQUE, sharpness, frame rate, and smoothness. UVQ is a picture-quality score. It is not PESQ or POLQA, and it is not the network voice MOS from the probe scripts.

YouTube OTT is a separate campaign from the application-campaign startup times. Five samples per network, about five minutes each, were read from YouTube Stats for Nerds: startup, resolution, frame rate, rebuffering, and dropped frames. The UVQ column in that workbook is empty.

Netflix OTT is five samples per network, about five minutes each, from the player diagnostics: representation, bitrate, buffer, dropped frames, and the player-reported VMAF.

## Application timing

Means in seconds. Lower is better. Source sheet: `Application QoE Summary`.

| Action | Fiber | 5G fixed wireless | Mobile 5G | LEO |
| --- | ---: | ---: | ---: | ---: |
| Facebook, first content | 2.347 | 2.313 | 1.793 | 2.163 |
| Instagram, first content | 2.107 | 2.013 | 1.608 | 2.508 |
| X, first content | 3.349 | 2.330 | 3.720 | 2.770 |
| TikTok feed | 2.826 | 2.510 | 2.396 | 2.377 |
| WhatsApp, audio received | 1.039 | 1.223 | 1.245 | 1.397 |
| BOTIM, call setup | 2.740 | 3.040 | 2.633 | 2.850 |
| GoChat, call setup | 4.807 | 4.996 | 4.758 | 5.413 |
| Teams, join | 5.380 | 6.120 | 5.054 | 7.697 |
| YouTube startup, application campaign | 1.096 | 3.737 | 1.604 | 1.350 |

Mobile 5G is the shortest on Facebook, Instagram, both call setups, and Teams. Fiber is the shortest on WhatsApp. Fixed wireless is the shortest on X. LEO and mobile TikTok differ by 0.019 s. These averages match the manuscript. They do not include a run-level confidence interval.

Teams quality scores of 4.000 and 3.400 are not in this workbook. The application YouTube file records startup time only. It does not record the 2 s and 3 s rebuffering durations mentioned in the manuscript.

## YouTube OTT

Twenty runs, five per path. Sheet: `CTO Summary` and `YouTube Raw Data`.

| Result | Fiber | 5G fixed wireless | Mobile 5G | LEO |
| --- | ---: | ---: | ---: | ---: |
| Startup (s) | 1.361 | 1.494 | 1.399 | 1.657 |
| Runs that reach 1080p | 5 of 5 | 5 of 5 | 1 of 5 | 4 of 5 |
| Runs that stay 1080p60 | 5 of 5 | 3 of 5 | 1 of 5 | 2 of 5 |
| Rebuffer events | 0 | 0 | 0 | 0 |
| Dropped frames | 0 | 0 | 0 | 0 |

Fiber is the only path that stays at 1080p60 for all five runs. Fixed wireless reaches 1080p in every run, and stays there for three of the five. This campaign is separate from the application-campaign startup row above. Fixed wireless startup is 3.737 s in the application campaign and 1.494 s here. Those two means should not be averaged.

## Netflix OTT

Twenty runs, all 1280×720 AV1, all dropped-frame counts zero.

| Result | Fiber | 5G fixed wireless | Mobile 5G | LEO |
| --- | ---: | ---: | ---: | ---: |
| Player VMAF | 89.8 | 89.0 | 89.8 | 89.0 |
| Playing bitrate (Mb/s) | 0.410 | 0.425 | 0.853 | 0.533 |
| Delivery throughput (Mb/s) | 165.7 | 44.2 | 30.3 | 18.2 |
| Buffer (s) | 219.9 | 230.5 | 228.2 | 231.2 |

LEO has the largest buffer. Mobile has the largest playing bitrate. Neither is a picture-quality ranking. VMAF sits in a narrow band, 89.0 to 89.8. Netflix startup times of 1.719 s and 2.525 s are not columns in this workbook.

## Google UVQ for GoChat and BOTIM

Means of five recordings per path. Source: `master_benchmarkingmos_results(1).xlsx`.

| Path in the video file | BOTIM UVQ | GoChat UVQ | BOTIM lab composite | GoChat lab composite |
| --- | ---: | ---: | ---: | ---: |
| Fiber | 3.680 | 3.579 | 3.819 | 3.769 |
| LEO | 3.720 | 3.496 | 3.795 | 3.752 |
| Mobile 5G | 3.756 | 3.598 | 3.261 | 3.673 |
| Mobile FWS | 3.756 | 3.598 | 3.261 | 3.673 |

Mobile 5G and Mobile FWS are the same filenames and the same scores on every recording. The fixed-wireless UVQ figures in the summary workbook are that copy. They are not a second, independent fixed-wireless video set. Until the original file assignment is checked, report fiber, LEO, and mobile only, and do not say that fixed wireless and mobile independently share the highest UVQ.

The lab composite is not a voice MOS. The workbook states that voice MOS was not calculated from the video.

## Network voice estimate, kept separate

The probe script estimates a voice MOS from ping delay and UDP loss. On the cleaned network sheets that estimate is about 4.41 on fiber, 4.40 on mobile, and 4.40 on LEO. Fixed wireless is about 3.78, because the loss term is still high on the cleaned voice-loss rows. That number is a network formula. It is not the UVQ score and it is not a listening score.

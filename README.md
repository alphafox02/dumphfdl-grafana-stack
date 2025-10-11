# dumphfdl-grafana-stack

A self-contained observability stack for dumphfdl --- visualize decoded
HFDL aircraft data in Grafana using Graphite, Loki, and Promtail via
Docker.

------------------------------------------------------------------------

## Overview

This stack connects dumphfdl's built-in StatsD and JSON outputs to a
monitoring pipeline:

     ┌──────────┐        StatsD        ┌──────────────┐
     │ dumphfdl │ ───────────────────▶ │  Graphite    │
     │ (HF data)│                     └──────────────┘
     │          │        JSON logs     ┌──────────────┐
     │          │ ───────────────────▶ │  Promtail    │
     └──────────┘                      │     │        │
                                       ▼     ▼
                                     ┌──────────────┐
                                     │   Loki +     │
                                     │   Grafana    │
                                     └──────────────┘

------------------------------------------------------------------------

## Requirements

-   Docker and Docker Compose installed
-   dumphfdl built with SoapySDR (e.g., AirspyHF, RTL-SDR, etc.)
-   Access to an HF receiver and antenna

------------------------------------------------------------------------

## Quick Start

Clone and start the stack:

``` bash
git clone https://github.com/yourusername/dumphfdl-grafana-stack.git
cd dumphfdl-grafana-stack
docker compose up -d
```

Then run dumphfdl and point its outputs to the stack:

``` bash
dumphfdl --soapysdr driver=airspyhf --sample-rate 768000   --centerfreq 8891 8834 8885 8894 8912 8927 8939 8942 8948   --statsd 127.0.0.1:8125   --system-table /usr/src/dumphfdl/etc/systable.conf   --output decoded:json:file:path=/tmp/hfdl.jsonl,rotate=hourly
```

Once running:

-   Grafana → http://localhost:3000 (admin / admin)
-   Data sources auto-configured for Graphite and Loki
-   Import the example dashboard (`dashboard-hfdl.json`) to visualize
    live HFDL metrics and logs

------------------------------------------------------------------------

## Repository Layout

    dumphfdl-grafana-stack/
    ├── docker-compose.yml          # Graphite, Grafana, Loki, Promtail setup
    ├── promtail-config.yaml        # JSON parsing + labeling stages
    ├── dashboard-hfdl.json         # Example Grafana dashboard
    └── README.md                   # This file

------------------------------------------------------------------------

## Troubleshooting

**No metrics in Grafana:**\
- Check dumphfdl is running with `--statsd 127.0.0.1:8125`\
- Run `docker logs graphite` to verify StatsD metrics

**No logs in Grafana (Loki):**\
- Ensure Promtail is tailing `/tmp/hfdl.jsonl` (check
`docker logs promtail`)\
- Confirm that file exists and has readable JSON lines

**Permission issues:**\
- `/tmp/hfdl.jsonl` must be world-readable, or mounted into `/logs`
inside the Promtail container

**Disk usage:**\
- Rotate logs hourly or daily (already configured in dumphfdl). Loki
stores compressed data; older chunks can be purged if needed.

------------------------------------------------------------------------

## Extending the Stack

Ideas for expansion:

-   Add InfluxDB instead of Graphite
-   Integrate a map panel showing decoded aircraft positions
-   Combine with VDL2 or ACARS decoders
-   Build automatic dashboards per receiver station

------------------------------------------------------------------------

## Credits

Created by **cemaxecuter**\
Tested on DragonOS with AirspyHF receiver

------------------------------------------------------------------------

## License

MIT License

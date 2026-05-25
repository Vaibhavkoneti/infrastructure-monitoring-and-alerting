# Infrastructure Monitoring & Alerting Stack

A production-ready observability stack built to detect, visualise, and alert on infrastructure health in real time. This project goes beyond basic setup — it implements a full monitoring pipeline with custom alert rules, Slack notifications, and persistent dashboards, all orchestrated via Docker Compose.

Built to demonstrate hands-on experience with the tools that operations and data engineering teams rely on every day: **Prometheus**, **Grafana**, **Node Exporter**, and **Alertmanager**.

---

## What This Project Does

Deployed a self-contained monitoring system that:

- **Scrapes** host-level system metrics every **15 seconds** using Prometheus + Node Exporter
- - **Visualises** CPU, memory, disk, and network metrics on Grafana dashboards with persistent storage
  - - **Triggers** automated alerts based on configurable thresholds — CPU spike detection (>80%) and instance availability checks
    - - **Routes** alert notifications to a Slack channel (#monitoring_alert) via Alertmanager webhooks with intelligent grouping and repeat suppression
      - - **Recovers gracefully** — sends resolve notifications automatically when incidents clear (within 5 minutes)
       
        - ---

        ## Architecture

        ```
        Host System
             |
        Node Exporter (port 9100)  -- Exposes /proc, /sys, host metrics
             |
        Prometheus (port 9090)  -- Scrapes every 15s, evaluates alert rules
             |
             |---> Alertmanager (port 9093)
             |           Groups alerts, suppresses repeats (12h interval)
             |           --> Slack #monitoring_alert
             |
        Grafana (port 3000)  -- Queries Prometheus, renders dashboards
        ```

        ---

        ## Alert Rules

        Defined in `alert_rules.yml` using PromQL expressions:

        | Alert | Condition | Severity | Fires After |
        |---|---|---|---|
        | **InstanceDown** | `up == 0` — target unreachable | Critical | 1 minute |
        | **HighCPUAlert** | CPU user mode sustained above 80% | Warning | 2 minutes |

        CPU usage is calculated using a rate-based PromQL expression:

        ```promql
        avg(rate(node_cpu_seconds_total{mode="user"}[1m])) * 100
        ```

        ---

        ## Key Metrics Monitored

        Collected via Node Exporter and scraped by Prometheus every 15 seconds:

        - **CPU** — Usage by mode (user, system, idle): `node_cpu_seconds_total`
        - - **Memory** — Available vs used: `node_memory_MemAvailable_bytes`
          - - **Disk** — I/O read/write throughput: `node_disk_read_bytes_total`
            - - **Network** — Traffic in/out per interface: `node_network_receive_bytes_total`
              - - **Uptime** — System boot time: `node_boot_time_seconds`
                - - **Instance health** — Scrape target availability: `up`
                 
                  - ---

                  ## Tech Stack

                  | Tool | Role |
                  |---|---|
                  | **Prometheus** | Metrics collection, time-series storage, and alerting engine |
                  | **Grafana** | Dashboard visualisation and metric exploration |
                  | **Node Exporter** | Host-level metrics exporter via /proc and /sys |
                  | **Alertmanager** | Alert routing, deduplication, grouping, and Slack delivery |
                  | **Docker Compose** | Multi-service orchestration and reproducible local deployment |
                  | **PromQL** | Query language for metric filtering, aggregation, and rate calculations |

                  ---

                  ## Alertmanager Behaviour

                  Configured to handle real-world alert fatigue:

                  - **Group wait**: 30s — batches related alerts before sending
                  - - **Group interval**: 5 min — prevents flooding during ongoing incidents
                    - - **Repeat interval**: 12h — suppresses duplicate notifications
                      - - **Resolve timeout**: 5 min — auto-closes incidents when metrics recover
                        - - **send_resolved: true** — notifies Slack when an alert clears
                         
                          - ---

                          ## Getting Started

                          Requires Docker and Docker Compose.

                          ```bash
                          # Clone the repo
                          git clone https://github.com/Vaibhavkoneti/infrastructure-monitoring-and-alerting.git
                          cd infrastructure-monitoring-and-alerting

                          # Configure Slack alerting in alertmanager.yml
                          # Replace: api_url: "REPLACE_WITH_YOUR_WEBHOOK"

                          # Launch the full stack
                          docker-compose up -d
                          ```

                          | Service | URL | Default Login |
                          |---|---|---|
                          | Prometheus | http://localhost:9090 | — |
                          | Grafana | http://localhost:3000 | admin / admin |
                          | Alertmanager | http://localhost:9093 | — |

                          Recommended: Import the **Node Exporter Full** dashboard (Grafana ID: **1860**) for comprehensive host visibility.

                          ```bash
                          # Shut down the stack
                          docker-compose down
                          ```

                          ---

                          ## Why I Built This

                          Most monitoring tutorials stop at "Prometheus is now running." I wanted to understand what happens next — how do you define meaningful thresholds, avoid alert storms, and make sure the right signal reaches the right person at the right time? Working through Alertmanager's grouping logic, writing PromQL rate expressions, and testing the full Slack notification path gave me a practical understanding of observability that goes beyond just knowing the tools exist.

                          ---

                          Feedback welcome — open to ideas on extending this with more alert rules or a custom Grafana dashboard config.

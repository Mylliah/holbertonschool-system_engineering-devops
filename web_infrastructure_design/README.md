# Web Infrastructure Design

## Overview
This repository contains whiteboard designs and concise explanations for four tasks that progressively evolve a simple web stack into a secure, monitored, and highly available architecture for **www.foobar.com**.  
All explanations are intentionally brief and focused on deliverables. Replace the placeholder screenshot links with your own (Imgur or any host).

## Components & Roles
- **DNS**: Resolves `www.foobar.com` to a public IP or VIP.
- **Load Balancer**: **HAProxy** front door; distributes traffic to healthy backends.
- **Web server**: **Nginx** serves static content and reverse‑proxies to the app tier.
- **Application server**: Runs business logic (e.g., **PHP-FPM** / **Gunicorn** / **uWSGI**).
- **Application files**: Codebase deployed on disk or artifact store.
- **Database**: **MySQL**; Primary (writer) with optional Replica(s) for reads.
- **Firewall**: Per-host least‑privilege rules (ingress/egress).
- **TLS/SSL cert**: Enables HTTPS, HSTS, and removes browser warnings.
- **Monitoring agent**: Ships metrics/logs to SaaS (Datadog/New Relic/Sumo, etc.).
- **VIP/VRRP (keepalived)**: Shared virtual IP for LB cluster failover.

## Redundancy & SPOF
- **Single machine** (Task 0) → **SPOF** for web/app/db; mitigation: add LB + multiple backends and split tiers (Tasks 1–3).
- **Single LB** (Task 1–2) → **SPOF** at the edge; mitigation: LB cluster with **VRRP** and a **VIP** (Task 3).
- **Single DB writer** → write-path **SPOF**; mitigations: failover plan, backups, semi-sync, or managed HA.
- **Backend cleartext leg** with TLS terminated at LB; mitigation: re-encrypt to backends or mTLS.
- **All-in-one backends** cause resource contention; mitigation: split Web/App/DB and scale independently.

## Deployments Without Downtime
- **Blue‑Green**: Run `blue` and `green` environments; switch at the LB/VIP.
- **Rolling**: Gradually drain and update instances behind the LB.
- **Canary**: Route a small % of traffic to new version; expand on success.
- **DB migrations**: Backward‑compatible, online schema change (e.g., gh-ost/pt-osc), feature flags.

## Acronyms
- **LAMP**: **L**inux, **A**pache/**N**ginx, **M**ySQL/MariaDB, **P**HP/Python/Perl stack.
- **SPOF**: **S**ingle **P**oint **o**f **F**ailure; one component whose failure breaks the service.
- **QPS**: **Q**ueries **P**er **S**econd; rate of requests handled by the system.

## Screenshots
- Task 0: [link-to-your-screenshot-task0]
- Task 1: [link-to-your-screenshot-task1]
- Task 2: [link-to-your-screenshot-task2]
- Task 3: [link-to-your-screenshot-task3]

## Tasks
### Task 0 — Simple web stack
- **Goal**: One-server stack reachable at `www.foobar.com`.
- **Components** (single server `8.8.8.8`): **Nginx**, **App server**, **App files**, **MySQL**, **DNS A** record `www.foobar.com → 8.8.8.8`.
- **Flow**: DNS resolves → browser sends HTTP/HTTPS to Nginx → reverse proxy to app → optional MySQL query.
- **Issues**: **SPOF** (one host), **maintenance downtime**, **no scale**.
- **Screenshot**: [link-to-your-screenshot-task0]

### Task 1 — Distributed web infrastructure (3 servers)
- **Goal**: Add a **Load Balancer** and two backends.
- **Added**: 1× **HAProxy** (front door), 2× backends (each: **Nginx + App + Code + MySQL**).
- **LB**: Round‑Robin; **Active‑Active** at web/app layer.
- **DB topology**: **Primary → Replica** (writes → Primary; reads → optional Replica; eventual consistency).
- **Issues**: **SPOFs** remain (single LB, single writer), **no HTTPS**, **no monitoring**.
- **Screenshot**: [link-to-your-screenshot-task1]

### Task 2 — Secured & monitored distributed infrastructure (3 servers)
- **Goal**: Encrypted traffic, host firewalls, and monitoring.
- **Added**: Per‑host **firewalls**, **TLS cert** (HTTPS at LB), **monitoring agents** (CPU/RAM/latency/5xx/**QPS** + logs).
- **QPS collection**: Nginx `stub_status`/VTS or access log parsing → aggregated in monitoring.
- **Issues**: TLS only at LB (backend leg may be cleartext), single DB writer, all‑in‑one backends.
- **Screenshot**: [link-to-your-screenshot-task2]

### Task 3 — Scale up (LB cluster + split tiers)
- **Goal**: Remove LB **SPOF** and split roles for independent scaling.
- **Added**: Second **HAProxy** + **VIP** via **VRRP/keepalived** (**Active‑Passive**).
- **Split**: Dedicated **Nginx** web tier, dedicated app tier, dedicated **MySQL** Primary (+ optional replicas).
- **Scaling**: Add Nginx nodes for **QPS**; add app instances for CPU; scale DB vertically and/or with read replicas.
- **Screenshot**: [link-to-your-screenshot-task3]

## Quick Reference
### DNS
- `www.foobar.com` → **A** record → public IP (Task 0: server IP; Task 1+: LB/VIP IP).

### Common Ports
- **53** DNS (UDP/TCP), **80** HTTP, **443** HTTPS, **3306** MySQL.
- **Nginx ↔ App**: HTTP/FastCGI/uWSGI (local socket or internal TCP).

### LB Distribution Algorithms (examples)
- **Round‑Robin**: cycles evenly.
- **Least‑Connections**: chooses backend with fewest active connections.
- **IP‑Hash**: client IP stickiness (basic session affinity).

### HA Modes (LB cluster)
- **Active‑Passive**: one serves, one stands by; simple; shared **VIP** via **VRRP**.
- **Active‑Active**: both serve (requires DNS RR/Anycast/ECMP and careful state handling).

### Typical SPOFs to Cite
- Single **LB** (fixed in Task 3 via cluster).
- Single **write‑capable DB** node.
- Single switch/router/ISP (usually out of scope for these tasks).

## Directory
- `web_infrastructure_design/`

## Files
- `0-simple_web_stack` — link to Task 0 screenshot
- `1-distributed_web_infrastructure` — link to Task 1 screenshot
- `2-secured_and_monitored_web_infrastructure` — link to Task 2 screenshot
- `3-scale_up` — link to Task 3 screenshot

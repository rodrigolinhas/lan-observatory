# Product Roadmap — LAN Observatory

This roadmap details the sequential engineering milestones for **LAN Observatory** (`lanobs`). Each milestone delivers a complete, verifiable capability following our vertical-slice methodology.

---

## Milestone Dependency Flow

```text
  [M0: Foundation]            [M1: Discovery]              [M2: Inventory]
 ┌─────────────────┐         ┌────────────────────┐       ┌──────────────────────┐
 │ • Go CLI root   │ ──────► │ • CIDR validation  │ ────► │ • Device model       │
 │ • CI & tooling  │         │ • Worker pool ICMP │       │ • MAC / OUI lookup   │
 │ • slog logging  │         │ • Table / JSON CLI │       │ • First/last seen    │
 └─────────────────┘         └────────────────────┘       └──────────────────────┘
                                                                     │
 ┌───────────────────────────────────────────────────────────────────┘
 ▼
  [M3: Persistent History]    [M4: Continuous Observation] [M5: Interactive TUI]
 ┌────────────────────────┐  ┌───────────────────────────┐┌──────────────────────┐
 │ • PostgreSQL & pgx     │─►│ • Scheduled runs          ││ • Bubble Tea UI      │
 │ • sqlc query engine    │  │ • Baseline diffing        ││ • Device drilldown   │
 │ • golang-migrate DDL   │  │ • Change event stream     ││ • Live change stream │
 └────────────────────────┘  └───────────────────────────┘└──────────────────────┘
                                           │
 ┌─────────────────────────────────────────┘
 ▼
  [M6: Local Intelligence]    [M7: Service Observation]   [M8: PCAP Analysis]
 ┌────────────────────────┐  ┌──────────────────────────┐┌──────────────────────┐
 │ • ARP cache inspection │─►│ • Safe service baseline  ││ • Offline pcap parse │
 │ • mDNS discovery       │  │ • Port drift detection   ││ • Host traffic audit │
 │ • Subnet metadata      │  │ • Non-invasive probing   ││ • Baseline validation│
 └────────────────────────┘  └──────────────────────────┘└──────────────────────┘
                                           │
 ┌─────────────────────────────────────────┘
 ▼
  [M9: Defensive Detection]   [M10: Linux Distribution]
 ┌────────────────────────┐  ┌──────────────────────────┐
 │ • Anomaly detection    │─►│ • Systemd daemon mode    │
 │ • Policy alerts        │  │ • Debian packaging (.deb)│
 │ • Rogue device findings│  │ • ARM64 / Pi builds      │
 └────────────────────────┘  └──────────────────────────┘
```

---

## Milestone Specifications

### M0 — Project Foundation *(Current)*
* **Objective**: Establish the core repository foundation, directory conventions, Go module scaffolding, Cobra CLI skeleton, structured logging, CI pipeline, and security boundaries.
* **Key Deliverables**:
  * Project governance (`CONTRIBUTING.md`, `AGENTS.md`, `SECURITY.md`, `docs/`).
  * GitHub Actions CI pipeline (`gofmt`, `go vet`, `go test -race`, `govulncheck`, `go build`).
  * Standardized error handling and `log/slog` logging configuration.
* **Acceptance Criteria**: Repository foundation established, CI configured, documentation fully aligned.

---

### M1 — Network Discovery *(First Operational Vertical Slice)*
* **Objective**: Implement the first end-to-end probing pipeline from CIDR input to formatted output.
* **Pipeline**:
  `CIDR → validation → bounded concurrent discovery → ICMP observations → normalized results → CLI table/JSON`
* **Key Deliverables**:
  * CIDR subnet parser with address space boundary validation (`internal/discovery`).
  * Bounded worker pool with explicit concurrency controls (e.g. `--concurrency` flag, default 16).
  * ICMP echo prober calculating round-trip time (RTT) and status.
  * Structured output formatters supporting human-readable terminal tables and machine-readable JSON (`--output json`).
* **Acceptance Criteria**: Running `lanobs discover 192.168.1.0/24` correctly identifies responsive hosts and prints structured latency metrics with zero data races.

---

### M2 — Device Inventory
* **Objective**: Transform raw discovery observations into durable, normalized device records.
* **Key Deliverables**:
  * Normalized `Device` domain entity tracking IP, MAC address, hostname, and status.
  * Reverse DNS lookup engine for automated host resolution.
  * OUI (Organizationally Unique Identifier) vendor prefix lookup for MAC addresses.
  * First-seen and last-seen timestamp calculation.
* **Acceptance Criteria**: Output displays enriched hostnames and vendor metadata alongside IP addresses.

---

### M3 — Persistent Network History
* **Objective**: Introduce relational persistence to track network states and discovery runs over time.
* **Key Deliverables**:
  * PostgreSQL integration using `jackc/pgx/v5` connection pool.
  * Versioned SQL migrations managed by `golang-migrate`.
  * Type-safe SQL query generation via `sqlc`.
  * Historical query commands (`lanobs device <ip>`, `lanobs history`).
* **Acceptance Criteria**: Discovery runs are persisted into PostgreSQL, allowing historical queries across runs without schema drift.

---

### M4 — Continuous Observation
* **Objective**: Continuously monitor authorized subnets and detect baseline changes over time.
* **Key Deliverables**:
  * Continuous observation sweep runner (`lanobs watch <cidr> --interval <duration>`).
  * Network diff engine computing device state transitions:
    * `HostJoined`: New IP/MAC observed for the first time.
    * `HostLeft`: Previously active device unresponsive across consecutive sweeps.
    * `IPChanged`: Known MAC address bound to a new IP address.
  * CLI change audit command (`lanobs changes --since 24h`).
* **Acceptance Criteria**: Automated detection and emission of network change events across repeated observation intervals.

---

### M5 — Interactive TUI
* **Objective**: Provide a responsive terminal user interface for interactive network exploration.
* **Key Deliverables**:
  * TUI built with Bubble Tea (`lanobs tui`).
  * Real-time network overview grid with live status indicators.
  * Device detail inspect view showing historical latency and metadata.
  * Live change event stream.
* **Acceptance Criteria**: Fluid terminal UI navigating inventory and events without embedding any network probing code in TUI views.

---

### M6 — Local Network Intelligence
* **Objective**: Enrich device profiles using non-invasive local network signals.
* **Key Deliverables**:
  * OS ARP table cache inspection for rapid MAC discovery on local subnets.
  * Passive mDNS service discovery (`_http._tcp.local`, `_workstation._tcp.local`) for friendly device naming.
  * Subnet gateway and broadcast address resolution.
* **Acceptance Criteria**: Automated populating of local device names and MAC addresses via ARP and mDNS without active port probing.

---

### M7 — Service Observation
* **Objective**: Safely observe legitimate network services and detect baseline port changes.
* **Key Deliverables**:
  * Non-invasive TCP port reachability checking against standard service baselines (e.g. HTTP, HTTPS, SSH, DNS).
  * Port drift detector flagging unexpected new open ports on monitored hosts.
  * Strict policy against intrusive banner grabbing, vulnerability scanning, or exploit fuzzing.
* **Acceptance Criteria**: Reporting of observed services against established baselines with clear drift alerts.

---

### M8 — PCAP Analysis
* **Objective**: Enable offline ingestion and behavioral auditing of packet capture files.
* **Key Deliverables**:
  * CLI command for capture ingestion (`lanobs pcap inspect <file.pcap>`).
  * Extraction of observed hosts, IP-MAC associations, and protocol distributions.
  * Reconciliation of PCAP evidence with established inventory baselines.
* **Acceptance Criteria**: Fast, memory-efficient ingestion of `.pcap` and `.pcapng` files producing normalized inventory records.

---

### M9 — Defensive Detection
* **Objective**: Generate deterministic security findings from network state deviations.
* **Key Deliverables**:
  * Policy evaluation engine checking rules against observed state:
    * Rogue Gateway: Gateway IP claimed by an unrecognized MAC address.
    * MAC Spoofing / IP Collision: Multiple distinct MAC addresses claiming the same IP.
    * Unauthorized Device: Host joined with an unknown vendor prefix on a restricted subnet.
  * Structured finding export (`lanobs findings`).
* **Acceptance Criteria**: Reliable generation of defensive security findings with zero false positives on known-clean test baselines.

---

### M10 — Linux Distribution
* **Objective**: Package LAN Observatory for long-running deployments on Linux servers and appliances.
* **Key Deliverables**:
  * Background daemon mode with systemd unit file templates.
  * Multi-architecture Linux binary builds (`amd64`, `arm64` for Raspberry Pi).
  * Debian package generation (`.deb`) and reproducible release automation via GitHub Actions.
* **Acceptance Criteria**: Seamless systemd service installation and automated service operation on Debian/Ubuntu systems.

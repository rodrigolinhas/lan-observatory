# LAN Observatory

**LAN Observatory** is a defensive network observability tool for discovering, inventorying, and monitoring devices across networks you control.

Designed as a CLI/TUI-first application, LAN Observatory aims to provide more than a snapshot of reachable IP addresses. Its long-term purpose is to build a historical view of a local network: what devices exist, when they appear or disappear, how they change, and what services and network behaviour can be observed over time.

The command-line executable is called **`lanobs`**.

> [!IMPORTANT]
> LAN Observatory is intended for networks you own or are explicitly authorized to analyze. The project focuses on defensive network visibility and observability, not exploitation.

## Vision

Traditional network discovery tools are excellent at answering questions such as:

> What is reachable right now?

LAN Observatory aims to additionally answer:

* What devices normally exist on this network?
* When was a device first or last observed?
* Which devices appeared or disappeared?
* How has the network changed over time?
* What services were previously observed on a device?
* Are there unexpected changes compared with the established network baseline?

The project is intentionally being developed incrementally. New protocols, collectors, persistence layers, or security capabilities are introduced only when they solve a concrete problem.

## Planned capabilities

### Discovery

* IPv4 host discovery
* ICMP observations
* latency measurement
* hostname resolution
* bounded concurrent probing
* cancellation and timeouts

### Inventory

* device inventory
* online/offline state
* first-seen / last-seen timestamps
* device metadata
* network observations

### Historical observability

* persistent observations
* discovery run history
* network change detection
* device state transitions
* historical queries

### Interactive TUI

* network overview
* device inventory
* device details
* live network state
* recent changes

### Future defensive capabilities

* ARP, DNS and mDNS observations
* service observation and baselining
* passive network data sources
* PCAP import and offline analysis
* network policy deviations
* defensive findings
* Linux daemon mode
* Debian packaging
* ARM64 / Raspberry Pi deployments

## CLI

The primary interface is the `lanobs` command.

Planned examples:

```bash
lanobs discover 192.168.1.0/24
lanobs inventory
lanobs device 192.168.1.20
lanobs changes --since 24h
lanobs watch 192.168.1.0/24
lanobs tui
lanobs pcap inspect capture.pcap
```

Commands will be introduced incrementally as their corresponding functionality is implemented.

## Architecture

LAN Observatory is designed around a reusable networking and observability core.

```text
                 CLI
                  │
                 TUI
                  │
         Application Layer
                  │
        Observatory Core
                  │
     ┌────────────┼────────────┐
     │            │            │
 Discovery    Inventory   Observations
     │
 Network primitives
```

The CLI and TUI are interfaces over the same underlying application logic.

This separation allows future interfaces such as a daemon or API without duplicating network discovery and analysis logic.

## Technology

### Core

* Go 1.27+
* Go standard library networking packages
* `golang.org/x/net/icmp`

### CLI

* Cobra

### TUI

* Bubble Tea

### Logging

* `log/slog`

### Testing and quality

* Go testing package
* race detector
* `go vet`
* `govulncheck`
* GitHub Actions

### Persistence

Persistent storage is intentionally not part of the initial discovery milestone.

PostgreSQL may be introduced when historical observations and persistent inventories require it.

## Project principles

### Defensive by design

LAN Observatory is built for defensive analysis of networks under the user's control.

### Observe before analyzing

The project should collect reliable evidence before attempting to derive conclusions from it.

### History matters

The goal is not simply to enumerate devices but to understand how a network changes over time.

### Bounded concurrency

Network operations must use explicit concurrency limits, cancellation, and timeouts.

### Minimal dependencies

New dependencies should solve concrete problems and should not replace straightforward standard-library functionality without good reason.

### No unnecessary complexity

LAN Observatory does not aim to become an Nmap replacement, exploit framework, or collection of unrelated cybersecurity features.

## Roadmap

### M0 — Project Foundation

Bootstrap the Go project, CLI, architecture, CI, documentation, and security model.

### M1 — Network Discovery

Discover reachable devices within an explicitly configured network and report their latency.

### M2 — Device Inventory

Build a normalized inventory containing device identities, hostnames, and observed state.

### M3 — Persistent Network History

Persist devices, observations, and discovery runs to enable historical queries.

### M4 — Continuous Observation

Continuously observe an authorized network and generate meaningful change events.

### M5 — Interactive TUI

Provide an interactive terminal interface for network state, devices, and changes.

### M6 — Local Network Intelligence

Expand observations using appropriate local-network protocols and metadata.

### M7 — Service Observation

Observe services and detect changes relative to established network baselines.

### M8 — PCAP Analysis

Analyze user-provided packet captures offline.

### M9 — Defensive Detection

Generate deterministic security findings from observed network changes and policies.

### M10 — Linux Distribution

Provide reproducible Linux releases, daemon integration, ARM64 builds, and Debian packages.

## Security and authorization

LAN Observatory must only be used on systems and networks that the user owns or is authorized to analyze.

The project does not aim to provide:

* credential attacks
* exploit delivery
* unauthorized access
* persistence on third-party systems
* defensive-evasion functionality
* automated exploitation

Protocol support and service observation are developed for inventory, monitoring, diagnostics, and defensive security analysis.

More details will live in [`docs/security.md`](docs/security.md).

## Development

The project is currently under active early development.

The first target is a small but complete vertical slice:

```text
CIDR
  ↓
validated target network
  ↓
bounded concurrent discovery
  ↓
ICMP observations
  ↓
normalized results
  ↓
CLI table / JSON
```

See the repository milestones and issues for implementation progress.

## License

A project license will be selected before the first public release.

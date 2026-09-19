# Security Policy — LAN Observatory

LAN Observatory is engineered with a strict defensive mission: empowering network owners and operators to observe, inventory, and audit local network environments.

---

## 1. Supported Versions

LAN Observatory is currently in active initial development (`v0.1.0-alpha`). Security updates and patches are applied to the `main` branch.

| Version | Supported |
| :--- | :--- |
| `main` | :white_check_mark: |
| < 0.1.0 | :x: |

---

## 2. Reporting a Vulnerability

If you discover a security vulnerability within LAN Observatory, please do **not** disclose it in public GitHub issues.

1. **GitHub Private Vulnerability Reporting**: Use the **Report a vulnerability** feature under the repository's **Security** tab.
2. **Direct Maintainer Contact**: If private vulnerability reporting is unavailable, reach out to the repository maintainer privately prior to any public disclosure.

Please include:
* A description of the issue and affected components.
* Steps or a minimal synthetic scenario to reproduce the behavior.
* Potential impact and any suggested remediations.

---

## 3. Authorization & Operational Boundaries

LAN Observatory is designed exclusively for authorized network observability.

### Authorization Prerequisite

* Operators must only execute LAN Observatory on networks and hardware they own or have explicit, documented authorization to assess.
* Unauthorized discovery or scanning across third-party, carrier, or public networks is strictly prohibited.

### Defensive Scope Invariants

LAN Observatory explicitly will not implement:

* **Credential Attacks**: No brute-force guessing, password spraying, or dictionary attacks.
* **Exploitation & Delivery**: No weaponized payloads, remote code execution primitives, or shell generation.
* **Defense Evasion**: No packet spoofing, firewall/IDS evasion flags, or intentional traffic disguise.
* **Persistence & Lateral Movement**: No software installation on discovered target nodes or pivoting mechanisms.
* **Service Disruption**: No aggressive fuzzing, buffer overflow probes, or SYN flood sweeps.

All probing logic is developed to gather non-invasive operational telemetry (ICMP latency, ARP cache entries, DNS resolution, and standard protocol metadata).

---

## 4. Network Safety & Fragile Host Protection

Local subnets often host sensitive or low-power devices, such as IoT sensors, smart home bridges, or industrial control systems. LAN Observatory enforces:

* **Bounded Concurrency**: Workers are strictly rate-limited to avoid saturating wireless spectrum or filling switch CAM tables.
* **Predictable Timeouts**: Every probe enforces deterministic timeout thresholds.
* **Graceful Termination**: Responds immediately to `SIGINT` / `SIGTERM` signals and cancels outstanding network requests via Go contexts.

---

## 5. Privacy & Data Handling

* **Zero Secrets**: Never commit `.env` files, SSH keys, or access credentials.
* **Sanitized Reporting**: When submitting bug reports, traces, or test fixtures, replace real public IP addresses and sensitive device identifiers with synthetic RFC 5737 ranges (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`).

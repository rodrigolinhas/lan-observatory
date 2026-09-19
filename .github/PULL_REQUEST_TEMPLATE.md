## What does this PR do?

<!-- Brief description of the changes introduced by this pull request. -->

Closes #
Parent Feature: #

## Milestone

<!-- Select the target milestone for this change -->
- [ ] M0 — Project Foundation
- [ ] M1 — Network Discovery
- [ ] M2 — Device Inventory
- [ ] M3 — Persistent Network History
- [ ] M4 — Continuous Observation
- [ ] M5 — Interactive TUI
- [ ] M6 — Local Network Intelligence
- [ ] M7 — Service Observation
- [ ] M8 — PCAP Analysis
- [ ] M9 — Defensive Detection
- [ ] M10 — Linux Distribution
- [ ] Other / Tooling

## Changes

- 
- 

## Verification & Testing

- [ ] Unit & race tests passed (`go test -v -race ./...`)
- [ ] Static analysis passed (`go vet ./...`)
- [ ] Formatting verified (`gofmt -l .`)
- [ ] Dependency vulnerabilities verified (`govulncheck ./...`)
- [ ] Local build verified (`go build -o bin/lanobs ./cmd/lanobs`)
- [ ] Network safety verified (bounded concurrency, context timeouts, no flood hazards)

## Checklist

- [ ] PR addresses a single focused implementation issue without unrelated changes.
- [ ] Implementation satisfies issue acceptance criteria.
- [ ] Network operations use explicit `context.Context`, timeouts, and cancellation.
- [ ] No offensive tooling, brute-force mechanics, or evasion techniques are introduced.
- [ ] No credentials, secrets, or private network topology data are committed.
- [ ] Documentation updated where applicable (`README.md`, `docs/`).
- [ ] Commit is cryptographically signed using SSH signing.

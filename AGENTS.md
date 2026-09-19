# AGENTS.md

Instructions, constraints, and architectural guidelines for AI coding assistants (Gemini, Claude, Codex, Cursor, etc.) working on **LAN Observatory** (`lanobs`).

---

## 1. Core Directives

1. **Defensive Network Observability Only**:
   - LAN Observatory is exclusively designed for defensive asset inventory and historical observation on networks owned or authorized by the operator.
   - **Never** implement offensive features: no credential brute-forcing, no exploit delivery, no vulnerability weaponization, no defense evasion flags, no packet spoofing, and no unauthorized persistence.

2. **Decoupled Architecture**:
   - The CLI (Cobra) and Terminal UI (Bubble Tea) are **presentation interfaces** over a reusable application core.
   - Network logic, probing primitives, and inventory records must **never** live inside TUI rendering views or CLI command definitions.
   - Keep business logic isolated in `internal/` packages.

3. **Standard Library First**:
   - Strongly prefer the Go standard library (`net`, `net/netip`, `context`, `sync`, `log/slog`) over third-party packages.
   - External networking libraries (such as `golang.org/x/net/icmp`) are used only when the standard library does not provide the required low-level protocol capabilities.

4. **Bounded Concurrency & Mandatory Timeouts**:
   - Network probing must enforce bounded concurrency using worker pools or buffered channels.
   - Never launch unbounded goroutines over an IP range.
   - Every network operation must accept a `context.Context` with explicit timeouts and cancellation handling.

5. **No Premature Dependencies & No Speculative Packages**:
   - Do **not** introduce PostgreSQL, Docker, Bubble Tea, PCAP parsers, or active service scanners before their designated roadmap milestone.
   - Do not create empty abstraction layers, dummy interfaces, or placeholder files. Build strictly what is required for the active vertical slice.

6. **Vertical Slice Development**:
   - Implement features as cohesive, verifiable vertical slices. The first vertical slice is:
     `CIDR → validation → bounded concurrent discovery → ICMP observations → normalized results → CLI table/JSON`.

7. **Zero Secrets & Sanitized Network Data**:
   - Never commit private keys, passwords, API tokens, or real enterprise network topologies.
   - Use synthetic RFC 5737 documentation ranges (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) in test fixtures, mocks, and examples.

8. **Rigorous Concurrency Testing**:
   - All concurrent code must be verified with the Go race detector (`go test -race ./...`).
   - Flaky tests, data races, or goroutine leaks are unacceptable.

---

## 2. Standard Verification Commands

Always run and verify these commands when modifying code:

```bash
# Verify formatting
gofmt -l .

# Run static analysis
go vet ./...

# Run test suite with race detector
go test -v -race ./...

# Check dependency vulnerabilities
govulncheck ./...

# Compile binary
go build -o bin/lanobs ./cmd/lanobs
```

---

## 3. Scope Rule

> **Do not implement unrelated features while solving a task.**  
> Keep every pull request, commit, and file change minimal, focused, and verified.

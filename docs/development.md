# Local Development Guide — LAN Observatory

This guide details the prerequisites, local environment configuration, verification workflows, and commit signing conventions for developing **LAN Observatory** (`lanobs`).

---

## 1. Prerequisites

Before contributing, verify your workstation has the following tools installed:

* **Go**: 1.27 or higher (`go version`)
* **Git**: 2.30 or higher (`git --version`)
* **govulncheck**: Go vulnerability scanner
  ```bash
  go install golang.org/x/vuln/cmd/govulncheck@latest
  ```

---

## 2. Initial Setup

### 1. Clone the Repository
```bash
git clone https://github.com/rodrigolinhas/lan-observatory.git
cd lan-observatory
```

### 2. Configure SSH Commit Signing
LAN Observatory requires all commits to be cryptographically signed using SSH keys. Set up your global Git configuration:

```bash
# Set SSH as the signing protocol
git config --global gpg.format ssh

# Point to your SSH public key
git config --global user.signingkey ~/.ssh/<your-public-key>.pub

# Enable automatic commit signing
git config --global commit.gpgsign true

# (Optional) Set up allowed signers file for local git log --show-signature
git config --global gpg.ssh.allowedSignersFile ~/.config/git/allowed_signers
```

> [!TIP]
> Ensure your public key is added to your GitHub profile under **Settings > SSH and GPG keys > New SSH Key** with **Key type: Signing Key** to receive the green **Verified** commit badge on GitHub.

---

## 3. Verification & Development Workflows

Run these commands regularly during development and before creating a pull request:

```bash
# 1. Verify module dependencies
go mod tidy
go mod verify

# 2. Verify code formatting conforms to standard Go formatting
gofmt -l .

# 3. Run static analysis
go vet ./...

# 4. Run automated test suite with race condition detection
go test -v -race ./...

# 5. Check for known dependency vulnerabilities
govulncheck ./...

# 6. Build local binary to bin/lanobs
go build -o bin/lanobs ./cmd/lanobs
```

---

## 4. Linux Network Permissions for ICMP

Operating systems restrict raw ICMP socket creation to prevent unauthorized packet crafting. For local testing on Linux, configure one of the following:

### Option A: Unprivileged Ping Sockets (Recommended)
Allow standard users to open unprivileged ICMP datagram sockets without requiring `root` or `sudo`:

```bash
sudo sysctl -w net.ipv4.ping_group_range="0 2147483647"
```
To make this persistent across reboots, add to `/etc/sysctl.d/99-ping.conf`:
```ini
net.ipv4.ping_group_range = 0 2147483647
```

### Option B: Set Capability on Compiled Binary
Grant the `CAP_NET_RAW` capability directly to the compiled executable:

```bash
go build -o bin/lanobs ./cmd/lanobs
sudo setcap cap_net_raw+ep ./bin/lanobs
```

---

## 5. Coding Standards & Idioms

* **Context Propagation**: Every network call, loop, and worker must take `ctx context.Context` as its first parameter and respect `ctx.Done()`.
* **Resource Cleanup**: Always ensure open sockets, listeners, and worker channels are closed cleanly using `defer`.
* **Structured Logging**: Use `log/slog` for internal telemetry. Use key-value attributes (`slog.String`, `slog.Duration`) instead of formatting text strings with `fmt.Sprintf`.
* **Error Wrapping**: Wrap errors with context using `%w` (`fmt.Errorf("discovery: failed to probe host %s: %w", ip, err)`).
* **Defensive Mindset**: Ensure inputs (CIDRs, IPs, timeouts) are sanitized and validated before initiating network traffic.

---

## 6. Troubleshooting Common Issues

### "socket: operation not permitted"
* Cause: Linux kernel requires privileges for raw socket creation.
* Solution: Run the `sysctl` command in [Section 4](#4-linux-network-permissions-for-icmp) or assign `CAP_NET_RAW` to `./bin/lanobs`.

### "DATA RACE" warning in tests
* Cause: Shared mutable state accessed across goroutines without proper synchronization.
* Solution: Inspect the race detector stack trace. Use channels, `sync.Mutex`, or `sync/atomic` to coordinate concurrent access.

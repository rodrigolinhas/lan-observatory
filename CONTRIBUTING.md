# Contributing to LAN Observatory

Thank you for your interest in contributing to **LAN Observatory** (`lanobs`)! We welcome contributions that align with our defensive engineering philosophy, high code quality standards, and strict architectural principles.

---

## 1. Core Principles

Before submitting code or proposing features, please ensure your work adheres to these core architectural rules:

1. **Defensive and Authorized Networks Only**: LAN Observatory is engineered exclusively for defensive visibility on networks owned by or explicitly authorized for the operator. Never implement offensive features, credential attacks, exploit delivery, or evasion mechanisms.
2. **Decoupled Architecture**: The command-line interface (CLI) and terminal user interface (TUI) are thin presentation layers over reusable application logic. Network probing and inventory logic must **never** live inside TUI rendering loops or CLI command definitions.
3. **Standard Library First**: Prefer the Go standard library (`net`, `net/netip`, `context`, `sync`, `log/slog`) wherever possible. Introduce external dependencies only when standard library facilities are strictly insufficient (e.g., `golang.org/x/net/icmp`).
4. **Bounded Concurrency & Deterministic Timeouts**: All network operations must enforce bounded concurrency pools (e.g., worker pools), explicit `context.Context` propagation, and deterministic timeouts. Unbounded goroutines and infinite loops are strictly prohibited.
5. **No Speculative Architecture**: Do not build empty abstraction layers, hypothetical plugin interfaces, or placeholder packages. Build strictly for the active vertical slice.
6. **Develop Through Small Vertical Slices**: Deliver complete end-to-end functionality incrementally (e.g. `CIDR → validation → bounded concurrent discovery → ICMP observations → normalized results → CLI table/JSON`).
7. **Zero Secrets or Real Network Topologies**: Never commit real internal network layouts, router passwords, private keys, or `.env` files. Use synthetic network ranges (RFC 5737: `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) in test fixtures and examples.

---

## 2. Local Setup & Verification

LAN Observatory targets **Go 1.27+**.

### Prerequisites

* Go 1.27+
* Git 2.30+
* `govulncheck` (`go install golang.org/x/vuln/cmd/govulncheck@latest`)

### Verification Commands

Before opening a pull request, run the complete verification suite locally:

```bash
# 1. Verify and tidy dependencies
go mod tidy
go mod verify

# 2. Check source formatting
gofmt -l .

# 3. Run static analysis
go vet ./...

# 4. Run tests with race detection
go test -v -race ./...

# 5. Check for known dependency vulnerabilities
govulncheck ./...

# 6. Build the binary
go build -o bin/lanobs ./cmd/lanobs
```

### Linux Network Permissions for ICMP

Low-level ICMP probing on Linux requires socket permissions. Choose one of the following configurations for local development:

```bash
# Option A (Recommended): Enable unprivileged ping sockets for local users
sudo sysctl -w net.ipv4.ping_group_range="0 2147483647"

# Option B: Set CAP_NET_RAW capability on the compiled binary
sudo setcap cap_net_raw+ep ./bin/lanobs
```

---

## 3. Branch Naming Conventions

All work must be performed on a focused branch created from `main`:

| Type | Branch Pattern | Example |
| :--- | :--- | :--- |
| Feature | `feat/<issue-number>-<description>` | `feat/12-icmp-worker-pool` |
| Bug Fix | `fix/<issue-number>-<description>` | `fix/19-cidr-parsing-overflow` |
| Documentation | `docs/<description>` | `docs/architecture-slice-diagram` |
| Testing | `test/<description>` | `test/add-netutil-concurrency-tests` |
| Refactoring | `refactor/<description>` | `refactor/normalize-device-state` |
| Maintenance / Chores | `chore/<description>` | `chore/update-ci-pipeline` |

---

## 4. Commit Message Conventions

We adhere to the [Conventional Commits](https://www.conventionalcommits.org/) standard:

```text
<type>(<optional scope>): <imperative summary>
```

### Allowed Types

* `feat`: A new user-facing capability, CLI flag, or discovery collector.
* `fix`: A bug fix or error handling correction.
* `docs`: Documentation updates or design guides.
* `test`: Adding or correcting tests.
* `refactor`: Code modifications that neither add a feature nor fix a bug.
* `chore`: Build tooling, dependency, or CI maintenance.

### Examples

* `feat(discovery): implement bounded concurrent ICMP prober`
* `fix(netutil): handle single IP edge cases in CIDR parsing`
* `docs(readme): document Linux raw socket permissions`
* `test(inventory): add race detection tests for device table`
* `chore(ci): enable govulncheck in GitHub Actions`

---

## 5. Signed Commits (SSH Signing)

To ensure authenticity and supply-chain integrity, all commits in LAN Observatory should be cryptographically signed. The project prefers **SSH signing**.

### Setup SSH Commit Signing

Configure Git globally to use your SSH key for signing:

```bash
# 1. Configure Git to use SSH as the signing format
git config --global gpg.format ssh

# 2. Specify your SSH public key path
git config --global user.signingkey ~/.ssh/<public-key>.pub

# 3. Enable automatic commit signing for all commits
git config --global commit.gpgsign true

# 4. (Optional) Configure local allowed signers for local git log verification
git config --global gpg.ssh.allowedSignersFile ~/.config/git/allowed_signers
```

### Important Distinctions & Best Practices

* **Automatic Signing**: When `commit.gpgsign=true` is set, executing `git commit` automatically cryptographically signs your commit with your SSH key.
* **Redundant `-S` Flag**: While `git commit -S` explicitly instructs Git to sign a commit, it is redundant when `commit.gpgsign=true` is enabled.
* **Do NOT Confuse with `-s` (Sign-off)**: The lowercase flag `git commit -s` appends a `Signed-off-by: Name <email>` metadata line (used for Developer Certificate of Origin). It does **not** cryptographically sign the commit and must not be treated as a substitute for SSH signing.
* **GitHub "Verified" Badge**: To display GitHub's green **Verified** badge, add your public key to your GitHub account under **Settings > SSH and GPG keys > New SSH Key**, selecting **Key type: Signing Key**.

> [!NOTE]
> Never commit private SSH keys (`id_ed25519`, `id_rsa`) or hardcode specific user paths into project scripts.

---

## 6. Pull Request Process

1. **One Issue, One PR**: Every PR must address one focused issue. Do not bundle unrelated refactorings or cosmetic cleanups with functional changes.
2. **Stay Linked**: Reference the parent issue in the PR description using `Closes #<issue-number>`.
3. **Small & Reviewable**: Keep PRs small (ideally under 300 lines changed). Large changes should be split into traceable implementation steps.
4. **Pass CI**: Ensure that formatting, vet, race tests, vulnerability checks, and builds pass cleanly.

---

## 7. Definition of Done (DoD)

A pull request is complete and eligible for merge only when all of the following conditions are satisfied:

- [ ] **Acceptance Criteria**: The implementation completely fulfills the requirements of the linked issue.
- [ ] **Tests Added & Passing**: Unit and/or integration tests cover the new logic.
- [ ] **Race Detector**: `go test -race ./...` passes with zero race conditions.
- [ ] **Static Analysis**: `go vet ./...` completes without warnings.
- [ ] **Formatting**: Code adheres strictly to standard Go formatting (`gofmt -l .` reports no files).
- [ ] **Build Verification**: `go build ./cmd/lanobs` compiles cleanly without warnings.
- [ ] **Network Safety & Timeouts**: Any network operation enforces bounded concurrency, context cancellation, and deterministic timeouts.
- [ ] **Defensive Scoping**: The change strictly adheres to defensive visibility boundaries (no exploit or evasion capabilities).
- [ ] **Documentation**: Corresponding documentation in `README.md` or `docs/` is updated.
- [ ] **No Secrets**: No private keys, credentials, or proprietary network layouts are present.
- [ ] **No Unrelated Scope**: Free of opportunistic refactoring or tangential dependency additions.
- [ ] **Cryptographically Signed**: Commits are signed with SSH signing.

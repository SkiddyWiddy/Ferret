# Stealth Recon Scanner — Roadmap

A stealth-and-speed-focused port scanner with vulnerability discovery at its core.
The thesis: **find what's exploitable, fast, without lighting up their logs.**

> ⚠️ Authorized testing only. Point this at your own machines, a lab VM, or explicitly
> authorized targets like `scanme.nmap.org`. Never scan hosts you don't own or have
> written permission to test.

---

## The three pillars (and the tension between them)

- **Stealth / opsec** — SYN scans, randomized timing/order, low footprint.
- **Speed** — concurrent probing (currently a thread pool; asyncio is a higher-ceiling alternative).
- **Vuln finding** — banner/version grab → match against known CVEs.

Stealth and speed pull in opposite directions. The *interesting* engineering is
letting the user choose the tradeoff (`--stealth` vs `--fast`). Designing that
toggle is your novelty — lean into it.

---

## Phase 1 — Basic TCP connect scan (the boring-but-working baseline)

Build the un-stealthy version first. It's easier to debug and gives you a
correctness baseline to compare the stealthy version against.

- [ ] Parse args: target host, port range, mode. *(still TODO — target is currently hardcoded)*
- [x] Loop over ports, attempt a full TCP connect, record open/closed. *(`scan_port` in scanner.py)*
- [x] Print results cleanly. *(basic `print(open_ports)` for now — upgrade in Phase 6)*

**Milestone:** ✅ correctly finds open ports on `scanme.nmap.org` (22, 80 should show).
**Why this order:** if the SYN scan later disagrees with this, you know where the bug is.

---

## Phase 2 — Concurrent speed  ✅ (done via thread pool)

Went with a **thread pool** (`concurrent.futures.ThreadPoolExecutor`) instead of
asyncio — port scanning is I/O-bound, so threads release the GIL while waiting on the
network and give real parallelism, with much simpler code. asyncio remains a valid
higher-ceiling option if you ever need tens of thousands of concurrent probes.

- [x] Run many probes concurrently. *(`base_scan` fans `scan_port` across a thread pool)*
- [x] Cap concurrency so you don't exhaust sockets. *(`workers=200` on the pool)*
- [x] Add a per-connection timeout. *(`s.settimeout(timeout)`)*
- [ ] Measure: time a 1000-port scan sequential vs threaded. Put the numbers in the README.

**Milestone:** ✅ full port range scanned in seconds, not minutes.

---

## Phase 3 — Stealth mode (SYN scan)

This is the heart of the "opsec" pillar. Requires raw sockets → `scapy` + admin/root.

- [ ] Send a lone SYN packet; read the response.
  - SYN-ACK → port open (then send RST so you never complete the handshake).
  - RST → port closed.
  - no reply → filtered.
- [ ] Randomize port order and source port.
- [ ] Add a timing knob: delay + jitter between probes to dodge rate-based IDS.
- [ ] Wire up the `--stealth` vs `--fast` toggle so the two modes visibly trade off.

**Milestone:** SYN scan agrees with the Phase-1 connect scan, but the target's
application logs stay quieter (test against your own logged VM to confirm).

**Honesty check:** this implements the *core* evasion primitives. Don't claim
"undetectable" — modern IDS/EDR is an arms race. "Implements evasion primitives
and explains the tradeoffs" is the accurate, impressive framing.

---

## Phase 4 — Banner grab + version detection

- [ ] For each open port, read the service banner (e.g. `SSH-2.0-OpenSSH_7.4`).
- [ ] Parse out product + version.

**Milestone:** open ports come back labeled with a best-guess service and version.

---

## Phase 5 — Vulnerability lookup

- [ ] Take the product+version and query a CVE source (NVD API is free; there are
      also local databases if you'd rather not hit the network mid-scan).
- [ ] Print matching CVE IDs + severity + a link. You are *reporting* known CVEs,
      not writing exploits.
- [ ] Cache lookups so repeat scans don't re-hit the API.

**Milestone:** `OpenSSH 7.4` open → tool lists associated CVEs with links.

---

## Phase 6 — Polish

- [ ] A clean live TUI with `rich` — ports filling in, color-coded, final summary.
- [ ] A README with: the three-pillar pitch, a demo GIF, setup steps, the
      stealth/speed tradeoff explained, and a responsible-use disclaimer.
- [ ] A "Future ideas" section (decoys, fragmentation, OS fingerprinting, diff mode)
      so reviewers see you know what's beyond the MVP without you having to build it.
- [ ] Tidy commit history + a tagged v0.1 release.

---

## Stretch goals (only after v0.1 ships)

- Diff mode: compare two scans, highlight what changed.
- Decoy / fragmented packets for deeper evasion.
- OS fingerprinting from TTL / window size.
- Export to JSON/HTML report.

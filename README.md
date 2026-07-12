# 🦡 Ferret

A stealth- and speed-focused port scanner with vulnerability discovery at its core.

**Thesis:** *find what's exploitable, fast, without lighting up their logs.*

> ⚠️ **Authorized testing only.** Only scan machines you own or have explicit
> written permission to test (e.g. `scanme.nmap.org`). Unauthorized port scanning
> is illegal in many jurisdictions.

---

## Why Ferret?

Most scanners optimize for one thing. Ferret is built around three goals that
deliberately pull against each other — and lets you choose the tradeoff:

- **Stealth / opsec** — half-open (SYN) scans, randomized timing and probe order,
  minimal footprint on the target's logs.
- **Speed** — concurrent probing so thousands of timeouts overlap instead of stacking.
- **Vulnerability discovery** — grab service banners, detect versions, and report
  known CVEs (reporting only — Ferret does not exploit).

The interesting engineering is the `--stealth` vs `--fast` toggle: the same scan,
tuned for either quietness or speed.

## Status

🚧 Early / in progress. Current state:

- [x] Basic TCP connect scan
- [x] Concurrent scanning (thread pool)
- [ ] Stealth SYN scan + timing evasion
- [ ] Banner grabbing + version detection
- [ ] CVE lookup
- [ ] Polished CLI + live output

See [`ROADMAP.md`](ROADMAP.md) for the full plan.

## Usage

```bash
python scanner.py
```

*(CLI arguments are on the roadmap — for now, configure the target in the script.)*

## Design notes

Ferret currently uses a **thread pool** for concurrency. Threads are the right call
here: port scanning is I/O-bound, so threads release the GIL while waiting on the
network and give real parallelism. `asyncio` would scale to higher concurrency, but
threads are more readable and plenty fast at this scale — a deliberate readability
tradeoff.

## Disclaimer

This tool is for educational purposes and authorized security testing only. The
author is not responsible for misuse. Scan only systems you own or are explicitly
permitted to test.

# bzminer v100.02b3 — public beta

A GPU/CPU cryptocurrency miner with a live web dashboard, per-GPU overclocking,
and thermal/power safety limits.

Mines **Ergo**, **Pearl**, **VERUS**, **Warthog** and **XELIS**, and ships
**sha256d** as the open-source SDK example.

This is a **beta**. It has been validated against real pools and real hardware, but
it is new — watch a rig before leaving it unattended.

## Downloads

| file | size | SHA-256 |
|---|---|---|
| `bzminer_v100.02b3_windows.zip` | 7.4 MB | `bf9e06f071597c0db9a7557dee9549fada8d8ca8cf18de444ddb3d2e6e956d7e` |
| `bzminer_v100.02b3_linux.tar.gz` | 8.5 MB | `d16776d8967d820d2e3640e6dfda5b9ddae6f4f4403457518ecb82b7b8c867d1` |
| `bzminer_v100.02b3_macos.tar.gz` | 7.9 MB | `4c5e0a9c0c8abd829a8dd477b863f1a4a2b1cef991fdd82cd4eeb2f1473d78a7` |

Verify before running:

```bash
# Linux / macOS
sha256sum -c <<< "<hash>  <file>"

# Windows (PowerShell)
Get-FileHash .\<file> -Algorithm SHA256
```

A checksum only proves the file is the one that was published — always download
from the official release page.

## Getting started

1. Unpack the archive for your platform.
2. Open a `start_<algo>` script and replace the placeholder wallet with your own.
3. Run it, then open <http://127.0.0.1:4014/> for the dashboard.

`readme.txt` inside each archive has the full quick start, the common options, and
the hardware-safety settings. `config.txt` is a commented copy of *every* setting,
generated from the shipped binary, so it always matches the build you have.

## Algorithms in this build

| algorithm | coin | dev fee | notes |
|---|---|---|---|
| `ergo` | Ergo | 1% | Autolykos v2. GPU only, and memory-hard: it builds a table of N 32-byte elements (about 2 GB) in VRAM, so a card needs the room for it. N grows with block height, and the table is rebuilt when it changes. Wallet addresses start with '9'. |
| `pearl` | Pearl | 2% | a zk proof-of-work: each share is a STARK proof, so it is far heavier per hash than a normal algorithm. Also mines SOLO - point -p at your node's RPC URL instead of a pool. Wallet addresses start with 'prl1'. |
| `sha256d` | — | none | The open-source SDK example: a full algorithm plugin (CPU+GPU, pool stratum, bench job source). Copy plugins/public/sha256d as a template for your own. |
| `verus` | VERUS | 1% | VerusHash v2.2 CPU mining. A Verus R-address, optionally followed by .worker. |
| `warthog` | Warthog | 2% | janushash - the GPU filters sha256t and the CPU runs verushash, on the same nonces. Needs BOTH a GPU and the CPU; one without the other finds nothing. Wallet addresses are 48 hex characters. |
| `xelis` | XELIS | 1% | xelhash. Wallet addresses start with 'xel:'. |

## Which download

| download | what it is |
|---|---|
| `bzminer_<ver>_<os>` | **Full.** All 16 plugins compiled in. One self-contained binary - take this one unless you know you want the other. |
| `bzminer_<ver>_<os>_lite` | **Lite.** No plugins compiled in; loads them from a `plugins/` folder beside it. Useless on its own - it needs the archive below. |
| `bzminer_<ver>_<os>_plugins` | The drop-in plugins. Required by lite; a full user can also drop a newer one here to override the built-in copy. |

The plugin CONTENT of full and lite is the same - the difference is delivery, not capability. Lite exists so a single plugin can be replaced without shipping a new miner.


Dev fee is read from the shipped binary itself (`bzminer --list-algos`), so this
table is what you are actually running — not a number someone remembered to update.

## Protecting your hardware

Off by default; set what you want:

```
--set safety.max_temp_c=85     # pause a GPU over 85C, resume when it cools
--set safety.max_power_w=600   # ...or over 600W
--set safety.sustain_s=15      # only after 15s over the limit, not on a spike
```

Only the offending card stops — the rest of the rig keeps mining. Temperature is
the one to set first: it works on every card, and hardware in trouble gets hot.

## Reporting problems

Please include: your OS, GPU model(s) and driver version, the algorithm and pool,
and the first ~40 lines of the log (it prints the build, the devices it found, and
what each backend reported). `--list-metrics` output helps for sensor issues.

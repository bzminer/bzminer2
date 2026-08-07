# bzminer v100.01b1 — public beta

A GPU/CPU cryptocurrency miner with a live web dashboard, per-GPU overclocking,
and thermal/power safety limits.

Mines **Pearl**, **Warthog** and **XELIS**, and ships **sha256d** as the
open-source SDK example.

This is a **beta**. It has been validated against real pools and real hardware, but
it is new — watch a rig before leaving it unattended.

## Downloads

| file | size | SHA-256 |
|---|---|---|
| `bzminer_v100.01b1_windows.zip` | 5.8 MB | `7fd47ba4a62ca218c3bfd0e81df91cdcb0318ceda520ac469da3e0b2876d497c` |
| `bzminer_v100.01b1_windows_lite.zip` | 2.5 MB | `ffd92db0b6b122d57e4c885b3cbb2a2ec6cdfeb93e7bb695efc80592e247754c` |
| `bzminer_v100.01b1_windows_plugins.zip` | 12.2 MB | `54d107b9c6c0ab2b8ddee8de35b1cf4abf73cc99cf7e0ea1bd380c45d0e15ed6` |
| `bzminer_v100.01b1_linux.tar.gz` | 6.6 MB | `2d204f7dc44a9f647eae0c3d139fd7f2c1ee09272e130765eb0d0c65a61c8c89` |
| `bzminer_v100.01b1_linux_lite.tar.gz` | 3.2 MB | `4ccf5254f862fdabe409f7053c8c29097742e05a8d9e582df1d9bdf02c668ec8` |
| `bzminer_v100.01b1_linux_plugins.tar.gz` | 15.3 MB | `47bc21c57c46aed0723af5a087a620d55409aeba36fb349f0f50e070a176b716` |
| `bzminer_v100.01b1_macos.tar.gz` | 6.3 MB | `f4de15ce203e60bcd38300b0cbade4b290d78c3ceb611b5208c38ea1a6ed6dcb` |
| `bzminer_v100.01b1_macos_lite.tar.gz` | 3.1 MB | `0e55c70257ea85cc4808bc3e8424515a74e045b1f2e7f2fe557166aec22067ed` |
| `bzminer_v100.01b1_macos_plugins.tar.gz` | 13.9 MB | `da974f5025ceb6630a15dca161e35e69b21beb30f70e1b95cc9ee52a30a866ea` |

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
| `pearl` | Pearl | 2% | a zk proof-of-work: each share is a STARK proof, so it is far heavier per hash than a normal algorithm. Also mines SOLO - point -o at your node's RPC URL instead of a pool. Wallet addresses start with 'prl1'. |
| `sha256d` | — | none | The open-source SDK example: a full algorithm plugin (CPU+GPU, pool stratum, bench job source). Copy plugins/public/sha256d as a template for your own. |
| `warthog` | Warthog | 2% | janushash - the GPU filters sha256t and the CPU runs verushash, on the same nonces. Needs BOTH a GPU and the CPU; one without the other finds nothing. Wallet addresses are 48 hex characters. |
| `xelis` | XELIS | 1% | xelhash. Wallet addresses start with 'xel:'. |

## Which download

| download | what it is |
|---|---|
| `bzminer_<ver>_<os>` | **Full.** All 14 plugins compiled in. One self-contained binary - take this one unless you know you want the other. |
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

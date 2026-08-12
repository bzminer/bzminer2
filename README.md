# bzminer

A GPU and CPU cryptocurrency miner for Windows, Linux and macOS.

Mines **Pearl**, **VERUS**, **Warthog** and **XELIS**, and ships **sha256d** as
the open-source SDK example.

One binary, no installer. It writes nothing outside its own folder except the log
file and the config you point it at.

> **Public beta.** It mines, and it has been run against real pools on real
> hardware — but it is new. Watch a rig before you leave it alone with it, and
> please report what you find.

## Download

Take the archive for your platform from the [latest release][releases], unpack it,
and check the SHA-256 published beside it.

[releases]: ../../releases

```bash
tar -xzf bzminer_<version>_linux.tar.gz && cd bzminer_<version>_linux
```

Three archives per platform:

| download | what it is |
|---|---|
| `bzminer_<ver>_<os>` | **Full.** All 15 plugins compiled in. One self-contained binary - take this one unless you know you want the other. |
| `bzminer_<ver>_<os>_lite` | **Lite.** No plugins compiled in; loads them from a `plugins/` folder beside it. Useless on its own - it needs the archive below. |
| `bzminer_<ver>_<os>_plugins` | The drop-in plugins. Required by lite; a full user can also drop a newer one here to override the built-in copy. |

The plugin CONTENT of full and lite is the same - the difference is delivery, not capability. Lite exists so a single plugin can be replaced without shipping a new miner.


## Start mining

Every archive has a launcher per algorithm. Put your wallet in one and run it, or
go straight to the command line:

```bash
./bzminer -a xelis -o stratum+ssl://ussw.vipor.net:5177 -w xel:YOUR_WALLET --worker rig1
```

Then open **<http://127.0.0.1:4014/>** for the dashboard.

macOS blocks unsigned downloads outright — if it refuses to open, clear the
quarantine flag once, in the unpacked folder:

```bash
xattr -dr com.apple.quarantine .
```

### Coming from bzminer 1.x

Old start scripts run as they are. `-p` is the **pool URL**, as it always was
(the password is `--pass`), and the underscore spellings of the tuning flags —
`--oc_power_limit`, `--oc_fan_speed`, `--cpu_threads`, `--warthog_verus_hr_target`
— are all accepted and mapped to their modern equivalents.

Two 1.x options describe machinery this version no longer has:
`--warthog_max_ram_gb` and `--warthog_target_cpu_pressure`. The GPU→CPU hand-off
is now a small cache-local ring rather than a RAM buffer you size, and the balance
between the two stages is found by a tuner rather than aimed at a pressure number.
bzminer accepts both options and says what governs that behaviour now, instead of
quietly mapping them onto something that merely looks similar.

Anything it does not recognise is reported rather than ignored, so a flag that has
been renamed shows up as a warning at startup instead of silently doing nothing.

## Algorithms

| algorithm | coin | dev fee | notes |
|---|---|---|---|
| `pearl` | Pearl | 2% | a zk proof-of-work: each share is a STARK proof, so it is far heavier per hash than a normal algorithm. Also mines SOLO - point -o at your node's RPC URL instead of a pool. Wallet addresses start with 'prl1'. |
| `sha256d` | — | none | The open-source SDK example: a full algorithm plugin (CPU+GPU, pool stratum, bench job source). Copy plugins/public/sha256d as a template for your own. |
| `verus` | VERUS | 1% | VerusHash v2.2 CPU mining. A Verus R-address, optionally followed by .worker. |
| `warthog` | Warthog | 2% | janushash - the GPU filters sha256t and the CPU runs verushash, on the same nonces. Needs BOTH a GPU and the CPU; one without the other finds nothing. Wallet addresses are 48 hex characters. |
| `xelis` | XELIS | 1% | xelhash. Wallet addresses start with 'xel:'. |

The dev fee is declared by each algorithm, printed in the log at startup, and
listed here straight from the shipped binary — so what you read is what you run.

### Mining each one

**XELIS** — CPU and GPU, both at once on the same rig.

```bash
./bzminer -a xelis -o stratum+ssl://us.vipor.net:5177 -w xel:YOUR_WALLET --worker rig1
```

Pools: `stratum+ssl://us.vipor.net:5177` ·
`stratum+ssl://us.xelis.herominers.com:1225` · solo to your own node with
`ws://127.0.0.1:8080`.

**Pearl** — a zk proof-of-work: every share is a STARK proof, so it is far heavier
per hash than an ordinary algorithm and the hashrate numbers look small by
comparison. Normal.

```bash
./bzminer -a pearl -o stratum+tcp://us.pearl.herominers.com:1200 -w prl1YOUR_WALLET --worker rig1
```

Pools: `stratum+tcp://us.pearl.herominers.com:1200` ·
`stratum+tcp://prl-us.kryptex.network:7048` ·
`stratum+tcp://pearl-cpu-eu1.luckypool.io:3370`.

Pearl also mines **solo** — point `-o` at your own node's RPC URL instead of a
pool.

**Warthog** — needs a GPU *and* the CPU together. The GPU filters sha256t and the
CPU runs VerusHash over the same nonces; neither half finds anything alone, so
disabling either one stops the rig. The first two minutes show `calibrating
verus` while it works out how hard to drive the filter — a near-zero rate during
that is expected, not a fault.

```bash
./bzminer -a warthog -o stratum+ssl://us.vipor.net:5120 -w YOUR_48_HEX_WALLET --worker rig1
```

Pools: `stratum+ssl://us.vipor.net:5120` · `stratum+tcp://warthogunited.com:2001` ·
`stratum+tcp://pool.us.woolypooly.com:3140` · `stratum+tcp://us.acc-pool.pw:12000`.
Solo against your own node, either way it exposes work — `http://127.0.0.1:3000`
for RPC, or `stratum+tcp://127.0.0.1:3456` if you started the node with
`--stratum 0.0.0.0:3456`.

Warthog's CPU stage needs hardware AES and carry-less multiply: any x86-64 from
2013 on, or Apple Silicon. On anything else the miner says so and refuses rather
than pretending.

**SHA-256d** is the open-source example that ships with the plugin SDK. It mines,
but SHA-256d is ASIC territory — it is there to be read and copied, not to earn.

## Testing without a pool

Two ways to make the miner work without pointing it at anyone, both useful for
different questions. Neither needs a wallet, an internet connection or an account.

**`--bench`** — mine a job made up locally. No sockets at all. This answers *"does
this machine mine this algorithm"*: it exercises the kernels and the CPU path and
nothing else.

```bash
./bzminer --bench -a pearl        # also: xelis, warthog, verus, sha256d
```

**`--benchmark [difficulty]`** — bzminer starts a **real stratum server inside
itself**, on a local port, and then connects to it over TCP like any other pool.
The whole client path runs: subscribe, authorize, `mining.notify`, share submit,
the pool's verdict, the a/r/s counters and the pool-hashrate column. It is the
honest way to measure a card, because a share only counts once the server has
accepted it.

```bash
./bzminer --benchmark -a warthog              # default difficulty 1000
./bzminer --benchmark 25000000000000 -a pearl # fixed difficulty 25T
```

Pick the difficulty deliberately. Too low and a fast rig floods the local server
with submissions and measures the plumbing instead of the kernel; too high and you
will wait a long time for enough shares to mean anything. As a rule aim for a
share every few seconds: Pearl on a big GPU wants `25000000000000` (25T), xelis
wants something in the thousands.

The share count is what to trust. `shares × difficulty ÷ seconds` is a rate the
miner cannot flatter — compare it against the displayed hashrate, and if the two
disagree for long, one of them is lying.

Both modes honour everything else: `--nvidia` to test one vendor, `--oc-*` to
measure at a given power limit, `-O log` for a scrolling log instead of the table.

## Pools, backups and failover

List as many pools as you like. The **first is primary and the rest are backups**:
bzminer connects to the primary, and on a disconnect rotates through the others
until one answers, then keeps mining there.

```bash
./bzminer -a xelis \
  -o stratum+ssl://main-pool:5177   -w xel:YOUR_WALLET --worker rig1 \
  -o stratum+ssl://backup-pool:5177 -w xel:YOUR_WALLET --worker rig1
```

Each `-o` starts a new pool, and the flags after it belong to that pool. Anything
you put *before* the first `-o` applies to all of them, so a single `-a xelis`
covers the whole list.

The same thing in `config.txt`:

```json
{
  "pools": [
    { "url": "stratum+ssl://main-pool:5177",   "wallet": "xel:YOUR_WALLET", "worker": "rig1", "algo": "xelis" },
    { "url": "stratum+ssl://backup-pool:5177", "wallet": "xel:YOUR_WALLET", "worker": "rig1", "algo": "xelis" }
  ]
}
```

Backups must be the **same algorithm** as the primary. `"pool": 0` picks one entry
out of the list; `"pool": [0, 2]` runs a chosen subset; omit it and you get all of
them.

Other pool options: `stratum+ssl://` for TLS (add `--ssl-verify` to check the
certificate), `-p` for a password, and `--proxy socks5://host:port` to route every
pool connection through a SOCKS5 proxy.

## The console

Three screens. Press **`o`** to cycle, or start on one with `--output <name>`.

| screen | what it shows | when |
|---|---|---|
| `tui` | boxed dashboard — devices, shares, pool state, log pane. In `--dmon` it drops the mining boxes and shows a sensors table instead | **the default whenever there is a terminal**, mining or monitoring |
| `monitor` | the same sensors, `nvidia-smi dmon` style: one dense table, no boxes | `--dmon`, when you want it plain |
| `log` | plain scrolling log | always; the default when output is redirected to a file |

If you pipe or redirect bzminer's output, it stays on `log` on purpose — a
full-screen dashboard sent to a file is just a file full of cursor codes. That is
what mining OSes get.

**Keys, on every screen:**

| key | |
|---|---|
| `o` | next screen |
| `+` / `-` | more / less log detail, live |
| `c` | open the command line |
| `q` | quit |

### Commands

Press `c`, type, press enter.

| command | what it does |
|---|---|
| `help` | list the commands |
| `pause` / `resume` | stop and restart mining without exiting |
| `oc ...` | overclock — see below |
| `intensity <n>` | mining intensity; `0` is auto |
| `level <name>` | log level: `trace`, `network`, `debug`, `info`, `warn`, `error` |
| `output <name>` | switch screen by name |
| `quit` | exit |

`level network` prints the raw stratum traffic, which is the fastest way to settle
an argument with a pool about what was actually sent.

## Overclocking

NVIDIA cards. Three routes — a start script, the console, or the web dashboard —
all going through **one implementation**, so they cannot disagree with each other.
Every refusal tells you *why* ("needs root/administrator", "not supported by this
card") instead of failing quietly.

### From the command line (or `config.txt`)

```
--oc-power-limit 160             board power cap, watts
--oc-core-clock-offset 150       core offset, MHz
--oc-memory-clock-offset 1000    memory offset, MHz
--oc-lock-core-clock 1600        pin the core clock; 0 unlocks
--oc-lock-memory-clock 810       pin the memory clock; 0 unlocks
--oc-reset                       back to driver defaults
```

Each takes one value for every card, or one per card: `--oc-power-limit 160,180`.
The same settings live under `oc` in `config.txt`.

**Fan control** takes a fixed duty, or a temperature target with a range to move
inside:

```
--oc-fan-speed 60                        just run at 60%
--oc-fan-speed "t:60[25-75]"             hold the CORE at 60C, using 25-75% fan
--oc-fan-speed "tm:80[50-100]"           the same, aimed at MEMORY temperature
--oc-fan-speed "t:60[25-75] tm:80[50-100]"   both — whichever wants more fan wins
```

That last form is the one worth knowing: a card can sit at a perfectly comfortable
core temperature while its memory cooks, and the second clause is what catches it.
The fan moves gradually toward the target rather than jumping, and stays inside
the range you gave.

A card keeps a fan duty or a locked clock until something clears it, so a run that
is **killed** rather than closed leaves them behind. `bzminer --oc-reset` on its
own undoes them.

Coming from bzminer 1.x? The underscore spellings (`--oc_power_limit`,
`--oc_fan_speed`, …) all still work, so an old start script needs no editing.

### Live, from the console

Press `c`, then:

```
oc                                  what is there, and what each card is set to
oc 150 500                          +150 core, +500 memory on every card
oc 0 core=+150 mem=+500             card 0 only
oc all fan=70                       every card
oc 1 power=300 clock=2600           card 1: 300 W limit, 2600 MHz locked core
oc 0 fan=auto                       hand the fan back to the driver
oc reset [dev|all]                  undo it
```

The settings are `core`, `mem`, `power`, `clock` and `fan`. `core` and `mem` are
**offsets** in MHz; `power` is watts; `clock` locks the core to an absolute MHz;
`fan` is a duty percentage or `auto`. A device is named by its number as shown in
the table, or `all`.

`fan=auto` is not `fan=0`. Zero means *stop the fan*.

On Linux, clock, power and fan changes need root. bzminer restores whatever it
changed when it exits.

## Safety limits

Off by default. Turn on what you want and bzminer will park a card that misbehaves
and bring it back when it recovers — **only the offending card stops, the rest of
the rig keeps mining**.

```bash
--set safety.max_temp_c=85      # pause a GPU over 85 C
--set safety.max_power_w=600    # ...or over 600 W board power
--set safety.sustain_s=15       # only after 15s over the limit, not on a spike
--set safety.resume_margin=5    # resume once it is 5 back under
```

Set the temperature limit first. It works on every card, and hardware in trouble —
a failing 12VHPWR connector included — gets hot.

There are 12VHPWR current limits too (`safety.max_current_a`, and per-pin
`safety.max_pin_current_a` / `safety.max_pin_imbalance_a`), but they need sensors
only some boards carry. `config.txt` says exactly which.

## The web dashboard

Served by the `webui` plugin at **<http://127.0.0.1:4014/>**, and it is the same
data the console shows: per-device hashrate, shares, pool state, every sensor, and
the overclock controls.

```bash
--set http_port=8080            # different port
--set http_address=0.0.0.0      # reachable from the rest of the LAN
--set http_enabled=false        # off
```

It binds to localhost by default. `0.0.0.0` exposes it to anything that can reach
the machine — there is no password on it, so put it behind something you trust.

JSON, for scripting or your own dashboard:

| endpoint | |
|---|---|
| `/api/snapshot` | everything: devices, pools, hashrate, shares |
| `/api/stream` | the same, pushed as it changes |
| `/api/gpus`, `/api/metrics`, `/api/topology`, `/api/ram`, `/api/storage` | hardware detail |
| `/api/oc` | read and apply overclocks |
| `/status` | the shape other mining dashboards expect |
| `/hive_status` | HiveOS |

Removing `webui` from the `plugins` folder is how you switch the HTTP server off
entirely — no plugin, no listener.

## Monitoring without mining

```bash
./bzminer --dmon
```

Telemetry and the web dashboard, no mining. Useful for watching a rig that is
busy with something else, or for checking sensors before you commit a card to a
job.

`--metrics` picks the columns, on both monitoring screens (`tui` and `monitor`):

```bash
./bzminer --dmon --metrics temps,powers,clocks
./bzminer --metrics ?            # every group and live metric on this machine
```

A group expands to every sensor of that kind the machine actually reports, so
`temps` on a card with per-die sensors gives you all of them without naming one.
Columns nothing reports are dropped rather than filled with dashes.

Groups are `temps`, `clocks`, `powers`, `volts`, `currents`, `pcie`,
`utilization`, `memory`, `timings`, `counters`, `performance`, `all`.

One-shot inspection, no mining, no dashboard:

```bash
./bzminer --gpu-info       # NVIDIA devices as CUDA sees them
./bzminer --cpu-info       # CPU topology and live metrics
./bzminer --ram-info       # memory modules and SMBus temperatures
./bzminer --list-metrics   # every sensor this machine exposes
```

## Rigs with several GPUs — or several CPUs

Every device mines by default — every GPU **and** the CPU. Two ways to narrow it.

**By type**, on the command line:

```
--nvidia --amd --intel --cpu
```

Naming any of them makes those the *only* ones that mine: `--nvidia` mines the
NVIDIA cards and nothing else, `--nvidia --cpu` adds the CPU back. Give a value
instead to change one type and leave the rest alone — `--amd 0` stops AMD mining
without touching anything else. The same lives under `device_types` in
`config.txt`.

**By single device**, for when four identical cards are fine and the fifth is
doing something else. Devices are addressed by the number in the table
(`0`, `1`, ...) — which is also what the `oc` command takes:

```json
{
  "devices": [
    { "index": 0, "enabled": true,  "intensity": 0 },
    { "index": 1, "enabled": false },
    { "index": 2, "enabled": true,  "intensity": 20 }
  ]
}
```

`intensity: 0` is auto, which is usually right.

Multi-socket machines are detected properly: bzminer reads the real topology —
packages, cores, threads, NUMA nodes and cache layout — and prints it at startup.
CPU mining threads are placed with that layout in mind rather than scattered, so a
dual-socket box does not spend its time moving work between NUMA nodes.

Warthog goes further and plans the whole rig at once, because its two halves feed
each other: it groups CPU workers by shared L3 cache and measures each GPU's PCIe
bandwidth so it knows an x16 slot from an x1 riser, then sizes the candidate flow
per card. `--no-bandwidth-test` skips that measurement if you would rather save
the second per GPU at startup.

## Configuration

Three ways to say the same thing. The command line always wins:

```bash
./bzminer -a xelis -o stratum+ssl://host:5177 -w <wallet>   # pool flags
./bzminer --set http_port=8080                              # any setting, by path
./bzminer --config myrig.txt                                # a config file
```

A `config.txt` next to the binary is picked up automatically, and the log says
which file it read. `bzminer --config-doc` prints a fully commented template of
every setting — generated from the binary itself, so it can never describe an
option your build does not have. `--save-config` writes the resolved settings,
after all three layers, to `config.effective.json`.

## Every command-line option

Printed by the binary itself, so this list cannot describe a flag your build does
not have. Same text as `bzminer --help`.

```
bzminer - GPU/CPU cryptocurrency miner

Usage: bzminer [options]

Run modes:
  --dmon                  Device-monitor mode: local metrics + web dashboard only (no mining)
  --cpu-info              Print CPU topology + live metrics, then exit
  --gpu-info              Print NVIDIA GPU (CUDA) devices, then exit
  --ram-info              Print memory modules + SMBus temp diagnostic, then exit
  --list-metrics          List every available metric (backends + per-device, incl. vendor), then exit
  --list-algos            Print this build's algorithms + dev fees as JSON, then exit
  --list-plugins          Print this build's plugins (static + dynamic) as JSON, then exit
  --build-info            Print version + whether release secrets were set; exit 3 if not
  --pawnio-dump           Dump the RyzenSMU PM-table (Zen4 per-CCD offset capture), then exit
  --print-config          Print the resolved settings as JSON, then exit
  --config-doc            Print a commented config.txt template, then exit
  --bench, --test         Mine a local bench job (default sha256d; select with -a)
  --benchmark [diff]      Run a local stratum server and mine it over real TCP
                          (default algo xelis, difficulty 1000; e.g. --benchmark -a xelis)
  --no-watchdog           Run the worker in-process (no supervisor; debugging)
  -h, --help              Show this help, then exit
  -V, --version           Print version, then exit

Display:
  -O, --output <name>     Console front-end: log (scrolling) | tui | monitor | a plugin
  -I, --interval <ms>     Metric/table update cadence (default 1000; log/tui/monitor/web)
  --metrics <list>        Columns/groups to show (comma/space list; the active output defines
                          the names). Groups expand to all matching sensors, e.g. 'temps',
                          'pcie', 'clocks', 'powers', 'volts', 'currents',
                          'utilization', 'counters', 'performance', 'memory', 'timings', 'all'.
                          '--metrics ?' lists every group and live metric.
  --color / --no-color    Force ANSI color on/off (default: auto-detect a terminal)

Devices:
  --nvidia / --amd / --intel / --cpu [0|1]
                          Which device TYPES may mine. With no value the flag is an
                          ALLOWLIST: naming any type means only the named types mine
                          (`--nvidia --cpu` = NVIDIA and the CPU, nothing else). With a
                          value it sets just that type (`--amd 0` turns AMD off and
                          leaves the rest alone). No flags at all = mine everything
  --bandwidth-test [arg]  Measure each GPU's host<->device PCIe bandwidth at startup and
                          publish it (pcie.h2d / pcie.d2h). on | off | <MiB buffer size>;
                          default on, 64 MiB. Algorithms that stream results off the GPU
                          (e.g. warthog) use it to tell an x16 slot from a x1 riser
  --no-bandwidth-test     Skip it (saves ~1s per GPU at startup)
                          To turn off ONE device rather than a whole type, set
                          devices[].enabled = false in config.txt (index counts every
                          device enumerated, so disabling one does not renumber the rest)

Overclocking (NVIDIA; on Linux every knob is a privileged NVML write):
  --oc-power-limit <W>            Board power cap
  --oc-core-clock-offset <MHz>    Persistent VF-curve offset for the core
  --oc-memory-clock-offset <MHz>  Same for memory
  --oc-lock-core-clock <MHz>      Pin the core clock instead of letting boost wander
                                  (0 = unlock)
  --oc-lock-memory-clock <MHz>    Pin the memory clock (0 = unlock)
  --oc-fan-speed <spec>           Either a fixed duty ('60') or a temperature curve:
                                    t:<target>[<min>-<max>]   keep the CORE at <target> C,
                                                              fan within <min>..<max> %
                                    tm:<target>[<min>-<max>]  the same for MEMORY temp
                                  Both may be given - whichever wants more fan wins:
                                    --oc-fan-speed "t:60[25-75] tm:80[50-100]"
  --oc-reset                      Put every card back to driver defaults FIRST (offsets
                                  cleared, clocks unlocked, power default, fan automatic).
                                  Use alone to undo settings a killed run left behind
  Every value may be one number for all cards, or a per-device list ('160,180').
  bzminer 1.x's --oc_* spelling (underscores) is accepted for all of these.

Logging:
  --log-level <level>     trace | network | debug | info (default) | warn | error
                          `network` shows the exact pool/stratum wire messages
  -v, -vv, -v2 ...        More verbose (each step toward trace; -v2 == -vv)
  -q, -qq, -q2 ...        Quieter (each step toward error)
  --log-wire-max <n>      In `network` logs, elide any single field longer than n chars
                          (default 512) - keeps huge opaque values like a Pearl share's
                          ~140KB proof from drowning the frame. 0 = print raw frames

Configuration:
  --config <path>         Config file to load (default: config.txt)
  --set <path>=<value>    Override any setting below (e.g. --set http_port=8080)
  --<path> <value>        Same as --set, as a flag (dashes map to '_')
  --save-config           Write the resolved config to config.effective.json
  -o, --url  <url>        Add a pool (starts a new pool entry)
  -p <url>                Same as -o - bzminer 1.x's spelling of the pool URL
  -w, --wallet <addr>     Payout wallet for the current pool
  --worker <name>         Worker/rig name (login sent to the pool: wallet.worker)
  -u, --user <user>       LEGACY combined login (use --wallet/--worker instead)
  --pass <pass>           Password for the current pool (NOTE: -p is the pool URL)
  -a, --algo <algo>       Algorithm for the current pool
  --ssl-verify            Verify the current pool's TLS cert (stratum+ssl; default off)
  --proxy <url>           Route all pools through a SOCKS5 proxy: socks5://[user:pass@]host:port
  --plugin-update <mode>  Auto-download/update plugins: on | off | auto (lite=on, full=off)
  --plugin-manifest <url> Override the plugin manifest URL (beta channel / testing)
  --pool <spec>           Active pool(s) from the config: an index, or [0,2]; [] = monitor
  --cu-kernel [algo=]<f>  Override a plugin's embedded CUDA kernel (.cubin/.ptx/.cu)
  --cl-kernel [algo=]<f>  Override a plugin's embedded OpenCL kernel (.spv/.cl/.bin)
                          (for debugging or third-party source plugins)

Settings (set in config.txt, or with --set <path>=<value>):
  log.level                  Log verbosity: trace, network, debug, info, warn, or error (network = raw pool traffic)
  log.file                   Also write logs to this file (empty = console only)
  network.timeout_ms         Pool / network socket timeout, in milliseconds
  network.reconnect_ms       Delay before reconnecting after a disconnect, in milliseconds
  network.proxy              Route all pool connections through a SOCKS5 proxy: socks5://[user:pass@]host:port (empty = direct). CLI: --proxy
  plugins.update             Auto-download/update plugins from the manifest: auto|on|off (auto = on for the lite build, off for the full build). CLI: --plugin-update
  plugins.manifest_url       Override the plugin manifest URL (empty = built-in GitHub default + bzminer.com fallback). CLI: --plugin-manifest
  safety.max_temp_c          Pause a GPU (mining stops on that card only, auto-resumes when it cools) when its core or hotspot temp exceeds this, in C. 0 = off
  safety.max_power_w         Pause a GPU when its board power exceeds this, in W. 0 = off
  safety.max_current_a       Pause a GPU when its 12VHPWR CONNECTOR current (aggregate of all pins, not per-pin) exceeds this, in A. NVIDIA Blackwell on Windows only. 0 = off
  safety.max_pin_current_a   Pause a GPU when its WORST SINGLE 12VHPWR pin exceeds this, in A - the connector failure mode the aggregate cannot see. Needs per-pin shunts: ASUS ROG Astral RTX 5080/5090 on Windows only. 0 = off
  safety.max_pin_imbalance_a Pause a GPU when the difference between its highest- and lowest-current 12VHPWR pins exceeds this, in A. ASUS ROG Astral RTX 5080/5090 on Windows only. GPUProbe's sample alarm uses 5.5 A. 0 = off
  safety.sustain_s           How long a card must stay over a limit before it is paused, in seconds (rejects transient spikes)
  safety.resume_margin       Resume only once the value drops to (limit - this), in the tripped limit's unit (C/W/A) - hysteresis so it does not flap
  safety.resume_s            And stayed under the resume point this long, in seconds
  dmon                       Device-monitor mode: telemetry + web UI only, no mining (CLI: --dmon)
  http_enabled               Serve the monitoring web UI + JSON API (needs the webui plugin; no plugin = no HTTP server)
  http_address               Web server bind address ("0.0.0.0" allows LAN access)
  http_port                  Web server TCP port
  save_effective             On startup, write the resolved config to config.effective.json
  pool                       Active pool(s) from pools[]: an index (0 or "0"), an array ([0, 2] = multiple pools), or [] for monitoring mode. Omit = all pools (first primary, rest failover). CLI: --pool
  bandwidth_test             Measure each GPU's host<->device PCIe bandwidth once at startup and publish it (pcie.h2d / pcie.d2h). Costs ~1s per GPU; algorithms that stream results off the GPU use it as a hard ceiling. CLI: --bandwidth-test
  bandwidth_test_mb          Per-copy buffer size for that measurement, in MiB. Large enough to measure throughput rather than launch latency
  pools[].url                Pool URL (CLI: -o / --url; stratum+ssl:// for TLS)
  pools[].wallet             Payout wallet address (CLI: -w / --wallet)
  pools[].worker             Worker / rig name; sent to the pool as wallet.worker (CLI: --worker)
  pools[].pass               Password (CLI: --pass; note -p is the POOL URL, as in bzminer 1.x)
  pools[].algo               Algorithm (CLI: -a / --algo)
  pools[].ssl_verify         stratum+ssl: verify the pool's TLS certificate chain + hostname (CLI: --ssl-verify)
  devices[].index            Device index - counts EVERY device enumerated, so disabling one does not renumber the others
  devices[].intensity        Mining intensity (0 = auto)
  devices[].enabled          Whether to mine on this device. To turn off a whole vendor or the CPU instead, use device_types below
  device_types.nvidia        Mine on NVIDIA devices (CLI: --nvidia)
  device_types.amd           Mine on AMD devices (CLI: --amd)
  device_types.intel         Mine on Intel devices (CLI: --intel)
  device_types.cpu           Mine on the CPU (CLI: --cpu)
  oc.power_limit             Board power cap in watts. One value for every card, or a per-device list ("160,180"). NVIDIA only (CLI: --oc-power-limit)
  oc.core_clock_offset       Core VF-curve offset in MHz (CLI: --oc-core-clock-offset)
  oc.memory_clock_offset     Memory VF-curve offset in MHz (CLI: --oc-memory-clock-offset)
  oc.lock_core_clock         Pin the core clock in MHz instead of letting boost wander; 0 = unlock (CLI: --oc-lock-core-clock)
  oc.lock_memory_clock       Pin the memory clock in MHz; 0 = unlock (CLI: --oc-lock-memory-clock)
  oc.fan_speed               Fixed duty ("60") or a temperature curve: t:<target>[<min>-<max>] for core temp, tm:... for memory, e.g. "t:60[25-75] tm:80[50-100]" (CLI: --oc-fan-speed)
  oc.reset                   Put every card back to driver defaults before applying anything above (CLI: --oc-reset)

Examples:
  bzminer -o stratum+tcp://pool:3333 -w <wallet> --worker rig --pass x -a sha256d
  bzminer -a warthog -w <wallet>.rig -p stratum+tcp://pool:3001 --nvidia
          --oc-power-limit 160 --oc-fan-speed "t:60[25-75] tm:80[50-100]"
  bzminer --pool 1                (mine only pools[1] from config.txt)
  bzminer --dmon                  (device-monitor mode: metrics + web UI, no mining)
  bzminer --oc-reset              (undo overclock settings a killed run left behind)
  bzminer --set http_address=0.0.0.0 --set http_port=8080

Environment:
  Some tuning and diagnostic settings have no flag - e.g. BZ_CPU_THREADS and
  BZ_ZK_THREADS cap worker counts. See docs/env-vars.txt.
```

## Every setting

`config.txt` is JSON, and every field is optional — what follows is the complete
set with its defaults, straight out of `bzminer --config-doc`. Any of them can
also be set on the command line with `--set <path>=<value>`.

The shipped `config.txt` is this file, so you can edit it in place. It starts in
**monitoring mode** (`"pool": []`) — sensors and the dashboard, no mining — so a
fresh install does not start hashing to a placeholder wallet. Put your wallet in
`pools[]` and set `"pool": 0` to mine.

```jsonc
// bzminer configuration (config.txt), JSON. Every field is optional; the
// values below are the defaults. Override any of them on the command line
// with:  --set <path>=<value>
{
  "log": {
    // Log verbosity: trace, network, debug, info, warn, or error (network = raw pool traffic)
    "level": "info",
    // Also write logs to this file (empty = console only)
    "file": ""
  },
  "network": {
    // Pool / network socket timeout, in milliseconds
    "timeout_ms": 30000,
    // Delay before reconnecting after a disconnect, in milliseconds
    "reconnect_ms": 5000,
    // Route all pool connections through a SOCKS5 proxy: socks5://[user:pass@]host:port (empty = direct). CLI: --proxy
    "proxy": ""
  },
  "plugins": {
    // Auto-download/update plugins from the manifest: auto|on|off (auto = on for the lite build, off for the full build). CLI: --plugin-update
    "update": "auto",
    // Override the plugin manifest URL (empty = built-in GitHub default + bzminer.com fallback). CLI: --plugin-manifest
    "manifest_url": ""
  },
  "safety": {
    // Pause a GPU (mining stops on that card only, auto-resumes when it cools) when its core or hotspot temp exceeds this, in C. 0 = off
    "max_temp_c": 0,
    // Pause a GPU when its board power exceeds this, in W. 0 = off
    "max_power_w": 0,
    // Pause a GPU when its 12VHPWR CONNECTOR current (aggregate of all pins, not per-pin) exceeds this, in A. NVIDIA Blackwell on Windows only. 0 = off
    "max_current_a": 0,
    // Pause a GPU when its WORST SINGLE 12VHPWR pin exceeds this, in A - the connector failure mode the aggregate cannot see. Needs per-pin shunts: ASUS ROG Astral RTX 5080/5090 on Windows only. 0 = off
    "max_pin_current_a": 0,
    // Pause a GPU when the difference between its highest- and lowest-current 12VHPWR pins exceeds this, in A. ASUS ROG Astral RTX 5080/5090 on Windows only. GPUProbe's sample alarm uses 5.5 A. 0 = off
    "max_pin_imbalance_a": 0,
    // How long a card must stay over a limit before it is paused, in seconds (rejects transient spikes)
    "sustain_s": 15,
    // Resume only once the value drops to (limit - this), in the tripped limit's unit (C/W/A) - hysteresis so it does not flap
    "resume_margin": 5,
    // And stayed under the resume point this long, in seconds
    "resume_s": 5
  },
  // Device-monitor mode: telemetry + web UI only, no mining (CLI: --dmon)
  "dmon": false,
  // Serve the monitoring web UI + JSON API (needs the webui plugin; no plugin = no HTTP server)
  "http_enabled": true,
  // Web server bind address ("0.0.0.0" allows LAN access)
  "http_address": "127.0.0.1",
  // Web server TCP port
  "http_port": 4014,
  // On startup, write the resolved config to config.effective.json
  "save_effective": false,
  // Active pool(s) from pools[]: an index (0 or "0"), an array ([0, 2] = multiple pools), or [] for monitoring mode. Omit = all pools (first primary, rest failover). CLI: --pool
  "pool": [],
  // Measure each GPU's host<->device PCIe bandwidth once at startup and publish it (pcie.h2d / pcie.d2h). Costs ~1s per GPU; algorithms that stream results off the GPU use it as a hard ceiling. CLI: --bandwidth-test
  "bandwidth_test": true,
  // Per-copy buffer size for that measurement, in MiB. Large enough to measure throughput rather than launch latency
  "bandwidth_test_mb": 64,
  "pools": [
    {
      // Pool URL (CLI: -o / --url; stratum+ssl:// for TLS)
      "url": "stratum+tcp://us.pearl.herominers.com:1200",
      // Payout wallet address (CLI: -w / --wallet)
      "wallet": "<your wallet address>",
      // Worker / rig name; sent to the pool as wallet.worker (CLI: --worker)
      "worker": "rig",
      // Password (CLI: --pass; note -p is the POOL URL, as in bzminer 1.x)
      "pass": "x",
      // Algorithm (CLI: -a / --algo)
      "algo": "pearl",
      // stratum+ssl: verify the pool's TLS certificate chain + hostname (CLI: --ssl-verify)
      "ssl_verify": false
    }
  ],
  "devices": [
    {
      // Device index - counts EVERY device enumerated, so disabling one does not renumber the others
      "index": 0,
      // Mining intensity (0 = auto)
      "intensity": 0,
      // Whether to mine on this device. To turn off a whole vendor or the CPU instead, use device_types below
      "enabled": true
    }
  ],
  "device_types": {
    // Mine on NVIDIA devices (CLI: --nvidia)
    "nvidia": true,
    // Mine on AMD devices (CLI: --amd)
    "amd": true,
    // Mine on Intel devices (CLI: --intel)
    "intel": true,
    // Mine on the CPU (CLI: --cpu)
    "cpu": true
  },
  "oc": {
    // Board power cap in watts. One value for every card, or a per-device list ("160,180"). NVIDIA only (CLI: --oc-power-limit)
    "power_limit": "",
    // Core VF-curve offset in MHz (CLI: --oc-core-clock-offset)
    "core_clock_offset": "",
    // Memory VF-curve offset in MHz (CLI: --oc-memory-clock-offset)
    "memory_clock_offset": "",
    // Pin the core clock in MHz instead of letting boost wander; 0 = unlock (CLI: --oc-lock-core-clock)
    "lock_core_clock": "",
    // Pin the memory clock in MHz; 0 = unlock (CLI: --oc-lock-memory-clock)
    "lock_memory_clock": "",
    // Fixed duty ("60") or a temperature curve: t:<target>[<min>-<max>] for core temp, tm:... for memory, e.g. "t:60[25-75] tm:80[50-100]" (CLI: --oc-fan-speed)
    "fan_speed": "",
    // Put every card back to driver defaults before applying anything above (CLI: --oc-reset)
    "reset": false
  }
}
```

## Environment variables

A few knobs are deliberately environment-only: they are tuning and diagnostic
settings rather than configuration, and giving each a flag would bury the options
that matter. These are the ones worth knowing.

| variable | what it does |
|---|---|
| `BZ_CPU_THREADS=<n>` | Cap CPU mining workers. Default is every logical processor — on Windows that means all processor groups, not just the 64 in the one the process started in |
| `BZ_XELIS_THREADS=<n>` | The same, for xelis specifically |
| `BZ_ZK_THREADS=<n>` | Worker cap for Pearl's ZK prover |
| `BZ_NO_CPU=1`, `BZ_NO_CUDA=1`, `BZ_NO_OPENCL=1` | Skip a whole compute backend. Blunter than `--nvidia`/`--amd`/`--cpu`, and useful for isolating one device while benchmarking |
| `BZ_WARTHOG_SHAVERUS=<H/s>` | Pin the candidate rate each Warthog GPU is asked for instead of letting the tuner find it. This is the modern equivalent of bzminer 1.x's `--warthog_verus_hr_target`, which maps onto it automatically |
| `BZ_WARTHOG_SHA_QUALITY=<pct>` | Nudge Warthog's balance point without pinning it |
| `BZ_WARTHOG_AFFINITY=auto\|group\|none\|pu` | How Warthog's verus workers are pinned to cores |
| `BZ_WARTHOG_INCLUDE_INTEGRATED=1` | Mine on an integrated GPU that shares system memory beside a discrete card. Off by default because on a Ryzen APU it took nearly half the CPU's verus rate to contribute almost no sha |
| `BZ_PEARL_HIP=0\|opencl\|force` | Pearl's AMD path: `0`/`opencl` force portable OpenCL, `force` fails rather than silently falling back |
| `BZ_ZK_GPU=1` | Run Pearl's ZK commitment on the GPU. Off by default: it wins on some machines and loses on others, because holding a CUDA context costs host CPU elsewhere |
| `BZ_POOL_PROBE_S=<n>` | Seconds between probes of a failed primary pool (minimum 5) |

The full list, including the diagnostic ones, is in `docs/env-vars.txt` in the
source repository.

## Plugins

Algorithms, hardware monitors, the console screens and the web dashboard are all
plugins behind one C ABI. This build contains:

- `mon_amd` — AMD GPU sensors, via ADL and sysfs.
- `mon_cpu` — CPU sensors: per-core temperature, frequency, package power.
- `mon_intel` — Intel GPU sensors, via IGCL and sysfs.
- `mon_nvidia` — NVIDIA sensors and overclocking, via NVML and NvAPI.
- `mon_pawnio` — the PawnIO driver path for AMD Ryzen SMU tables.
- `mon_priv_cpu` — CPU sensors that need elevated access (MSRs).
- `out_log` — the log file and its rotation.
- `out_monitor` — pushes stats to an external monitoring endpoint.
- `out_tui` — the interactive console - live table, hotkeys, and the 'c' command menu.
- `pearl` — a zk proof-of-work: each share is a STARK proof, so it is far heavier per hash than a normal algorithm. Also mines SOLO - point -o at your node's RPC URL instead of a pool.
- `sha256d` — The open-source SDK example: a full algorithm plugin (CPU+GPU, pool stratum, bench job source). Copy plugins/public/sha256d as a template for your own.
- `verus` — VerusHash v2.2 CPU mining.
- `warthog` — janushash - the GPU filters sha256t and the CPU runs verushash, on the same nonces. Needs BOTH a GPU and the CPU; one without the other finds nothing.
- `webui` — the web dashboard on http_port: live hashrate, shares, every hardware sensor, and overclocking. Also serves /status and /hive_status for mining OSes.
- `xelis` — xelhash.

Drop a newer `.dll`/`.so` into the `plugins` folder and it takes precedence over
the built-in copy — that is how a single algorithm gets updated without a new
miner. `bzminer --list-plugins` prints what a binary actually loaded.

## If something goes wrong

The first forty lines of the log name the build, the config file, the CPU, every
device found and the pool being tried. Start there, then:

| | |
|---|---|
| no GPUs found | update the GPU driver; `--gpu-info` shows what bzminer can see |
| pool rejects shares | almost always the wallet — check it is for the right coin, and that the placeholder in the sample script was replaced |
| overclock does nothing | on Linux these are privileged; run as root. The miner reports which knob the driver refused |
| dashboard not reachable | it listens on `127.0.0.1` only unless you set `http_address` |
| warthog finds nothing | it needs both a GPU and the CPU; check neither is disabled |

More detail: `--log-level debug`, or `--log-level network` for the raw pool
conversation.

**Reporting a bug** — please include your OS, GPU models and driver version, the
algorithm and pool, and the first ~40 lines of the log. For sensor or overclock
problems, `--list-metrics` and `--gpu-info` output help.

## Dev fee

Some algorithms mine to the developer for a small share of the time. Each declares
its own, it is in the table above, and it is printed in the log at startup — so
you can see it before committing a rig, not discover it later.

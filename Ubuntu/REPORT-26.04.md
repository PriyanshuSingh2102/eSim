# eSim on Ubuntu 26.04: installer testing report

Author: Priyanshu Singh
Fork: https://github.com/PriyanshuSingh2102/eSim (branch `installers`, commit `941a8ff6`)

## 1. Environment
- Ubuntu 26.04.1 LTS (resolute) under WSL
- Upstream `master` at `d86c1891` (eSim 2.6), checked out as a git worktree
- IHP Open PDK skipped on purpose (optional, analog IC design only)
- Proxy: no

## 2. What I built
A per-version installer, `Ubuntu/install-eSim-scripts/install-eSim-26.04.sh`, added to the
`installers` branch together with a `"26.04"` case in `install-eSim.sh`. Changes against the 25.04 script:
- KiCad from the distro (no PPA, no version pin)
- KiCad 9 config: sym-lib-table copied to `~/.config/kicad/9.0` and stale `Legacy` entries removed
- `~/Desktop` created before copying the launcher
- NGHDL fixes applied as patches after unzip (`patches/fix1-version-case.patch`, `patches/fix2-nghdl-script.patch`)

While doing this I found that `install-eSim.sh` at HEAD was damaged by an earlier merge
(`52d77f6d`, installer code pasted into the dispatcher, failing `bash -n`). I restored it
from `f4cede1a` and added only the 26.04 case.

Known cosmetic issue: with `kicadppa=""` the script prints "KiCad PPA is already present in sources".

## 3. What upstream master already does
Upstream `master` has since replaced the per-version scripts with one unified installer
(`Ubuntu/install-eSim.sh`, June 2026) that already supports 26.04:
- KiCad from universe, requiring version 9 or newer
- `mkdir -p "$HOME/Desktop"` before copying the launcher
- sym-lib-table seeded into the KiCad config folder that exists
- `nghdl/install-nghdl.sh` has a 26.04 case and `GHDL-BACKEND-26.04.md`

So my per-version script is largely superseded. I did not open a PR against `installers`.

## 4. Testing
All tests ran the unified `master` installer, not my per-version script.

| Run | Result |
|---|---|
| `--dry-run` | Profile "Ubuntu 26.04, KiCad 9+ via universe", 11 steps listed |
| `--install` run 1 | Succeeded in 26m45s, all toolchain checks passed |
| `--install` run 2 (after removing stale Verilator) | Succeeded in 23m28s, all checks passed |

Result: ngspice 45, KiCad 9.0.8, GHDL 4.1.0, Icarus Verilog 14.0, eSim symbol libraries in
`/usr/share/kicad/symbols`. The `Legacy` entries in sym-lib-table point to existing `.lib`
files in `~/.esim/kicad_static/legacy/`.

GUI / NgVeri example: NOT TESTED (edit this line if you run it).

## 5. Finding: stale /usr/local Verilator
- Run 1 reported Verilator 4.210 at `/usr/local/bin/verilator`, although apt had just installed 5.032.
  The NgVeri runtime objects were compiled from `/usr/local/share/verilator/include` (4.210 tree).
  The self-check still printed "All checks passed".
- The stale files were dated Nov 21, the day before the install, so they came from my earlier
  runs on this machine, not from the unified installer.
- After moving them aside, run 2 used `/usr/bin/verilator` 5.032 and built from `/usr/share/verilator/include`.
- Possible upstream improvement: `cleanLegacyEsim` removes an old GHDL from `/usr/local` but not an old
  Verilator, and the self-check does not enforce Verilator 5 or warn about shadowing.
- Not shown: whether NgVeri actually fails with 4.210.

## 6. Limits
- `install-eSim-26.04.sh` was only syntax-checked (bash -n), never executed.
- Tested on WSL only, not on a clean 26.04 VM; earlier runs on the same machine affected run 1.
- IHP PDK not installed.

## 7. Files
Logs: dry-run-26.04.log, install-26.04.log, install-26.04-run2.log, eSim-install.log.

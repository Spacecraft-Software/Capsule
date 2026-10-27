<!--
SPDX-FileCopyrightText: 2026 Mohamed Hammad <Mohamed.Hammad@SpacecraftSoftware.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->
<!-- GFM Document
     title:      Capsule — Compliance
     author:     Mohamed Hammad & Spacecraft Software
     date:       2026-10-27
     license:    CC-BY-SA-4.0
     project:    Capsule
-->

# Capsule — Compliance

Capsule declares assurance **Category B** under The Steelbore Standard §19: a compositor crash
ends every program in the session and loses unsaved work. This file is the §19.5 tailoring
register. A clause absent from the register is applied in full; every entry is re-read at the
G3 release gate.


## Tailoring register

| Clause | Status | Justification | Date |
|--------|--------|---------------|------|
| CLI Standard rules 4 and 7 (`schema` and `describe` sub-commands, noun-verb tree) | tailored | The first positional argument is always the program to run (Cage-compatible shape), so a sub-command would shadow a program of the same name. Introspection is offered as `--describe` and `--schema` with the same output (CPS-SRS-019, 020); there is no noun-verb tree because there is one command | 2026-10-27 |
| §24.2 toolchain pin in `rust-toolchain.toml` | tailored | The release toolchain is pinned by `flake.lock`, as in Escape Pod; a rustup-downloaded toolchain does not run on NixOS without an FHS loader, which would break the maintainer's builds and the §21.5 validation environment. The MSRV is `rust-version` in `Cargo.toml`, checked in CI | 2026-10-27 |
| §13 design system | not-applicable | Capsule draws no widgets: its only visible output is the client's surfaces on a canvas-colour background (CPS-SRS-048) | 2026-10-27 |
| §10.1–§10.3 CUA and Vim bindings, remappability | not-applicable | Capsule binds no key except Ctrl+Alt+F1–F12 under DRM (CPS-SRS-077); §10.4 (reserved assistive-technology chords) is applied in full | 2026-10-27 |
| §18.3 GUI accessibility (accessible names and roles, screen-reader verification) | not-applicable | Capsule has no user interface of its own to expose. The program publishes its own accessibility tree, and Capsule passes the session and accessibility buses through untouched (CPS-SRS-034). §18.1 accessible mode applies to Capsule's diagnostics (CPS-SRS-078) | 2026-10-27 |


## Gate records

### G1 — Requirements

**Characteristics gate, first pass: 2026-10-27** (T-001). §20.4 run over CPS-SRS-001–105 and
CPS-NEED-001–010, each requirement and then the set, with the §20.5 wording rules. Reviewer:
the authoring session (Claude Code), not independent. Result: **13 findings, gate not passed.**
Fixes land in their own PR (T-002); new obligations take new identifiers from CPS-SRS-106,
because identifiers are never reused.

| # | Requirement | Characteristic | Finding | Resolution |
|---|-------------|----------------|---------|------------|
| G1-01 | CPS-SRS-036, 037 | Correct | "First toplevel fullscreen, later ones centred" breaks any program that maps a splash or launcher window first: its main window would be centred at its own size. CPS-NEED-001 wants the program's window on the whole screen | Every toplevel **without a parent** is fullscreen; a toplevel **with a parent** (a dialog) is centred |
| G1-02 | CPS-SRS-037 | Singular | Placement and stacking order in one sentence | Stacking moves to a new requirement, CPS-SRS-109 |
| G1-03 | CPS-SRS-100, 101 | Correct, Singular | The X11 kiosk mode repeats G1-01 and G1-02 (first window vs. later windows) | Windows without `WM_TRANSIENT_FOR` fill the root window; transient windows are centred; stacking moves to CPS-SRS-112 |
| G1-04 | CPS-SRS-013 | Unambiguous | "No introspection flag" is not defined, and leaves open whether `--version`, `--help`, or `--dry-run` need a program | Name the flags: `--version`, `--help`, `--describe`, `--schema`, `--dry-run` |
| G1-05 | CPS-SRS-018 | Verifiable | "The effect that section defines" has no observable meaning for flags Capsule has no use for (`--print0`, `--yes`, `--absolute-time`) | Require acceptance without a usage error; the flags whose effect matters get their own requirements (G1-06, G1-12) |
| G1-06 | set | Complete | Nothing covers `--quiet`, or `--quiet` together with `--verbose` | New CPS-SRS-106 (both together exit 2) and CPS-SRS-107 (`--quiet` writes only errors) |
| G1-07 | CPS-SRS-021 | Feasible | Under the DRM backend the renderer cannot be known without opening the GPU, which needs the seat session that `--dry-run` must not open | Report backend, GPU, and output, all readable from udev and sysfs; drop the renderer |
| G1-08 | CPS-SRS-029 | Consistent | `--backend x11` is mandatory while the X11 kiosk mode it selects is optional | Drop `x11` from the mandatory value list; new CPS-SRS-108: a backend not built into the binary exits 2 with a hint |
| G1-09 | CPS-SRS-034 | Consistent | "No other variable added" contradicts CPS-SRS-095, which adds `DISPLAY` under `--xwayland` | Except CPS-SRS-095 by name |
| G1-10 | CPS-SRS-033 | Complete | A runtime directory that exists but where the socket cannot be created is not covered | Extend the condition to a failed socket creation |
| G1-11 | CPS-SRS-044, 057, 059 | Complete | No fallback when `XCURSOR_THEME`/`XCURSOR_SIZE` are unset; "internal panel" is undefined; no rule when no mode is marked preferred | Default theme `default` at 24 px; define *internal panel* (eDP, LVDS, DSI) in Terminology; first listed mode when none is preferred |
| G1-12 | CPS-SRS-022, 078 | Complete | `--version --json` lacks the CLI Standard's `maintainer` and `website` keys; `--no-color` and `--color never` are missing from the plain-output conditions | New CPS-SRS-111 for the JSON keys; add both flags to CPS-SRS-078 |
| G1-13 | CPS-NEED-005 | Able to be validated | No requirement traces to "no desktop's configuration leaks in" | New CPS-SRS-110: Capsule reads no configuration file |

Checked and kept: CPS-SRS-071 now names Capsule as the subject of a hand-back it can only
enable (the kernel and seat manager complete it), which is acceptable because the evidence is a
test of Capsule's own exit paths; reworded in T-002 so the sentence reads that way. The budget
figures (CPS-SRS-079–083) stay provisional and are confirmed at G2, not G1.

**Carried to G2:** whether Smithay's software renderer can import dmabufs. If it cannot,
`zwp_linux_dmabuf_v1` cannot be advertised under the CPS-SRS-061 fallback, and CPS-SRS-035
needs a conditional form.

**Before baselining:** a second pass by a reader that has not seen this session, prompted
from the requirement text alone (§21.4), recorded here.

---

*— Built by [Spacecraft Software](https://SpacecraftSoftware.org/) —*

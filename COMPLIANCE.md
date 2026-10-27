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

**Fixes: 2026-10-27** (T-002). All 13 findings resolved as the table states. New
requirements CPS-SRS-106–112 (112 optional); terms *parent*, *dialog*, *internal panel*,
*introspection flag*, and *configuration directory* added to Terminology. The set now holds
112 requirements, 99 of them required.

**Carried to G2:** whether Smithay's software renderer can import dmabufs. If it cannot,
`zwp_linux_dmabuf_v1` cannot be advertised under the CPS-SRS-061 fallback, and CPS-SRS-035
needs a conditional form.

**Characteristics gate, second pass: 2026-10-27** (§21.4, "a different reader decides").
Reviewer: a fresh Claude Code agent with no access to this session, prompted from the
requirement text and Standard §20–§21.1 alone, told not to read `PRD.md`, `PLAN.md`, `TODO.md`,
this file, or git history. It disclosed that the harness also loaded `AGENTS.md` and `CLAUDE.md`
into its context, and that it did not rely on them. Result: **18 findings plus one note; the
gate is not passed.** Triage by the author. The reviewer's three technical claims that change
the design were checked before acceptance (R-04, R-07, R-11).

| # | Requirement | Characteristic | Finding | Triage |
|---|-------------|----------------|---------|--------|
| R-01 | 010, 032–053, 060–072, 102 | Unambiguous, Feasible | Requirements carry no backend scope, so Wayland-socket obligations bind the X11 kiosk mode, where they are infeasible; 071 is meaningless when nested; 102 lacks its mode condition | Accept: a normative applicability sentence per section, explicit conditions where a section mixes modes |
| R-02 | 029, 098; NEED-010 | Consistent | `--backend` values exclude `x11` though 098 uses it; `auto` never picks it; NEED-010 says "the same command" | Accept: `x11` back in 029's values (108 covers a build without it); NEED-010 reworded to the same session through `xinit` |
| R-03 | 071 | Feasible, Correct | After `SIGKILL`, Capsule cannot act; the seat manager restores the VT | Accept: split into Capsule restoring the VT on every path it can catch, and a `will` statement about the seat manager, evidenced by a system test |
| R-04 | 057, 068, 069 | Correct, Complete | A closed lid usually leaves eDP connected, so "internal panel first" keeps the session on a dark panel beside a live external monitor | Accept (checked): lid state from the libinput switch device enters output choice |
| R-05 | 056–058, 068 | Unambiguous | Outputs on which GPU, which connector identifier, two internal panels, which output after removal; `boot_vga` is a sysfs attribute | Accept: outputs on the chosen GPU only; order by connector name; 068 reapplies 057/058 |
| R-06 | 079–082 | Feasible, Verifiable | DRM budgets cannot run on CI runners; 081 names no measurement points | Accept: measured on the reference machine as a release-gate procedure; 081 names its two points |
| R-07 | NEED-006, 035, 094–105 | Consistent, Correct | X11 clients can capture and inject through `GetImage` and XTEST, so NEED-006 cannot hold under the X modes; 035 omits `xwayland_shell_v1` | Accept (checked): NEED-006 scoped to Wayland clients and the X limit stated; 035 admits `xwayland_shell_v1` under `--xwayland` |
| R-08 | 034 | Correct | Under the nested backend the child inherits the host's `DISPLAY`, so an X11 fallback reaches the host desktop | Accept: under the nested backend Capsule removes `DISPLAY` unless `--xwayland` sets its own |
| R-09 | set | Complete | Missing: nested host unreachable, seat never active, GPU lost, seat manager gone, Capsule fatal while the child runs, X display unreachable, `--xwayland` on a build without it | Accept: one requirement each, with an exit status |
| R-10 | NEED-003, 077 | Able to be validated | A hung child leaves no path back to the greeter except an external signal | Accept: NEED-003 reworded to "reach another virtual terminal, from which the session can be ended"; no emergency chord, which would contradict 077 |
| R-11 | 110, 041 | Feasible | libxkbcommon's default include path reads `$XDG_CONFIG_HOME/xkb`, `~/.xkb`, and `/etc/xkb` | Accept (checked): 110 states it covers every file the process opens; the design disables xkbcommon's default includes |
| R-12 | 036, 074 | Complete | Output scale and transform unspecified, so "logical size" has two readings on HiDPI | Accept: scale 1 and normal transform in v0.1 |
| R-13 | 037, 040, 042, 044 | Complete | Dialog larger than the output; popup that fits no adjustment; "default keymap" undefined; cursor theme missing; no surface under the cursor | Accept: clamp to output; `evdev`/`pc105`/`us`; built-in cursor image; default cursor over the background |
| R-14 | 019, 021–023 | Complete | No exit status; no JSON shape; GPU and output meaning under nested | Accept: exit 0; shapes in the `Interfaces` node; nested reports the host window |
| R-15 | 009, 089, 092 | Unambiguous | "Evidence marker", SBOM format and place, and "parser" undefined | Accept: define the marker, CycloneDX JSON attached to the release, list the fuzz targets |
| R-16 | 018, 024, 026, 033, 055, 099 | Complete | The CLI Standard version is not pinned | Accept: CLI Standard v1.1.0 |
| R-17 | Terminology | Conforming, Consistent | "Output" carries an obligation; "top-level window" vs "toplevel" | Accept: obligation moved to a requirement; one term |
| R-18 | 001, 091 | Method | `reuse lint` is executed, so test; coverage is analysis | Reject: §21.1 lists an SPDX header as its example of an inspection property, and coverage is measured from a test run, so the declared methods stand |
| Note | NEED-005 | Able to be validated | "Rebuilding a desktop never changes the login path" has no requirement | Accept: a requirement that Capsule's package closure holds no desktop-environment package, by inspection |

The reviewer judged sound, unprompted: every global named in 035 exists under that name; the
exit-code conventions (126, 127, 128+N, so 143 and 137 after 051 and 052); the `XKB_DEFAULT_*`
and `XCURSOR_*` variables; the pixman fallback on dumb buffers; the idle requirements together.

**Fixes: 2026-10-27** (T-080). R-01 to R-17 and the NEED-005 note resolved as triaged:

- Scope sentences open every requirement section and the chapter (R-01); CPS-SRS-010 and 102
  name their modes.
- New requirements CPS-SRS-113 to 127: host `DISPLAY` removed when nested (113); X11 child
  environment (114, optional); nested host unreachable (115); seat never active (116); fatal
  error while the child runs (117); GPU lost (118); seat lost (119); X display unreachable
  (120, optional); `--xwayland` not built (121); scale 1 (122); normal transform (123);
  unconstrained popup (124); built-in cursor (125); one output at a time (126, moved out of
  Terminology); no desktop package in the closure (127).
- CPS-SRS-071 now covers the exit paths Capsule handles; the `SIGKILL` case is assumption A-01,
  a `will` statement checked by a system test (R-03).
- Reworded: NEED-003, NEED-006, NEED-010; CPS-SRS-014, 015, 019, 021–023, 029, 034, 035,
  037, 042, 044, 056–058, 068, 081, 089, 092, 110; the Budgets preamble.
- Terminology: *connector name*, *lid closed*, *CLI Standard* (pinned to v1.1.0), *evidence
  marker*, *parser*; "top-level window" replaced by *toplevel*.

The set now holds 127 requirements, 112 of them required. **Next:** a third pass over the
changed requirements only, by a fresh reader (T-081).

---

*— Built by [Spacecraft Software](https://SpacecraftSoftware.org/) —*

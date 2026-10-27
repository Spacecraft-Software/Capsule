<!--
SPDX-FileCopyrightText: 2026 Mohamed Hammad <Mohamed.Hammad@SpacecraftSoftware.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->
<!-- GFM Document
     title:      Capsule — Plan
     author:     Mohamed Hammad & Spacecraft Software
     date:       2026-10-27
     version:    0.1.0
     license:    CC-BY-SA-4.0
     project:    Capsule
     website:    https://Capsule.SpacecraftSoftware.org/
-->

# Capsule — Plan

MVP: M0–M5 — Capsule v0.1 in the Bravais greeter

This is the §17.5 plan for [`PRD.md`](PRD.md). It cuts the PRD's milestones into work
packages; the requirements themselves live in [`doc/capsule.texi`](doc/capsule.texi) (from G1,
in `doc/requirements.toml`) and are never restated here. Each `P-` item names the requirements
it discharges. **Done rule:** a requirement-backed item is ticked when every requirement it
cites is `verified` in the traceability matrix (`nu tools/trace.nu`); an infrastructure or gate
item is ticked when it has landed on `main` and its gate record is written. Executable tasks
are in [`TODO.md`](TODO.md).

Template: `/spacecraft-software/escapepod` (tools, CI, posture set, packaging triad).
Skills per milestone: `microsoft-rust-guidelines` before any `.rs`, with
`spacecraft-rust-guidelines` for the event loop and the budgets; `spacecraft-cli-standard` and
`spacecraft-agentic-cli` at M1; `steelbore-color-palette` and
`spacecraft-accessibility-support` at M1–M2; `spacecraft-nix-guidelines` for `flake.nix` and
`packaging/default.nix`; `spacecraft-guile-guidelines` for `packaging/guix.scm`;
`spacecraft-texinfo-document` for the manual; `spacecraft-nu-guidelines` for `tools/`.

## Gates

| Gate | Closes with | Tag |
|---|---|---|
| G1 — Requirements | P-001 | `g1-requirements-0.1` |
| G2 — Design | P-008 | `g2-design-0.1` |
| G3 — Release | P-042 | `v0.1.0` |

Tags are signed and pushed only on the maintainer's word.

## M0 — Foundation and gates

- [ ] P-001 G1: §20.4 characteristics gate over CPS-SRS-001–105 and the needs; findings in
      `COMPLIANCE.md` "Gate records / G1"; statuses `draft → baselined`; signed tag
- [ ] P-002 Posture set: `README.md` (Category B, §19.6 claim, §13 N/A, support window),
      `SECURITY.md`, `CREDITS.md` (Cage, Smithay, ISO/IEC/IEEE 29148, ECSS-E-ST-40C),
      `COMPLIANCE.md`; `reuse lint` clean — CPS-SRS-001, 002, 004, 005, 006
- [ ] P-003 Agent context: `AGENTS.md` (Category B, layout, commands, invariants, forbidden
      patterns), `CLAUDE.md`, `SKILL.md` (command surface) — CPS-SRS-003, 004
- [ ] P-004 Registry: `PROJECTS.md` row (Category B, prefix `CPS`) in the Projects repo —
      CPS-SRS-004
- [ ] P-005 Workspace skeleton: `Cargo.toml` (crates `capsule-policy`, `capsule`; edition 2024;
      `rust-version`; release profile with every flag noted, `panic = "abort"`), `clippy.toml`,
      `rustfmt.toml`, `deny.toml`, `.cargo/audit.toml`, `flake.nix` devShell, `flake.lock` as
      the toolchain pin
- [ ] P-006 CI and local gate: `.github/workflows/ci.yml` and `tools/ci.nu` (fmt, clippy
      `-D warnings`, test, audit, deny, `reuse lint`, CR-byte check) — CPS-SRS-007, 008
- [ ] P-007 Requirements as data: `doc/requirements.toml`, `tools/srs.nu` rendering
      `doc/requirements.texi` into the manual, `tools/trace.nu` gating CI — CPS-SRS-009
- [ ] P-008 G2: manual `Design` node (architecture, the §3.2 serial trade-off, safety and
      privacy models); reference machine and workload recorded; budgets confirmed; `Interfaces`
      node drafted; dependencies qualified (Smithay crates.io vs git decided); seatd and logind
      hand-back evidence; signed tag

## M1 — Command line and child

- [ ] P-009 `capsule-policy`: exit-status mapping and the termination-deadline state machine
      — CPS-SRS-014, 015
- [ ] P-010 Argument parsing: program, trailing arguments, `--`, missing program, global
      flags, `--backend`, `--quiet` with `--verbose`, a backend or `--xwayland` not built in —
      CPS-SRS-011, 012, 013, 018, 029, 106, 108, 121
- [ ] P-011 Backend choice in `capsule-policy` — CPS-SRS-030
- [ ] P-012 Diagnostics: output-mode cascade, severity tags, JSON error object, UTC
      timestamps, plain mode, `--quiet` — CPS-SRS-024, 025, 026, 028, 078, 107
- [ ] P-013 Introspection: `--version`, `--help`, `--describe`, `--schema`, `--dry-run`
      (GPU and output from M3), `--version` as JSON — CPS-SRS-019, 020, 021, 022, 023, 111
- [ ] P-014 Child process: spawn with `PR_SET_PDEATHSIG`, 126/127 detection, `SIGCHLD`
      reaping, standard output left to the child — CPS-SRS-016, 017, 027
- [ ] P-015 `--verbose` report — CPS-SRS-031

## M2 — Nested compositor

- [ ] P-016 Compositor core: Smithay state, calloop loop, socket, child started after the
      socket, child environment and host `DISPLAY` removal, no configuration read, one
      output at scale 1 and normal transform — CPS-SRS-010, 032, 033, 034, 110, 113, 122,
      123, 126
- [ ] P-017 Closed global list, with a test that reads the registry — CPS-SRS-035
- [ ] P-018 Placement, stacking, and focus in `capsule-policy`, wired into `xdg_shell` —
      CPS-SRS-036, 037, 038, 039, 109
- [ ] P-019 Popups, constrained or unconstrained — CPS-SRS-040, 124
- [ ] P-020 Seat: keymap and fallback, pointer focus, cursor and built-in cursor —
      CPS-SRS-041, 042, 043, 044, 125
- [ ] P-021 Clipboard and primary selection — CPS-SRS-045, 046
- [ ] P-022 Decorations and the canvas-colour background (§11.6 resolution) —
      CPS-SRS-047, 048
- [ ] P-023 Robustness: protocol errors, lifetime, termination and its deadline, fatal
      errors — CPS-SRS-049, 050, 051, 052, 117
- [ ] P-024 Damage-driven rendering — CPS-SRS-053
- [ ] P-025 Nested backend: window, resize, close, no device nodes, host unreachable —
      CPS-SRS-073, 074, 075, 076, 115
- [ ] P-026 Key delivery: nothing bound except the virtual-terminal chords — CPS-SRS-077
- [ ] P-027 Integration harness: Capsule driven by a test client in CI; Alacritty + Nushell
      exercised nested under niri and under LeftWM

## M3 — Real hardware

- [ ] P-028 Seat session through libseat; the no-seat, never-active, and seat-lost cases —
      CPS-SRS-054, 055, 116, 119
- [ ] P-029 GPU and output choice in `capsule-policy` — CPS-SRS-056, 057, 058
- [ ] P-030 DRM output at the preferred mode, paced to vertical blank — CPS-SRS-059, 072
- [ ] P-031 Renderers: GLES through GBM and EGL, pixman fallback, no-renderer exit —
      CPS-SRS-060, 061, 062
- [ ] P-032 libinput devices at their defaults — CPS-SRS-063, 064
- [ ] P-033 Virtual-terminal switching, inactive and reactivated sessions —
      CPS-SRS-065, 066, 067
- [ ] P-034 Output re-choice on hotplug and lid events, the no-output state, GPU removal —
      CPS-SRS-068, 069, 118
- [ ] P-035 Hardware tests (feature `hardware-tests`): suspend, hand-back, and the
      `SIGKILL` system test for assumption A-01 —
      CPS-SRS-070, 071
- [ ] P-036 Greeter smoke test on the ThinkPad: an ad-hoc session entry, Alacritty + Nushell
      usable for a working session

## M4 — Budgets and release

- [ ] P-037 Budget harness on the reference machine; release-build measurement and margin
      report — CPS-SRS-079, 080, 081, 082, 083
- [ ] P-038 Fuzz targets for every parser in Capsule's own crates, run in CI — CPS-SRS-092
- [ ] P-039 Coverage gate and the `cargo mutants` triage — CPS-SRS-091, 093
- [ ] P-040 Manual: user chapters, `Design`, `Interfaces`; Info, HTML, and PDF builds —
      CPS-SRS-084, 090
- [ ] P-041 Packaging triad with `install-info` hooks; closure free of desktop packages —
      CPS-SRS-085, 086, 087, 088, 127
- [ ] P-042 G3: SBOM, release manifest, traceability matrix artifact, `CHANGELOG.md`, signed
      tag `v0.1.0` — CPS-SRS-089

## M5 — Bravais switch-over

- [ ] P-043 Bravais flake input `capsule`, built from the Nix package
- [ ] P-044 `mkShellSession` runs `capsule -- alacritty -e <shell>`; cosmic-comp leaves the
      login path; Bravais `PRD.md`, `AGENTS.md`, `CHANGELOG.md` updated
- [ ] P-045 Validation of CPS-NEED-001–009 in the greeter, recorded in the release manifest

## M6 — XWayland: optional

- [ ] P-046 `xwayland` feature: on-demand Xwayland, `DISPLAY`, window handling, off by default
      — CPS-SRS-094, 095, 096, 097 (optional)

## M7 — X11 kiosk mode: optional

- [ ] P-047 `capsule-x11`: window-manager ownership and the display-taken exit —
      CPS-SRS-098, 099, 120 (optional)
- [ ] P-048 Placement, stacking, focus, no key grabs, and root background through
      `capsule-policy` —
      CPS-SRS-100, 101, 102, 104, 105, 112 (optional)
- [ ] P-049 Exit-status contract and `xinit` use, tested under Xephyr and then on a machine
      without KMS; the unchanged child environment — CPS-SRS-103, 114, CPS-NEED-010 (optional)

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

Not yet run.

---

*— Built by [Spacecraft Software](https://SpacecraftSoftware.org/) —*

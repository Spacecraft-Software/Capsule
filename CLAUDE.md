<!--
SPDX-FileCopyrightText: 2026 Mohamed Hammad <Mohamed.Hammad@SpacecraftSoftware.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# CLAUDE.md

@AGENTS.md

> Record project knowledge in `AGENTS.md`, not here. This file holds only
> Claude-Code-only context (Standard §5.7).

## Skills to load

1. `engram-context capsule`, then `spacecraft-steelbore-standard`.
2. `microsoft-rust-guidelines` before any `.rs` file. Add `spacecraft-rust-guidelines`
   on top for the event loop and the frame-latency budget.
3. `spacecraft-cli-standard` + `spacecraft-agentic-cli` for the command line.
4. `spacecraft-accessibility-support` before anything touching key handling (§10.4
   reserved chords, §18 tailoring).
5. `spacecraft-nix-guidelines` for `flake.nix` and `packaging/default.nix`, and
   `spacecraft-guile-guidelines` for `packaging/guix.scm`.
6. `spacecraft-texinfo-document` for `doc/*.texi`.
7. `spacecraft-cli-shell` + `spacecraft-cli-preference` before shell commands;
   `spacecraft-missing-pkg` when a tool is absent.

Give the maintainer shell commands in Nushell syntax.

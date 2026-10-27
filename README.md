<!--
SPDX-FileCopyrightText: 2026 Mohamed Hammad <Mohamed.Hammad@SpacecraftSoftware.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Capsule

Capsule runs one program fullscreen on a Wayland seat, and exits with that
program's exit status when it ends. It is a memory-safe Rust successor to
[Cage](https://github.com/cage-kiosk/cage), built on
[Smithay](https://github.com/Smithay/smithay).

```sh
capsule -- alacritty -e nu
```

Status: pre-release, no code yet. See [`AGENTS.md`](AGENTS.md) for the scope,
including what "X11 support" does and does not mean here.

## Project Posture

Capsule is a **personal hobby project** under the Spacecraft Software umbrella
(The Steelbore Standard §5.1). It is provided as is, with no warranty and no
support commitment. Contributions are welcome and accepted at the maintainer's
discretion. See [`NOTICE.md`](NOTICE.md) and
[`CONTRIBUTING.md`](CONTRIBUTING.md).

## License

Software is `GPL-3.0-or-later`; documents are `CC-BY-SA-4.0`. The project
follows [REUSE](https://reuse.software): the texts are in `LICENSE` and
`LICENSES/`.

## Maintainer

Mohamed Hammad &lt;Mohamed.Hammad@SpacecraftSoftware.org&gt;
Website: <https://Capsule.SpacecraftSoftware.org/>

---

*— Built by Spacecraft Software —*

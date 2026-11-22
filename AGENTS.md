# AGENTS.md — pod-kde-vnc

The org rulebook at the umbrella (`opencharly/opencharly` `AGENTS.md`) applies in full.
This file adds only what is specific to this repo.

Canonical files:

- `charly.yml` — the `kde-vnc-session` candy (composes pod-sway, pod-wayvnc and the
  pod-kde-nested session with this pod's `KDE_NESTED_*` knobs), the
  `kde-vnc` image (based on `podcdi.pod-cdi`), the `check-kde-vnc-pod` disposable bed,
  and the `kde-vnc-skill` routing node.
- `kde-vnc.conf` — the sway drop-in: fullscreens the KWin window by its title (it has no
  app_id) and starts sway in passthrough mode so Plasma owns the keyboard.
- `.agents/skills/kde-vnc/SKILL.md` — the skill body.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-selkies:kde-vnc` — this repo's own skill.
- `/charly-selkies:sway`, `/charly-selkies:wayvnc`, `/charly-selkies:kde-nested`,
  `/charly-selkies:kde-shell` — the leaves this pod composes.
- `/charly-check:vnc`, `/charly-check:check` — the bed's verbs and the R10 sequence.
- `/charly-image:layer`, `/charly-image:image` — the candy and image authoring surface.
- `/charly-internals:git-workflow` — before any git/PR action.

## Rules specific to this repo

- **The pod's GPU comes from the host through CDI.** The base is `podcdi.pod-cdi`; never
  compose `layer-nvidia` or bake a driver here (see opencharly/pod-cdi `AGENTS.md`).
- **The nested Plasma session is pod-kde-nested's**, never a launcher of this repo's (R2).
  This repo only sets its `KDE_NESTED_*` knobs. The KWin socket must not be numbered
  (`wayland-kde`), because pod-sway's sway serves `wayland-1` behind its `wayland-0`
  symlink.
- **Keyboard input in the bed goes through `vnc:`**, never `wl: type`: `wtype` uploads a
  private keymap that the nested KWin re-maps (measured: typed text arrives garbled).
- **R10 gate:** `charly check run check-kde-vnc-pod` on a host with a GPU exposed through
  CDI. A change to a composed leaf (pod-sway, pod-wayvnc, layer-kde-shell) is verified
  here with `CHARLY_REPO_OVERRIDE=opencharly/<leaf>=<local tree>` before its own PR.

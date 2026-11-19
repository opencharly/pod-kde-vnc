---
name: kde-vnc
description: 'The pod-kde-vnc GPU pod: a fullscreen, hardware-accelerated KDE Plasma session nested in headless sway and served over wayvnc on :5900, on the CDI pod base.

  MUST be invoked before building, deploying, configuring or troubleshooting the kde-vnc image, its kde-vnc-session candy, or the check-kde-vnc-pod bed.'
---

# kde-vnc — fullscreen KDE Plasma over VNC, GPU-accelerated

## The stack

| layer | provided by | detail |
|---|---|---|
| base image | `podcdi.pod-cdi` | CachyOS; the GPU driver is injected from the host through CDI |
| outer compositor | `pod-sway` | headless sway 1.12; serves `wayland-1` with a `wayland-0` symlink; gles2 on the GPU |
| VNC | `pod-wayvnc` | wayvnc 0.10.2 on `:5900`; no power-on workaround needed at 0.10.x |
| desktop | `layer-kde-shell` | Plasma 6 without SDDM; `kdotool`, `spectacle` |
| session | `kde-vnc-session` (this repo) | nested `kwin_wayland` + `plasmashell` |

## How the session starts

`kde-vnc-session` (supervisord service, priority 15):

1. waits for `$XDG_RUNTIME_DIR/$KDE_NESTED_HOST_DISPLAY` (default `wayland-0`, sway);
2. starts `kwin_wayland --wayland-display <host> --socket wayland-kde --xwayland --width W --height H`;
3. starts `plasmashell` on `wayland-kde`;
4. lives as long as KWin does (supervisord restarts the whole session).

It exports `KDE_SESSION_VERSION=6`: nothing else sets it in a pod, and `kdotool` refuses to
run without it. `kwin_wayland` has its `cap_sys_nice` stripped at build, because the pod's
no-new-privileges refuses to exec a file-capability binary. A cap-less **copy** does not
work: KWin then cannot load its built-in Qt platform plugin.

## Fullscreen and keyboard

The nested KWin window has **no app_id**. Its title is
`KDE Wayland Compositor WL-0 — Press right control key to grab pointer`, so `kde-vnc.conf`
matches `title="^KDE Wayland Compositor"` and makes it fullscreen and borderless. sway
starts in **passthrough mode**, so its Alt bindings never swallow keys meant for Plasma.
`Alt+Pause` toggles passthrough off for sway administration.

## GPU acceleration

KWin renders through EGL on the GPU behind sway's buffers. With NVIDIA through CDI,
pod-sway's wrapper picks gles2 with the NVIDIA GBM/EGL vendor. Measured:
`Compositing Type: OpenGL ES`, `OpenGL renderer string: NVIDIA GeForce RTX 4080 SUPER`.
The same stack on an AMD GPU composites on `radeonsi`. Check it live with:

```bash
qdbus6 org.kde.KWin /KWin supportInformation | grep -E 'Compositing Type|OpenGL renderer'
```

## Testing (`check-kde-vnc-pod`)

| what | verb |
|---|---|
| services RUNNING | `command:` `supervisorctl status` |
| KWin fullscreen | `command:` `swaymsg -t get_tree` → `"fullscreen_mode": 1` |
| hardware GL | `command:` KWin `supportInformation`, renderer not `llvmpipe` |
| VNC up and the frame | `vnc: status`, `vnc: screenshot` + `artifact_min_dimensions` + `artifact_not_uniform` |
| input reaches Plasma | `vnc: type` + `vnc: key Return` into a Konsole, then `file:` the marker it created (absence asserted first) |

Drive keyboard input with `vnc:`, never `wl: type`: `wtype`'s private keymap is re-mapped
by the nested KWin, so the typed text arrives garbled.

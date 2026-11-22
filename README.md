# pod-kde-vnc — a GPU-accelerated KDE Plasma desktop over VNC

A pod that serves a **fullscreen KDE Plasma desktop** on VNC port 5900, rendered with
**hardware OpenGL** on the host's GPU.

```
sway (headless)  ── wayland-0 ──►  wayvnc  ── :5900 ──►  your VNC client
  └─ KWin (nested, fullscreen, --socket wayland-kde)
        └─ plasmashell + KDE applications
```

- **Base:** [opencharly/pod-cdi](https://github.com/opencharly/pod-cdi). The GPU driver
  comes from the host through CDI; nothing is baked into the image.
- **Compositor:** sway 1.12 (headless, from
  [pod-sway](https://github.com/opencharly/pod-sway)), the latest upstream release.
- **VNC:** wayvnc 0.10.2 (from [pod-wayvnc](https://github.com/opencharly/pod-wayvnc)),
  the latest upstream release.
- **Desktop:** KDE Plasma 6 without SDDM, as the shared nested session of
  [pod-kde-nested](https://github.com/opencharly/pod-kde-nested), nested in sway as one
  fullscreen, borderless window. sway runs in passthrough mode, so every key reaches
  Plasma (`Alt+Pause` leaves passthrough).

## Use it

```bash
charly box build kde-vnc
charly deploy add my-desktop kde-vnc
charly config my-desktop
charly start my-desktop
# connect a VNC client to the published 5900 port
```

Set `RESOLUTION=WxH` (default `1920x1080`) to size the desktop.

## Proof

`charly check run check-kde-vnc-pod` builds and deploys the pod, then checks that:

- sway, wayvnc and the Plasma session all run;
- KWin is fullscreen in sway;
- KWin composites with OpenGL on the GPU (not llvmpipe);
- wayvnc streams a full 1920x1080 frame;
- keystrokes sent over VNC run a command typed into a Konsole inside Plasma.

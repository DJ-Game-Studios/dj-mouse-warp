# DJ Mouse Warp

A GNOME Shell extension that makes cursor movement feel natural across monitor rows with different widths.

Built by [DJ Game Studios](https://github.com/DJ-Game-Studios) for the Linux workstations used to develop, test, and operate our games—and released because the same monitor-layout problem affects plenty of other setups.

When a smaller display sits above or below a wider monitor row, GNOME can leave parts of the boundary unreachable. DJ Mouse Warp maps the cursor proportionally between rows, eliminating those dead zones on Wayland and X11.

## Features

- Proportional horizontal mapping between monitor rows.
- Pressure-based warping across otherwise unreachable boundary areas.
- Safe destination snapping for layouts with gaps or negative coordinates.
- Live response to monitor hot-plug and layout changes.
- Optional visual feedback and per-monitor cursor overlays.
- Configurable sensitivity, cooldown, tolerance, and polling rate.
- Automatic idle behavior on single-monitor systems.

## Compatibility

- GNOME Shell 47–50
- Wayland and X11
- Landscape, portrait, ultrawide, and mixed-resolution displays
- Negative coordinates and non-contiguous monitor rows
- NVIDIA and Mesa graphics stacks

The extension currently handles top/bottom boundaries. Left/right dead zones caused by side-by-side monitors of different heights are not yet supported.

## Install

Clone the repository, then run:

```bash
./install.sh
```

Log out and back in on Wayland so GNOME Shell loads the extension. On X11, you can restart the shell with <kbd>Alt</kbd>+<kbd>F2</kbd>, then `r`.

Useful commands:

```bash
./install.sh --reload
./install.sh --uninstall
```

Extension UUID: `dj-mouse-warp@djmsqrvve`

## Configure

Open the preferences window with:

```bash
gnome-extensions prefs dj-mouse-warp@djmsqrvve
```

| Setting | Default | Purpose |
| --- | ---: | --- |
| Pressure threshold | 150 ms | How long the cursor pushes against a dead zone before warping. |
| Edge tolerance | 2 px | Distance from the boundary that activates dead-zone detection. |
| Warp cooldown | 100 ms | Prevents rapid back-and-forth warps. |
| Poll rate | 8 ms | Cursor sampling interval. |
| Overlap remapping | On | Preserves relative horizontal position during normal row crossings. |
| Visual feedback | On | Briefly marks the warp destination. |

If warps feel too sensitive, increase the pressure threshold. If they feel sluggish, decrease it. The defaults are designed to avoid accidental movement while remaining responsive.

## How it works

GNOME represents monitors in logical pixel space. DJ Mouse Warp groups displays into horizontal rows, calculates the cursor's relative position within the source row, and maps that ratio into the destination row:

```text
ratio = (x - sourceLeft) / sourceWidth
newX  = targetLeft + ratio * targetWidth
```

Before moving the pointer, the extension verifies that the destination is a real monitor pixel and snaps away from any layout gap. Geometry is recalculated when the monitor configuration changes.

## Visualizer

The included interactive visualizer shows monitor geometry, dead zones, and mapping paths:

```bash
xdg-open visualizer.html
```

## Development

Run the test suite locally with Node.js:

```bash
make test
```

Or use the containerized test environment:

```bash
docker compose run tests
```

The suite covers geometry, proportional mapping, pressure timing, gap safety, hot-plug resets, settings, and lifecycle cleanup.

See [DEVELOPMENT.md](DEVELOPMENT.md) for contributor setup and implementation notes.

## License

[MIT](LICENSE)

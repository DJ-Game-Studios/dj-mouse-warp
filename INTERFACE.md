# D-Bus Interface — dj-mouse-warp

Added in v2 (2026-05-19). Lets callers introspect the extension's live state and toggle it without round-tripping through GSettings.

## Bus Details

- **Bus Type**: Session bus
- **Destination**: `org.gnome.Shell`
- **Object Path**: `/org/gnome/Shell/Extensions/DjMouseWarp`
- **Interface**: `org.gnome.Shell.Extensions.DjMouseWarp`

## Methods

### Status

Return full extension state as a JSON string. Shape:

```jsonc
{
  "is_enabled": true,
  "warp_enabled": true,
  "overlap_remap_enabled": true,
  "overlay_enabled": false,
  "click_flash_enabled": false,
  "visual_feedback_enabled": true,
  "debug_logging": false,
  "hide_top_bar": false,
  "edge_tolerance": 2,
  "pressure_threshold_ms": 150,
  "warp_cooldown_ms": 100,
  "poll_rate_ms": 8,
  "row_tolerance": 5,
  "warp_count": 42,            // monotonic counter — incremented on every successful warp
  "last_warp": {               // null until a warp has occurred this session
    "x": 1234, "y": 800,
    "ts": 1747680000000000     // GLib.get_real_time() — microseconds since epoch
  },
  "monitors": [
    {"index": 0, "x": 0, "y": 0, "w": 2560, "h": 1440},
    {"index": 1, "x": 2560, "y": 0, "w": 2560, "h": 1440}
  ]
}
```

**Signature**: `Status() → string`

```bash
gdbus call --session \
  --dest org.gnome.Shell \
  --object-path /org/gnome/Shell/Extensions/DjMouseWarp \
  --method org.gnome.Shell.Extensions.DjMouseWarp.Status
```

### SetEnabled

Set the `is-enabled` GSetting. Returns the new state (same as the argument — the property write is synchronous).

**Signature**: `SetEnabled(enabled: boolean) → boolean`

### Toggle

Flip `is-enabled`. Returns the new state.

**Signature**: `Toggle() → boolean`

```bash
gdbus call --session \
  --dest org.gnome.Shell \
  --object-path /org/gnome/Shell/Extensions/DjMouseWarp \
  --method org.gnome.Shell.Extensions.DjMouseWarp.Toggle
```

### ResetCounters

Reset `warp_count` to 0 and `last_warp` to null. Useful for instrumented testing — start a known state, do some moves, read the counter. Returns `true` on success.

**Signature**: `ResetCounters() → boolean`

## Integration

| dj-cli | MCP | What it calls |
|---|---|---|
| `dj mouse warp status --json` | `dj_mouse_warp_status` | `Status` |
| `dj mouse warp toggle` | `dj_mouse_warp_toggle` | `Toggle` |
| `dj scene apply <name>` | `dj_scene_apply` | `SetEnabled` (via the scene's `warp` step) |
| — | — | `ResetCounters` (manual, no consumer wired) |

The top-bar dj-gnome-status widget's Quick Actions "Mouse warp: toggle" entry also calls `Toggle` directly over D-Bus.

## Error Handling

- If the extension is disabled, the D-Bus call fails with `org.freedesktop.DBus.Error.UnknownMethod` (the object goes away on disable).
- All handlers are wrapped in try/catch; a JS exception returns `org.freedesktop.DBus.Error.Failed` with the exception message rather than hanging the caller.
- Subprocess-level errors (e.g. settings write failing) surface as a thrown `Gio.IOErrorEnum` inside the handler, caught and returned to the caller.

## Implementation Notes

- The D-Bus object is registered in `enable()` and unregistered in `disable()`. Live reloads (Wayland: log out + back in) toggle this cleanly.
- `Status()` reads GSettings synchronously each time — no caching. 4 boolean keys + 5 int keys + monitor enumeration = sub-millisecond.
- `_warpCount` and `_lastWarp` are in-memory only; they reset on every shell reload. ResetCounters does the same thing as disabling + re-enabling, but cheaper.

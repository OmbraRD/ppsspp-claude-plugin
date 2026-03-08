# PPSSPP System Control & I/O Tools

## System Status (2 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `get_status()` | -- | Get the current emulator status (running, paused, stepping, no game loaded) |
| `get_game_info()` | -- | Get information about the currently loaded game (title, disc ID, version) |

**Notes:**
- `get_status` works even with no game loaded -- useful for checking if the emulator is ready.
- `get_status` returns: `status` (one of: "no_game", "stepping", "paused", "running"), `gameLoaded` (bool), `stepping` (bool), and `pc` (hex, only when a game is loaded).
- `get_game_info` returns: `id` (disc ID), `version` (disc version string), and `title`. Returns error if no game is loaded.

## Thread Inspection (1 tool)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_threads()` | -- | List PSP kernel threads with their status and PC |

**Notes:**
- Returns an array of threads, each with: `name`, `id`, `status` (running/ready/waiting/dormant/dead/suspended), `pc` (hex), `entrypoint` (hex), `priority` (int), `isCurrent` (bool).
- Useful for understanding which thread is executing, finding thread entry points, and debugging multi-threaded games.
- The PSP kernel is multithreaded -- games typically have separate threads for main logic, rendering, audio, and I/O.

## Screenshots (1 tool)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `take_screenshot(type?)` | type: "display" (default) or "render" | Capture a screenshot of the current PSP display as PNG |

**Screenshot types:**
- `display` (default) -- captures the final game output as shown on screen.
- `render` -- captures the in-progress render (may show partially drawn frame).

**Tips:**
- Returns the screenshot as an inline base64-encoded PNG image (not a file path).
- Use screenshots to verify game state, check UI navigation results, or document visual bugs.
- Combine with `pause()` to capture a specific frame.
- Has a 5-second timeout -- if the screenshot doesn't complete in time, it returns an error.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AbletonMCP is a Model Context Protocol (MCP) server that enables AI assistants to directly interact with and control Ableton Live digital audio workstation. It allows natural language commands to create tracks, add instruments, manipulate MIDI clips, and control playback for prompt-assisted music production.

**Repository:** https://github.com/ahujasid/ableton-mcp
**License:** MIT
**Python Version:** 3.10+ (server), 2/3 compatible (remote script)

## Architecture

The system uses a **two-component architecture** communicating via TCP sockets:

### 1. MCP Server (`MCP_Server/`)
- **Entry Point:** `MCP_Server/server.py` (main function)
- **Framework:** FastMCP (decorator-based MCP server framework)
- **Protocol:** stdio-based MCP server
- **Role:** Exposes 15 MCP tools to AI assistants, acts as TCP client to Ableton Remote Script
- **Key Class:** `AbletonConnection` - manages socket connection with retry logic and connection validation

### 2. Ableton Remote Script (`AbletonMCP_Remote_Script/`)
- **Entry Point:** `AbletonMCP_Remote_Script/__init__.py` (create_instance function)
- **Framework:** Inherits from `_Framework.ControlSurface`
- **Role:** Runs inside Ableton Live as MIDI Remote Script, creates TCP server on localhost:9877
- **Threading:** Uses daemon threads with queue-based responses for thread-safe communication
- **Key Class:** `AbletonMCP(ControlSurface)` - routes commands to handler methods, schedules state changes on main thread

### Communication Protocol
- **Format:** JSON over TCP sockets (localhost:9877)
- **Command Structure:** `{"type": "command_name", "params": {...}}`
- **Response Structure:** `{"status": "success|error", "result": {...}|"message": "..."}`
- **Buffer:** 8192 bytes with chunked receiving for large responses
- **Timeouts:** 10s (reads) / 15s (state-modifying operations)

## Development Commands

### Running the Server

**Development/Testing:**
```bash
uvx ableton-mcp
```

**Local Development:**
```bash
uv pip install -e .
python -m MCP_Server.server
```

**Integration:** Configure in Claude Desktop or Cursor MCP settings (stdio transport)

### Installation

**Via Smithery (for users):**
```bash
npx -y @smithery/cli install @ahujasid/ableton-mcp --client claude
```

**Via uvx:**
```bash
uvx ableton-mcp
```

### Remote Script Installation

Required for the MCP server to communicate with Ableton:

1. Download `AbletonMCP_Remote_Script/__init__.py`
2. Create folder `AbletonMCP` in Ableton's MIDI Remote Scripts directory:
   - macOS: `~/Music/Ableton/User Library/Remote Scripts/`
   - Windows: `%USERPROFILE%\Documents\Ableton\User Library\Remote Scripts\`
3. Place `__init__.py` in the `AbletonMCP/` folder
4. Configure in Ableton: Preferences > Link, Tempo & MIDI > Control Surface > AbletonMCP
5. Restart Ableton Live

### Building

**Package:**
```bash
uv build
```

**Docker:**
```bash
docker build -t ableton-mcp .
```

## MCP Tools Exposed

The server exposes 15 tools for controlling Ableton Live:

**Session Management:**
- `get_session_info()` - Get tempo, tracks, time signature
- `set_tempo(tempo)` - Set session tempo

**Track Operations:**
- `get_track_info(track_index)` - Get track details (name, color, devices, clips)
- `create_midi_track(index)` - Create MIDI track at position
- `set_track_name(track_index, name)` - Rename track

**Clip Operations:**
- `create_clip(track_index, clip_index, length)` - Create MIDI clip
- `add_notes_to_clip(track_index, clip_index, notes)` - Add MIDI notes (pitch, start, duration, velocity)
- `set_clip_name(track_index, clip_index, name)` - Rename clip
- `fire_clip(track_index, clip_index)` - Start clip playback
- `stop_clip(track_index, clip_index)` - Stop clip playback

**Device/Instrument Management:**
- `load_instrument_or_effect(track_index, uri)` - Load instrument/effect by browser URI
- `get_browser_tree(category_type)` - Get hierarchical browser structure (instruments/audio_effects/midi_effects/etc.)
- `get_browser_items_at_path(path)` - Get items at browser path (e.g., "instruments/synths/bass")

**Playback Control:**
- `start_playback()` - Start session playback
- `stop_playback()` - Stop session playback

## Critical Technical Details

### Connection Management
- Persistent global connection with 3 retry attempts
- Connection validated on startup via `get_session_info` test
- Automatic reconnection on socket errors
- Server must start **after** Ableton is fully loaded

### Thread Safety
- **Critical:** All Ableton Live API modifications must occur on main thread
- State-modifying commands automatically scheduled via `schedule_message()` in Remote Script
- Queue-based response handling with 10-second timeout prevents race conditions
- 100ms delays added after state changes for stability

### Python Compatibility (Remote Script)
The Remote Script must support Python 2 for older Ableton versions:
- Use conditional imports: `Queue` (Py2) vs `queue` (Py3)
- Explicit encoding: `json.dumps(...).encode('utf-8')` and `data.decode('utf-8')`
- Use `.format()` instead of f-strings
- Handle `JSONDecodeError` as `ValueError` for Py2 compatibility

### Important Constraints
1. **Single Instance Only:** Run MCP server in either Claude Desktop OR Cursor, not both simultaneously
2. **Port Requirement:** localhost:9877 must be available (no other socket servers on this port)
3. **Ableton Requirement:** Ableton Live 10+ must be running with AbletonMCP Remote Script loaded
4. **Browser Timing:** Browser queries fail if Ableton is still loading after startup

### Browser Navigation
- **Hierarchical paths:** Use forward slashes (e.g., `"instruments/synths/bass"`)
- **URI format:** Complex browser URIs required for loading (e.g., `query:Synths#Instrument%20Rack:Bass:FileId_5116`)
- **Workflow:** Use `get_browser_tree()` or `get_browser_items_at_path()` to find URIs, then `load_instrument_or_effect()` to load

## Code Patterns

### Error Handling Pattern
```python
try:
    # Operation
except socket.error as e:
    logger.error(f"Socket error: {e}")
    # Attempt reconnection or graceful failure
```

### Command Routing Pattern (Remote Script)
```python
def handle_command(self, command_type, params):
    if command_type == "set_tempo":
        self.schedule_message(lambda: self._set_tempo(params))
    elif command_type == "get_session_info":
        return self._get_session_info()  # Read-only, no scheduling needed
```

### Connection Validation Pattern
```python
# Always validate connection before use
conn = get_ableton_connection()
if not conn:
    raise ValueError("Failed to connect to Ableton")
```

## Dependencies

**Core:**
- `mcp[cli]>=1.3.0` - FastMCP framework
- `anyio` - Async I/O
- `httpx` - HTTP client
- `pydantic` - Data validation

**Deployment:**
- `uv` - Package manager (Astral)
- Smithery - Deployment platform
- Docker - Alpine-based Python 3.10 image

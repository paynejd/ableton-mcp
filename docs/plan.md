# AbletonMCP Development Plan

This document tracks completed features and planned enhancements for the AbletonMCP server.

---

## Ableton MCP Features

### ✅ Device Parameter Control (Added by @paynejd)
- Get all parameters for a device with names, values, min/max ranges, and quantization info
- Get individual parameter value with detailed metadata
- Set device parameter values with range validation and thread safety

### ✅ Track Mixer Controls (Added by @paynejd)
- Set track properties via consolidated `set_track_property` tool (11 properties):
  - Mixer controls: volume, pan, mute, solo, arm
  - Visual organization: name, color, color_index, fold_state
  - Routing: input_routing_type, output_routing_type
- Set track send levels to return tracks
- Enhanced track info retrieval with sends, color, and organizational properties

### ✅ Cue Point Management (Added by @paynejd)
- List all cue points in arrangement
- Create cue points at specific times
- Rename and reposition cue points
- Delete cue points

### ✅ Core Session & Track Operations (Original by @ahujasid)
- Get session info (tempo, time signature, track counts)
- Get detailed track info (clips, devices, mixer state)
- Create MIDI tracks
- Set track names
- Set session tempo

### ✅ Clip Operations (Original by @ahujasid)
- Create MIDI clips with specified length
- Add MIDI notes to clips (pitch, time, duration, velocity)
- Set clip names
- Fire and stop clips

### ✅ Playback Control (Original by @ahujasid)
- Start and stop session playback

### ✅ Browser & Device Loading (Original by @ahujasid)
- Get hierarchical browser tree by category
- Get browser items at specific paths
- Load instruments and effects by URI
- Load drum kits

---

## Near-Term Planned Features (Top 10 Priority)

### 1. MIDI Clip Reading & Editing (HIGH PRIORITY - Critical Gap)
**Tools to Add:**
- `get_clip_notes(track_index, clip_index, start_time?, end_time?)` - Read MIDI notes from existing clips
- `remove_clip_notes(track_index, clip_index, note_ids)` - Delete specific notes by ID
- `replace_clip_notes(track_index, clip_index, notes)` - Replace all notes in a clip

**Why:** Community request (Issue #35), enables bidirectional MIDI workflow, composition editing, pattern analysis

---

### 2. Scene Management (MEDIUM PRIORITY)
**Tools to Add:**
- `get_scenes()` - List all scenes with names, colors, tempo overrides
- `create_scene(index, name?)` - Insert new scene at position
- `fire_scene(scene_index)` - Launch all clips in a scene simultaneously
- `delete_scene(scene_index)` - Remove scene
- `set_scene_property(scene_index, property, value)` - Set scene name, color, tempo

**Why:** Core session view workflow, enables structured composition, missing "fire all" capability

---

### 3. View & Navigation Control (MEDIUM PRIORITY)
**Tools to Add:**
- `switch_to_session_view()` - Show Session View
- `switch_to_arrangement_view()` - Show Arrangement View
- `select_track(track_index)` - Programmatically select track
- `show_clip(track_index, clip_index)` - Highlight clip in UI
- `get_playback_position()` - Get current transport position in beats
- `set_playback_position(time)` - Jump to specific position

**Why:** PR #17 exists, enables arrangement view workflows, better UX feedback, navigation automation

---

### 4. Track Deletion & Management (LOW PRIORITY - PR in progress)
**Tools to Add:**
- `delete_track(track_index)` - Remove track from session
- `delete_clip(track_index, clip_index)` - Remove clip from slot
- `duplicate_track(track_index, new_index?)` - Duplicate track with all settings

**Why:** PR #23 in draft, completes track lifecycle management

---

### 5. Clip Properties & Loop Settings (MEDIUM PRIORITY)
**Tools to Add:**
- `set_clip_property(track_index, clip_index, property, value)` - Consolidated clip control
  - Loop settings: loop_start, loop_end, loop_enabled
  - Playback: start_marker, end_marker
  - Color and name (consolidate existing)
  - Quantization settings

**Why:** Essential for clip editing workflows, complements MIDI note reading/writing

---

### 6. Recording & Capture (MEDIUM PRIORITY)
**Tools to Add:**
- `capture_midi()` - Retroactively capture recent MIDI input
- `start_recording(track_index?)` - Begin recording on armed tracks
- `stop_recording()` - Stop all recording
- `set_record_mode(mode)` - Set recording mode (overdub, punch-in, etc.)
- `set_metronome(enabled)` - Enable/disable metronome

**Why:** PR #17 mentions recording, essential for input-based workflows

---

### 7. Return Track Controls (MEDIUM PRIORITY)
**Tools to Add:**
- `create_return_track(index)` - Add new return track
- `get_return_track_info(index)` - Get return track details
- `set_return_track_property(index, property, value)` - Control return track mixer

**Why:** PR #26 exists for this, completes mixing workflow with sends

---

### 8. Audio Clip Controls (LOW PRIORITY)
**Tools to Add:**
- `set_audio_clip_property(track_index, clip_index, property, value)` - Audio clip settings
  - Warp mode (Complex, Beats, Texture, etc.)
  - Warp enabled/disabled
  - Pitch shift
  - Gain/volume

**Why:** Enables audio production workflows, currently MIDI-only

---

### 9. Arrangement Operations (MEDIUM PRIORITY)
**Tools to Add:**
- `get_arrangement_info()` - Get loop brackets, playback state
- `set_loop_brackets(start, end)` - Set arrangement loop region
- `enable_arrangement_loop(enabled)` - Toggle arrangement loop
- `get_arrangement_clips(track_index)` - List clips in arrangement view

**Why:** Issue #13 requests arrangement view support, enables linear composition workflows

---

### 10. Advanced Device Management (LOW PRIORITY)
**Tools to Add:**
- `get_device_chains(track_index, device_index)` - Access rack device chains
- `get_drum_pads(track_index, device_index)` - Access drum rack pads
- `load_device_preset(track_index, device_index, preset_path)` - Load device presets
- `set_device_enabled(track_index, device_index, enabled)` - Enable/disable devices

**Why:** Enables complex instrument programming, rack-based workflows

---

## Future LOM Features (By Category)

### Groove & Quantization
- Access groove pool
- Apply grooves to clips
- Set global quantization settings
- Clip-level quantization controls
- Swing and groove amount parameters

### Automation
- Read automation envelopes for parameters
- Write automation data
- Set automation arm state per parameter
- Re-enable automation after manual override
- Automation modes (latch, touch, write)

### Track Organization & Grouping
- Create group tracks
- Add tracks to groups
- Reorder tracks in session
- Track folding (expand/collapse groups)
- Track freezing for CPU optimization

### Advanced Routing
- Set input/output routing channels (beyond type)
- Configure monitoring modes (In, Auto, Off)
- Crossfader assignment (A, B, Off)
- Track available input/output lists
- External audio/MIDI routing

### Application-Level Controls
- Undo/redo operations
- Save and save-as operations
- CPU usage monitoring (average/peak)
- Tempo automation
- MIDI/key mapping management
- Dialog and message display

### Performance & Monitoring
- Read track output meter levels
- Observe clip playing status changes
- Read peak levels
- Master output monitoring
- Audio engine state

### Clip Envelopes & Follow Actions
- Clip volume/pan envelopes
- Device parameter envelopes within clips
- Clip follow actions (what happens after clip ends)
- Clip launch quantization
- Clip velocity/pitch modulation

### Browser Advanced Features
- User library management
- Favorites and tags system
- Recent/used items tracking
- Pack installation and management
- Color scheme browsing

### Specialized Devices
- Specific device parameter mapping (Compressor, EQ8, Reverb, etc.)
- Simpler/Sampler zone manipulation
- Wavetable oscillator controls
- Max for Live device communication
- VST/AU plugin parameter access by name

### Audio Warp & Time Manipulation
- Warp marker creation and manipulation
- Transient detection
- Audio clip stretching
- Time signature per clip
- Sample start/end point editing

---

## LOM Coverage Statistics

- **Current Coverage:** ~25-30% of Live Object Model
- **After Near-Term Features:** ~50-60% of LOM
- **Total MCP Tools:**
  - Currently: 20 tools (15 original + 3 device parameter + 2 track mixer)
  - After top 10: ~30-35 tools (using consolidated approach)
  - Avoided tool proliferation: Would be 40+ with individual tools

---

## Community-Requested Features

### From GitHub Issues:
- **Issue #35:** Read MIDI notes from clips ✅ **PLANNED - Priority #1**
- **Issue #7:** Modify device parameters ✅ **COMPLETED**
- **Issue #13:** Arrangement view support ✅ **PLANNED - Priority #9**
- **Issue #14:** Access samples through browser ✅ **PARTIALLY COMPLETED** (browser access exists)
- **Issue #27:** Problems creating beats/melodies → **Addressed by MIDI reading (#1)**

### From Pull Requests:
- **PR #41:** Device parameter control ✅ **COMPLETED**
- **PR #8:** Get/set device parameters ✅ **COMPLETED**
- **PR #26:** Return track and mixing controls ✅ **PARTIALLY COMPLETED** (mixer done, return tracks planned #7)
- **PR #23:** Delete track/clip ✅ **PLANNED - Priority #4**
- **PR #17:** Record session clip & view switching ✅ **PLANNED - Priorities #3, #6**

---

## Design Principles

### Tool Consolidation Strategy
- Use property-based tools (`set_track_property`, `set_clip_property`) to avoid tool proliferation
- Group related operations under single tools with string enum parameters
- Separate tools only when multi-index parameters are required (e.g., `set_track_send`)
- Balance between LLM clarity and maintainability

### Future-Proofing
- Property-based tools can be extended with new enum values without adding tools
- Maintain consistent patterns across Track, Clip, Scene, and Device operations
- Document property options comprehensively in tool docstrings
- Use read-write pairs for complex objects (get_X, set_X)

### Thread Safety
- All state-modifying operations scheduled on Ableton's main thread
- Queue-based response handling with timeouts
- 100ms delays after state changes for stability
- Read-only operations execute directly without scheduling

---

## Version History

- **v0.1:** Initial release - basic session, track, clip, and playback control
- **v0.2:** Added cue point management, browser navigation improvements
- **v0.3:** ✅ Device parameter control (3 tools)
- **v0.4:** ✅ Track mixer controls with consolidated approach (2 tools, 11 properties)
- **v0.5:** *NEXT* - MIDI clip reading & editing

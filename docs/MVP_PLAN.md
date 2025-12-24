# LapD - YouTube to MP3 Converter for Swimming Headphones

## Project Overview

LapD is a Tauri-based desktop application designed to download YouTube videos, convert them to MP3, and prepare them for swimming headphones that have limited playback features (no resume, basic ordering).

### Tech Stack
- **Frontend**: Vue.js 3 with TypeScript
- **Backend**: Rust (Tauri 2.x)
- **External Tool**: yt-dlp for YouTube downloads
- **Audio Processing**: FFmpeg for MP3 conversion and splitting

---

## MVP Feature Set

### Core Features

#### 1. YouTube Video Queue
- **Add videos to queue**
  - Paste YouTube URL
  - Validate URL before adding
  - Display video title, duration, thumbnail (fetched via yt-dlp)
  - Support for single videos and playlists

- **Queue management**
  - View all queued videos
  - Remove individual videos from queue
  - Clear entire queue
  - Show download progress per video
  - Show overall queue progress

#### 2. Output Folder Configuration
- **Folder selection**
  - Browse and select output directory
  - Remember last used folder (persistent settings)
  - Display current output folder path
  - Validate folder exists and is writable

#### 3. Download & Convert
- **YouTube to MP3 pipeline**
  - Download video using yt-dlp
  - Extract audio and convert to MP3
  - Show real-time progress (percentage, download speed)
  - Handle errors gracefully (network issues, invalid URLs, etc.)
  - Background processing (non-blocking UI)

#### 4. File Naming & Ordering
- **Custom naming scheme**
  - Prefix files with numbers for ordered playback (e.g., `001_video_title.mp3`)
  - Auto-increment based on queue order
  - Sanitize filenames (remove invalid characters)
  - Optional: Allow user to define naming template

- **Ordering strategies**
  - Queue order (default)
  - Optional: Manual reordering in queue before download

#### 5. MP3 Splitting & Transformation

##### A. Chapter-based Splitting
- **YouTube chapters detection**
  - Extract chapter markers from YouTube video metadata
  - Display chapters to user before download
  - Allow user to enable/disable splitting

- **Smart splitting logic**
  - If chapters exist and are < max duration threshold: split by chapters
  - If chapters exist but exceed threshold: apply interval splitting within long chapters
  - If no chapters: apply interval-based splitting

##### B. Interval-based Splitting
- **Configurable intervals**
  - Default: 5 minutes
  - User-configurable (e.g., 3, 5, 10, 15 minutes)
  - Split long files into segments

- **Segment naming**
  - Maintain ordered numbering across segments
  - Example: `001_video_title_part1.mp3`, `002_video_title_part2.mp3`

---

## Architecture & Implementation Plan

### Rust Backend (Tauri Commands)

#### Core Modules

```
src-tauri/src/
├── main.rs                 # Entry point
├── lib.rs                  # Tauri setup and command registration
├── commands/
│   ├── mod.rs              # Command module exports
│   ├── youtube.rs          # YouTube-related commands
│   ├── audio.rs            # Audio processing commands
│   └── settings.rs         # Settings management
├── services/
│   ├── mod.rs              # Service module exports
│   ├── ytdlp.rs            # yt-dlp wrapper
│   ├── ffmpeg.rs           # FFmpeg wrapper for splitting
│   └── queue.rs            # Queue management
├── models/
│   ├── mod.rs              # Model exports
│   ├── video.rs            # Video/queue item model
│   └── settings.rs         # Settings model
└── utils/
    ├── mod.rs              # Utility exports
    └── file.rs             # File naming, sanitization
```

#### Key Tauri Commands

```rust
// YouTube operations
#[tauri::command]
async fn add_to_queue(url: String) -> Result<VideoInfo, String>

#[tauri::command]
async fn remove_from_queue(id: String) -> Result<(), String>

#[tauri::command]
async fn get_queue() -> Result<Vec<VideoInfo>, String>

#[tauri::command]
async fn start_download_queue(
    output_dir: String,
    split_config: SplitConfig
) -> Result<(), String>

// Settings operations
#[tauri::command]
async fn get_settings() -> Result<Settings, String>

#[tauri::command]
async fn update_settings(settings: Settings) -> Result<(), String>

#[tauri::command]
async fn select_output_folder() -> Result<String, String>

// Video info operations
#[tauri::command]
async fn fetch_video_info(url: String) -> Result<VideoInfo, String>
```

#### Data Models

```rust
#[derive(Serialize, Deserialize, Clone)]
struct VideoInfo {
    id: String,
    url: String,
    title: String,
    duration: u32,  // seconds
    thumbnail: Option<String>,
    chapters: Vec<Chapter>,
    status: DownloadStatus,
}

#[derive(Serialize, Deserialize, Clone)]
struct Chapter {
    title: String,
    start_time: u32,  // seconds
    end_time: u32,
}

#[derive(Serialize, Deserialize, Clone)]
enum DownloadStatus {
    Queued,
    Downloading { progress: f32 },
    Converting,
    Splitting,
    Completed,
    Failed { error: String },
}

#[derive(Serialize, Deserialize, Clone)]
struct SplitConfig {
    enabled: bool,
    use_chapters: bool,
    interval_minutes: u32,
    max_chapter_minutes: u32,  // Threshold for "too long"
}

#[derive(Serialize, Deserialize, Clone)]
struct Settings {
    output_folder: String,
    split_config: SplitConfig,
    naming_template: String,  // Future: custom naming
}
```

### Frontend (Vue.js)

#### Component Structure

```
src/
├── App.vue                 # Main app component
├── components/
│   ├── ui/                 # Reusable UI components (buttons, etc.)
│   ├── Queue/
│   │   ├── QueueList.vue           # Display queue items
│   │   ├── QueueItem.vue           # Individual queue item
│   │   └── AddVideoForm.vue        # URL input form
│   ├── Settings/
│   │   ├── SettingsPanel.vue       # Settings UI
│   │   ├── FolderSelector.vue      # Output folder selection
│   │   └── SplitSettings.vue       # Split configuration
│   └── Progress/
│       └── DownloadProgress.vue    # Progress indicators
├── stores/
│   ├── queue.ts            # Queue state management (Pinia)
│   └── settings.ts         # Settings state management
└── utils/
    └── tauri.ts            # Tauri command wrappers
```

---

## Implementation Phases

### Phase 1: Foundation ✓ Target: Week 1
- [x] Set up Tauri project with Vue.js
- [ ] Fix any build/dependency issues
- [ ] Install yt-dlp and FFmpeg dependencies
- [ ] Create basic UI layout with routing (if needed)

### Phase 2: Basic Download ✓ Target: Week 2
- [ ] Implement yt-dlp wrapper in Rust
- [ ] Create `fetch_video_info` command
- [ ] Create `download_video` command (no splitting)
- [ ] Build queue UI components
- [ ] Implement add/remove from queue
- [ ] Basic progress tracking

### Phase 3: Settings & Persistence ✓ Target: Week 3
- [ ] Implement settings storage (tauri-plugin-store)
- [ ] Create settings UI
- [ ] Folder selection dialog
- [ ] Persist queue state
- [ ] File naming with ordered prefixes

### Phase 4: Splitting Features ✓ Target: Week 4
- [ ] Implement FFmpeg wrapper for splitting
- [ ] Chapter detection from yt-dlp metadata
- [ ] Interval-based splitting logic
- [ ] Smart splitting (chapters + intervals)
- [ ] Split configuration UI
- [ ] Testing with various video types

### Phase 5: Polish & Testing ✓ Target: Week 5
- [ ] Error handling improvements
- [ ] Loading states and user feedback
- [ ] Batch download queue processing
- [ ] Final UI polish
- [ ] Cross-platform testing (if applicable)
- [ ] Documentation

---

## Technical Decisions

### Why yt-dlp?
- Most reliable YouTube downloader
- Handles various formats and sites (future expansion)
- Active development and maintenance
- Chapter/metadata extraction support

### Why FFmpeg?
- Industry standard for audio processing
- Precise splitting at any timestamp
- No quality loss with stream copying
- Cross-platform

### State Management
- **Queue**: In-memory during runtime, persisted to disk
- **Settings**: Tauri plugin store for persistence
- **Progress**: Event-based streaming from Rust to frontend

### File Naming Strategy
```
Format: {order}_{sanitized_title}_{part}.mp3

Examples:
- Single file: "001_How_To_Swim_Faster.mp3"
- Split file: "001_How_To_Swim_Faster_part1.mp3"
              "002_How_To_Swim_Faster_part2.mp3"
```

---

## Dependencies

### Rust (Cargo.toml)
```toml
[dependencies]
tauri = { version = "2", features = ["dialog-open"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.11", features = ["json"] }
tauri-plugin-store = "2"  # For settings persistence
tauri-plugin-dialog = "2"  # For folder selection
uuid = { version = "1", features = ["v4"] }  # For queue IDs
```

### System Dependencies
- **yt-dlp**: Must be installed and in PATH
- **FFmpeg**: Must be installed and in PATH

### Frontend (package.json)
```json
{
  "dependencies": {
    "vue": "^3.4",
    "@vueuse/core": "^10.7",
    "pinia": "^2.1"  // State management
  }
}
```

---

## Future Enhancements (Post-MVP)

### Content Sources
- Podcast RSS feeds
- SoundCloud
- Spotify (if possible)
- Local file import

### Advanced Features
- Playlist auto-download
- Scheduled downloads
- Audio quality selection
- Speed adjustment (1.25x, 1.5x, etc.)
- Metadata editing (ID3 tags)
- Cloud sync for queue/settings

### Headphone-Specific
- Bookmarking system (external tracking file)
- Resume position tracking
- Multiple device profiles

### UI/UX
- Dark mode
- Drag-and-drop URLs
- Keyboard shortcuts
- System tray integration
- Desktop notifications

---

## Open Questions

1. **Headphone playback order**: How exactly do the target headphones order files? (alphabetically, by date, etc.)
2. **Chapter length threshold**: What's the maximum acceptable chapter length before splitting? (suggest 15 minutes)
3. **Queue persistence**: Should queue survive app restarts or start fresh?
4. **Concurrent downloads**: Download one at a time or allow parallel downloads?
5. **Storage limits**: Should there be warnings for large downloads or disk space checks?

---

## Success Criteria

The MVP will be considered successful when:
- ✓ Users can add multiple YouTube URLs to a queue
- ✓ Videos download and convert to MP3 format
- ✓ Files are named with ordered prefixes
- ✓ Long videos can be split based on chapters or intervals
- ✓ All files save to user-specified folder
- ✓ Progress is visible for all operations
- ✓ Settings persist across sessions
- ✓ The app handles errors gracefully

---

## Development Setup

### Prerequisites
```bash
# Install yt-dlp
pip install yt-dlp
# or
brew install yt-dlp  # macOS

# Install FFmpeg
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt install ffmpeg

# Windows (using Chocolatey)
choco install ffmpeg
```

### Running the App
```bash
# Install dependencies
npm install
cd src-tauri && cargo build

# Development mode
npm run tauri dev

# Build for production
npm run tauri build
```

---

## Risk Mitigation

### Technical Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| yt-dlp API changes | High | Pin version, monitor updates |
| YouTube blocking | High | Use latest yt-dlp, consider fallbacks |
| Large file processing | Medium | Stream processing, progress feedback |
| Cross-platform issues | Medium | Test on all target platforms early |

### User Experience Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| Complex UI | Medium | Progressive disclosure, sensible defaults |
| Slow downloads | Medium | Clear progress, allow background operation |
| File organization confusion | Low | Clear naming, preview before download |

---

*Last updated: 2025-12-24*

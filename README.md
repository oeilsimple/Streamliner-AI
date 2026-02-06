# Streamliner-AI 🤖🎬

## Overview

**Streamliner-AI** is a fully automated, asynchronous Python pipeline designed to monitor Kick streamers, detect viral high-energy moments, generate vertical clips optimized for social media, and publish them to TikTok without manual intervention.

The system leverages the **official Kick API** with **OAuth2 Client Credentials** authentication for stable, efficient, and robust stream monitoring. It intelligently processes live streams in real-time or recorded VODs to identify highlights using audio energy analysis, speech-to-text transcription, and keyword detection.

## Features

- **Official API Authentication**: Connects to Kick using OAuth2 Client Credentials flow for stable and authorized access
- **Asynchronous Monitoring**: Uses `asyncio` and `httpx` to monitor multiple streamers concurrently with a single process
- **Intelligent Detection System**:
  - Analyzes audio energy (RMS) to quickly identify emotional peaks
  - Uses `faster-whisper` to transcribe only high-energy segments, saving processing time
  - Customizable scoring system combining audio energy and 200+ keyword patterns
- **Automatic Vertical Rendering**: Uses `ffmpeg` to create 9:16 format clips with blurred backgrounds, centered original content, and burned-in subtitles
- **TikTok Publishing**: Integrates with TikTok Content API for automatic clip uploads
- **Robust CLI**: Command-line interface built with `click` for easy management
- **Production Ready**: Includes Docker configuration, unit tests, and CI pipeline with GitHub Actions
- **Real-time Processing**: Supports both live stream chunk processing and full VOD analysis
- **Flexible Storage**: Supports local filesystem and S3-compatible storage (AWS S3, Cloudflare R2)

## Technology Stack

### Core Technologies
- **Python 3.10+** (developed and tested with Python 3.13.3)
- **asyncio** - Asynchronous I/O for concurrent stream monitoring
- **httpx** - Modern async HTTP client with HTTP/2 support

### Media Processing
- **FFmpeg** - Video/audio processing, cutting, and rendering
- **Streamlink** - Stream extraction and downloading
- **faster-whisper** - Efficient speech-to-text transcription
- **soundfile** & **scipy** - Audio analysis and processing
- **scenedetect** - Scene change detection for highlight optimization

### AI/ML
- **PyTorch** - Deep learning framework for Whisper model
- **faster-whisper** - Optimized Whisper implementation

### APIs & Integration
- **Kick API** - OAuth2 authentication and stream monitoring
- **TikTok Content API** - Automated video publishing

### Development & Testing
- **pytest** - Unit testing framework
- **ruff** - Fast Python linter and formatter
- **Docker** - Containerization for deployment
- **GitHub Actions** - Continuous Integration pipeline

### Configuration & Logging
- **python-dotenv** - Environment variable management
- **PyYAML** - Configuration file parsing
- **loguru** - Advanced logging with structured output
- **click** - CLI framework

## Project Structure

```
streamliner-ai/
├── .github/
│   └── workflows/
│       └── ci.yml              # CI/CD pipeline configuration
├── assets/
│   ├── architecture-diagram.png
│   └── logo.png
├── scripts/
│   └── generate_tiktok_tokens.py  # TikTok OAuth token generator
├── src/
│   └── streamliner/
│       ├── __init__.py
│       ├── cli.py              # Command-line interface
│       ├── config.py           # Configuration management
│       ├── cutter.py           # Video cutting utilities
│       ├── detector.py         # Highlight detection engine
│       ├── downloader.py       # Stream/VOD downloader
│       ├── monitor.py          # Stream monitoring system
│       ├── pipeline.py         # Main processing pipeline
│       ├── render.py           # Video rendering engine
│       ├── stt.py              # Speech-to-text transcription
│       ├── worker.py           # Real-time chunk processor
│       ├── publisher/
│       │   ├── __init__.py
│       │   └── tiktok.py       # TikTok API integration
│       └── storage/
│           ├── __init__.py
│           ├── base.py         # Storage interface
│           ├── local.py        # Local filesystem storage
│           └── s3.py           # S3-compatible storage
├── tests/
│   ├── test_cutter.py
│   ├── test_detector.py
│   └── test_worker_cleanup.py
├── .env.template               # Environment variables template
├── .gitignore
├── config.yaml.example         # Configuration template
├── docker-composer.yml         # Docker Compose configuration
├── Dockerfile                  # Docker image definition
├── pyproject.toml              # Python project metadata
├── README.md
└── requirements.txt            # Python dependencies
```

### Key Components

- **cli.py**: Entry point for all commands (monitor, process, upload)
- **monitor.py**: Manages real-time stream monitoring and chunk recording
- **detector.py**: Analyzes audio and transcripts to identify highlights
- **pipeline.py**: Orchestrates the complete processing workflow
- **worker.py**: Handles real-time chunk processing and buffering
- **render.py**: Creates vertical format videos with subtitles
- **publisher/tiktok.py**: Handles TikTok API authentication and uploads
- **storage/**: Abstraction layer for local and cloud storage

## Architecture

The system operates as a stable pipeline using official Kick authentication to ensure reliable data access.

![Architecture Diagram](assets/architecture-diagram.png)

### Workflow

1. **Monitor**: Continuously checks configured streamers' status via Kick API
2. **Record**: When live, records stream in chunks using FFmpeg
3. **Detect**: Analyzes audio energy (RMS) to find emotional peaks
4. **Transcribe**: Uses Whisper to transcribe only high-energy segments
5. **Score**: Combines audio energy and keyword matching to rank moments
6. **Cut**: Extracts highlight clips from the stream
7. **Render**: Creates vertical format with subtitles and branding
8. **Publish**: Uploads to TikTok automatically

## Quick Start

### Prerequisites

Before installing Python dependencies, ensure you have:

- **Python 3.10 or higher** (download from [python.org](https://www.python.org/downloads/))
- **FFmpeg** (CRITICAL - required for all video/audio processing)

#### Installing FFmpeg

**Windows:**
1. Download from [ffmpeg.org/download.html](https://ffmpeg.org/download.html)
2. Extract to a location (e.g., `C:\ffmpeg`)
3. Add the `bin` folder to your PATH environment variable (e.g., `C:\ffmpeg\bin`)
4. Verify: Open a new terminal and run `ffmpeg -version`

**macOS (using Homebrew):**
```bash
brew install ffmpeg
```

**Linux (Debian/Ubuntu):**
```bash
sudo apt update
sudo apt install ffmpeg
```

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-username/streamliner-ai.git
cd streamliner-ai

# 2. Create virtual environment
python -m venv venv

# Activate virtual environment
# Windows PowerShell:
.\venv\Scripts\Activate
# Linux/macOS:
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment
cp .env.template .env
cp config.yaml.example config.yaml
# Edit .env and config.yaml with your credentials

# 5. Test with a local video
python -m src.streamliner.cli process "data/video.mp4" --streamer "test" --dry-run

# 6. Start monitoring
python -m src.streamliner.cli monitor
```

## Configuration

### Kick API Setup

1. Create an application at [Kick Developer Portal](https://dev.kick.com)
2. Set Redirect URL to `http://localhost` (required but not used)
3. Select scopes: `channel:read`, `user:read`
4. Copy your `Client ID` and `Client Secret`
5. Add credentials to `.env`:

```env
KICK_CLIENT_ID="your_client_id"
KICK_CLIENT_SECRET="your_client_secret"
```

### TikTok API Setup

1. Register your application at [TikTok Developer Center](https://developers.tiktok.com/)
2. Configure a valid Redirect URI (e.g., `https://www.example.com/oauth`)
3. Add credentials to `.env`:

```env
TIKTOK_CLIENT_KEY=your_client_key
TIKTOK_CLIENT_SECRET=your_client_secret
TIKTOK_ENVIRONMENT=sandbox  # or 'production'
```

4. Generate initial tokens:

```bash
python scripts/generate_tiktok_tokens.py
```

Follow the prompts to authorize the application and paste the authorization code.

### Application Configuration

Edit `config.yaml` to customize:

- **streamers**: List of Kick usernames to monitor
- **detection**: Highlight detection parameters (thresholds, keywords, scoring weights)
- **transcription**: Whisper model settings (model size, device, compute type)
- **rendering**: Video rendering options (logo, subtitle style, fonts)
- **publishing**: TikTok upload strategy and description template

## Usage

### Monitor Mode (Production)

Continuously monitors configured streamers and processes highlights automatically:

```bash
python -m src.streamliner.cli monitor
```

Press `Ctrl+C` to stop gracefully.

### Process VOD (Testing)

Process a downloaded video file or URL:

```bash
# Process local file
python -m src.streamliner.cli process "path/to/video.mp4" --streamer "streamer_name" --dry-run

# Process from URL
python -m src.streamliner.cli process "https://kick.com/video/..." --streamer "streamer_name"
```

### Upload Clip

Upload a pre-rendered clip to TikTok:

```bash
python -m src.streamliner.cli upload \
    --file "data/clips/my_clip_rendered.mp4" \
    --streamer "test" \
    --strategy MULTIPART
```

### TikTok Diagnostics

Check sandbox state and backoff status:

```bash
# View current state
python -m src.streamliner.cli tiktok-diagnose

# Clear sandbox state
python -m src.streamliner.cli tiktok-clear-sandbox-state

# Upload with automatic backoff handling
python -m src.streamliner.cli upload-when-ready \
    --file "data/clips/my_clip.mp4" \
    --streamer "test" \
    --max-wait-seconds 2400
```

## Docker Deployment

Docker simplifies deployment by packaging the application with all dependencies including FFmpeg:

```bash
# Build the image
docker-compose build

# Start the service
docker-compose up -d

# View logs
docker-compose logs -f

# Stop the service
docker-compose down
```

## Development

### VS Code Setup

Create `.vscode/settings.json` for automatic formatting:

```json
{
    "python.defaultInterpreterPath": "${workspaceFolder}/venv/bin/python",
    "[python]": {
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.fixAll": true
        }
    }
}
```

### Code Quality

```bash
# Check for errors
ruff check .

# Format code
ruff format .
```

### Testing

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_detector.py

# Run with coverage
pytest --cov=src/streamliner
```

## Technical Deep Dive

### Async-First Architecture

The choice of `asyncio` enables handling multiple I/O operations (API calls, downloads, uploads) concurrently in a single thread, which is far more resource-efficient than traditional thread-based approaches.

- **Non-blocking Subprocesses**: Uses `asyncio.create_subprocess_exec` for FFmpeg and Streamlink, allowing the main program to continue while external processes run
- **Concurrent Monitoring**: Single process monitors multiple streamers simultaneously
- **Efficient Resource Usage**: Minimal CPU overhead during I/O-bound operations

### Optimized Detection

The decision not to transcribe the entire VOD is the system's most important optimization:

1. **RMS Energy Analysis**: Computationally cheap, acts as a high-speed filter
2. **Selective Transcription**: Only processes "interesting" audio segments with Whisper
3. **Keyword Scoring**: Combines audio energy with 200+ contextual keywords
4. **Scene Detection**: Bonus scoring for highlights coinciding with scene changes

This approach reduces hours of VOD to minutes of processing time.

### OAuth2 Implementation

The project evolved from initial attempts using unofficial endpoints (blocked by Cloudflare) to the official OAuth2 Client Credentials flow:

1. Requests an App Access Token from `https://id.kick.com/oauth/token`
2. Stores token in memory and refreshes automatically before expiration
3. Uses token for authenticated calls to `/public/v1/channels`

This approach is more stable, lightweight, and respectful of the platform.

## Future Improvements

- **Advanced Token Management**: Persist access tokens to Redis or database for state preservation across restarts
- **Real-time Processing Enhancement**: Redesign downloader to work with video chunks for near-instant clip creation
- **Metrics Dashboard**: Integrate Prometheus and Grafana for monitoring and visualization
- **Machine Learning Scoring**: Train advanced models analyzing chat velocity and game events
- **Multi-platform Support**: Abstract modules to support Twitch, YouTube, and other platforms

## License

This project is licensed under the MIT License. See the LICENSE file for details.

## Acknowledgments

This project represents an intensive development journey from initial concept to a robust, functional solution. The development process involved exploring different architectures and solving complex technical challenges, including bypassing anti-bot protections and ultimately implementing the official Kick API.

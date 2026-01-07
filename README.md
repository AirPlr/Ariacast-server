# AriaCast Server

**High-performance, low-latency PCM audio streaming over WebSockets.**

AriaCast Server is a lightweight, bidirectional audio streaming receiver written in Python. It transforms any device (Windows, Linux, macOS, Raspberry Pi) into a network "speaker" capable of receiving raw PCM audio, synchronizing metadata in real-time, and offering remote volume control.

Designed for stability and minimal overhead, it uses binary WebSockets for audio transport while maintaining fallback compatibility for standard HTTP players.

## Features

- **Hybrid Audio Transport**:
  - **WebSocket Stream**: Low-latency raw PCM transmission (48kHz/16-bit default).
  - **HTTP Stream**: `/stream.wav` endpoint for compatibility with generic players (VLC, Browser).
- **Bidirectional Control**:
  - Native system volume control (Windows `pycaw` support with PowerShell fallback).
  - Real-time metadata synchronization (Title, Artist, Artwork).
- **Adaptive Buffering**: Smart server-side buffering to handle network jitter.
- **Auto-Discovery**: Dual-stack discovery using Multicast DNS (Bonjour/ `_audiocast._tcp`) and custom UDP broadcast.
- **Cross-Platform**: Core logic in Python with platform-specific extensions.

## Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/yourusername/ariacast-server.git
   cd ariacast-server
   ```

2. **Install dependencies**:
   Ensure you have Python 3.9+ installed.
   ```bash
   pip install -r requirements.txt
   ```
   *Note: On Windows, some build tools might be required for `pycaw` or `sounddevice` depending on your environment. On Linux, you may need `libportaudio2`.*

## Usage

Start the server using the provided launcher script:

```bash
python start.py
```

### Command Line Options

```bash
# Run with default configuration
python start.py

# Run in verbose mode (debug logs)
python start.py -v

# Run with a specific configuration profile (see config_examples.py)
python start.py -c living_room

# Run in web-only mode (no local playback, just stream relay)
python start.py --web-only
```

---

# AriaCast Protocol Specification

## Overview

AriaCast is a protocol for streaming low-latency, high-quality audio over IP networks to distributed audio devices. The protocol is designed for simplicity, reliability, and minimal latency.

## Architecture

AriaCast uses a **client-server** architecture where:
- **Client**: Streams audio to the server, sends metadata, and controls playback
- **Server**: Receives audio, provides playback, and exposes control/metadata endpoints

### WebSocket Endpoints

| Endpoint | Type | Direction | Description |
|----------|------|-----------|-------------|
| `/audio` | Binary | Client → Server | PCM audio stream |
| `/stats` | JSON | Server → Client | Playback statistics |
| `/control` | JSON | Bidirectional | Volume and playback control |
| `/metadata` | JSON | Bidirectional | Track metadata (title, artist, artwork) |

## Discovery Phase

### Step 1: Discover Available Servers

Clients discover AriaCast servers using one of two methods:

#### Method A: mDNS (Primary, Recommended)

Browse for services of type `_audiocast._tcp` using mDNS/Bonjour:

```
Service Type: _audiocast._tcp.local.
```

**TXT Records** (advertising the server's capabilities):

| Key | Type | Example | Description |
|---|---|---|---|
| `name` | String | "Living Room Speaker" | Human-readable server name |
| `version` | String | "1.0" | Protocol version |
| `codecs` | String | "PCM" | Comma-separated supported codecs |
| `samplerate` | String | "48000" | Audio sample rate in Hz |
| `channels` | String | "2" | Number of audio channels |
| `platform` | String | "RaspberryPi" | Server platform identifier |

#### Method B: UDP Broadcast (Fallback)

If mDNS is unavailable or times out:

1. **Send Discovery Request**:
   - Destination: Broadcast address (e.g., `255.255.255.255`) or network broadcast
   - Port: `12888` (UDP)
   - Message: ASCII string `"DISCOVER_ARIACAST"`

2. **Receive Response**:
   - Source: Server IP address
   - Port: `12888` (UDP)
   - Message: JSON object (UTF-8 encoded)

### Step 2: Connect to Server

Once a server is discovered, establish connections to the WebSocket endpoints.

## Streaming Protocol

### Audio Streaming (`/audio` endpoint)

Audio data is sent as **binary WebSocket frames**:

| Component | Details |
|---|---|
| **Frame Type** | Binary (OpCode 0x2) |
| **Frame Size** | Exactly 3840 bytes |
| **Content** | Raw PCM audio data |
| **Timing** | Frames should be sent at 50 frames/second (20ms intervals) |

**Audio Data Format**:
```
Sample Rate:     48000 Hz
Bit Depth:       16-bit signed integer (little-endian)
Channels:        2 (Stereo)
Duration:        20 milliseconds
Frame Size:      960 samples × 2 channels × 2 bytes = 3840 bytes
```

### Control (`/control` endpoint)

The control endpoint allows bidirectional communication for volume and playback control.

**Volume Commands** (Client → Server):
```json
{"command": "volume", "direction": "up"}
{"command": "volume", "direction": "down"}
{"command": "volume_set", "level": 75}
```

### Metadata (`/metadata` endpoint)

The metadata endpoint allows clients to push track information and receive updates.

**Update Metadata** (Client → Server):
```json
{
  "type": "update",
  "data": {
    "title": "Song Title",
    "artist": "Artist Name",
    "album": "Album Name",
    "artwork_url": "https://example.com/cover.jpg",
    "is_playing": true
  }
}
```

## Network Topology

```
Local Area Network (192.168.1.0/24)
├─ AudioCast Server
│  ├─ mDNS Advertiser (_audiocast._tcp)
│  ├─ UDP Discovery Listener (port 12888)
│  └─ WebSocket Server (port 12889)
│
└─ Clients
   ├─ Discovery Phase (mDNS or UDP)
   └─ Streaming Phase (WebSockets)
```

---

## Web Dashboard

The server includes a built-in web dashboard and player accessible at:
`http://<server-ip>:8090/`

This allows you to listen to the stream directly from a browser or view server statistics.

## Requirements

- `aiohttp` - Async Web Server
- `sounddevice` - Audio Playback
- `zeroconf` - mDNS Discovery
- `numpy` - Audio Buffer Management
- `pycaw` - Windows Core Audio Library (Optional, for volume control)

## License

MIT License

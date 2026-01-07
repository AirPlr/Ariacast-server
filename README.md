# AriaCast

**Complete high-performance audio streaming solution for your local network.**

AriaCast is a comprehensive project that enables streaming audio from Android devices to custom receivers on your local network. It consists of two main components: an Android application for capturing and transmiting audio, and a lightweight Python server that acts as a receiver/speaker.

## Components

### 📱 AriaCast Android App

**Stream anything, anywhere.**

AriaCast is an Android application that lets you stream **any** audio playing on your device (Spotify, YouTube Music, Pocket Casts, etc.) to your custom server. It captures internal audio directly from the device and sends it to a designated receiver in real-time.

*   **Universal Audio Capture**: Works with any app that plays audio.
*   **Rich Metadata**: Automatically detects what's playing and syncs title, artist, album, and artwork to the receiver.
*   **Quick Settings Tile**: Start and stop casting instantly from your notification shade without opening the app.
*   **Auto-Discovery**: Automatically finds AriaCast receivers on your network.

### 🔊 AriaCast Protocol Server

**Turn any device into a Hi-Fi Wireless Speaker.**

The server component is a high-performance Python application designed to receive the stream from the AriaCast app. It transforms any computer (Windows, Linux, macOS, Raspberry Pi) into an endpoint speaker.

*   **Low Latency**: Uses binary WebSockets for efficient PCM audio transport.
*   **Robust Playback**: Features adaptive buffering to handle network jitter without skipping.
*   **Remote Control**: Allows the app to control the server's system volume explicitly.
*   **Hybrid Stream**: Also provides a standard HTTP WAV stream (`/stream.wav`) for compatibility with VLC or web browsers.

---

## 🚀 Getting Started (Server)

The code in this repository's root allows you to run the **AriaCast Server**.

### Prerequisites

- Python 3.9 or higher
- Network access (Port 8090 default)

### Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/yourusername/ariacast.git
   cd ariacast
   ```

2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```
   *Note: On Windows, `pycaw` (included) is used for native volume control. On Linux, `sounddevice` requires PortAudio (`sudo apt-get install libportaudio2`).*

### Usage

Start the server using the launcher script:

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

# Web-only mode (if you just want the stream endpoint without local playback)
python start.py --web-only
```

---

## 📡 Protocol Specification

AriaCast uses a custom protocol over WebSockets for minimum latency and maximum control.
For detailed information on message formats, endpoints, and the discovery process, please refer to the [Protocol Specification](PROTOCOL.md).

### Quick Protocol Overview

| Endpoint | Type | Description |
|----------|------|-------------|
| `/audio` | WebSocket | Accepts binary PCM frames (48kHz/16-bit by default). |
| `/control`| WebSocket | JSON-based volume and playback control. |
| `/metadata`| WebSocket | Pub/Sub for track information. |
| `/stats` | WebSocket | Real-time buffer and playback statistics. |
| `/listen` | WebSocket | For web-based audio listeners. |

## 🌐 Web Client

The server includes a built-in feedback player accessible at:
`http://<server-ip>:8090/`

This allows you to verify the stream or usage effectively as a "web speaker" if the host device doesn't have speakers attached.

## License

MIT License

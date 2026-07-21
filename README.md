# Embedded-Audio-Suite-Showcase
# 🎙️ Linux Embedded Audio Suite & Seamless MP3 Loop Engine

> A modular CLI toolkit for ALSA audio device configuration, system diagnostics, and sample-accurate seamless MP3 looping on embedded Linux environments.

---

## 🎯 Overview

This project provides a lightweight, terminal-based audio management suite designed for embedded Linux systems (Raspberry Pi, industrial PCs, interactive installations). 

It combines hardware diagnostic utilities with an **in-memory DSP pipeline** that eliminates clicks, pops, and phase gaps when looping MP3 files.

---

## 🛠️ Suite Features

The toolkit is orchestrated via `main.sh`, providing an interactive CLI menu divided into audio management tools and the seamless playback engine:

| Module | Feature | Description |
|---|---|---|
| **01** | **Device Selection** | Scans available ALSA soundcards and configures target output (`plughw:X,Y`). |
| **02** | **Audio Output Test** | Plays test tones across channels (Left / Right / Stereo) to verify wiring. |
| **03** | **Input Recording Test** | Records from micro/line-in and replays audio to validate input pipelines. |
| **04** | **Mixer & Volume Control** | Quick access to volume levels and gain adjustments. |
| **05** | **ALSA Diagnostics** | Displays active PCM streams, card hardware IDs, and system audio status. |
| **06** | **Config Management** | Saves device preferences persistently to `~/.alsa_config`. |
| **07** | **Seamless Loop Engine** | **(Core Feature)** RAM-buffered DSP engine for click-free MP3 playback. |

---

## 🧠 Core Feature: Seamless MP3 Loop Engine (Option 07)

Standard MP3 playback in loops usually produces audible "clicks" or brief silences at loop points due to encoder frame padding (1,152 sample blocks) and phase discontinuity.

To solve this without re-encoding source files, **Option 07** delegates processing to a dedicated engine (`player.sh`):

```text
[ Selected MP3 ] ──> [ FFmpeg PCM Decode ] ──> [ RAM Buffer (/dev/shm) ]
                                                        │
                                                        ▼
[ ALSA Hardware ] <── [ MPV Streamer ] <── [ Python 50ms Loop Crossfade ]

Key Technical Specs of the Engine
RAM-First Processing (/dev/shm): Eliminates disk I/O bottlenecks and latency on embedded storage.

Sample-Accurate Crossfade: A embedded Python DSP script blends the last 50ms of the audio track directly over the first 50ms in memory.

Phase Continuity: Ensures exact amplitude alignment at the loop boundary—preventing zero-crossing violations (speaker pops)

📂 Project Structure
├── main.sh             # Main CLI menu orchestrator & ALSA configuration
├── player.sh           # Standalone seamless audio loop engine (Option 07)
├── ~/.alsa_config      # Auto-generated runtime settings (saved device preferences)
└── README.md           # Documentation

🚀 Installation & Quick Start
1. Prerequisites
Install the required system utilities:

sudo apt update
sudo apt install -y ffmpeg mpv python3 alsa-utils sox

2. Setup
Clone the repository and set execution permissions:
git clone [https://github.com/](https://github.com/)<your-username>/linux-audio-suite.git
cd linux-audio-suite
chmod +x main.sh player.sh

3. Usage
Create an ambiance folder and add your MP3/WAV files:
mkdir -p ~/ambiances

Launch the main control interface:

Bash
./main.sh

🔬 In-Memory Crossfade Algorithm (Python DSP)
# Extract of the crossfade processing executed in /dev/shm
xfade_frames = int(framerate * 0.05)  # 50ms window
xfade_samples = xfade_frames * nchannels

# Linear crossfade blending tail into head
for i in range(xfade_samples):
    t = i / xfade_samples
    head_val = samples[i]
    tail_val = samples[len(samples) - xfade_samples + i]
    
    blended = int(head_val * t + tail_val * (1.0 - t))
    samples[i] = max(-32768, min(32767, blended))

# Trim merged tail for a seamless loop boundary
samples = samples[:-xfade_samples]
📄 License
Distributed under the MIT License. Free for personal and commercial embedded projects.


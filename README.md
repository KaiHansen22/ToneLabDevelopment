# ToneLab

> **Spectrally Controlled Parallel Multi-FX Plugin**  
> Built with JUCE · C++ · Real-Time DSP

---

## Overview

ToneLab is a VST3/AU audio effects plugin that gives producers and sound designers surgical control over where and how effects are applied — by frequency band. Rather than processing a signal as a whole, ToneLab splits the incoming audio into spectral regions and routes each band through its own independent effects chain, then recombines them in a parallel architecture. The result is a level of tonal precision not achievable with traditional serial FX chains.

Whether sculpting a mix bus, designing textured pads, or creating frequency-dependent saturation that only affects the low-mids — ToneLab puts the spectrum itself in your hands.

---

## Key Features

- **Spectral Band Splitting** — Divide incoming audio into up to N independent frequency bands using crossover filters
- **Per-Band FX Routing** — Each band hosts its own effects chain, processed independently in parallel
- **Real-Time Parameter Control** — All parameters are fully automatable and host-syncable with zero-latency UI feedback
- **Parallel Mix Architecture** — Wet/dry blending at both the band level and master output level
- **Low-Latency DSP Core** — Optimized C++ signal processing pipeline designed for live and studio use
- **JUCE Framework** — Cross-platform VST3 and AU support (macOS / Windows)

---

## Technical Architecture

### Signal Flow

```
Input Signal
     │
     ├──► Band 1 (Low)     → [FX Chain A] ─┐
     ├──► Band 2 (Mid-Low) → [FX Chain B] ─┤
     ├──► Band 3 (Mid-High)→ [FX Chain C] ─┼──► Parallel Mixer ──► Output
     └──► Band 4 (High)    → [FX Chain D] ─┘
```

### Core Components

| Module | Description |
|---|---|
| `SpectrumSplitter` | Linkwitz-Riley crossover filter bank for phase-coherent band splitting |
| `BandProcessor` | Per-band DSP chain manager with effect slot allocation |
| `ParallelMixer` | Weighted recombination of processed bands with master gain |
| `ParameterManager` | APVTS-based parameter tree with full DAW automation support |
| `PluginEditor` | Custom JUCE component UI with real-time spectrum visualization |

### DSP Design Decisions

**Why Linkwitz-Riley crossovers?**  
LR4 filters (fourth-order Butterworth pairs) sum to a flat magnitude response at the crossover point — essential for a parallel architecture where recombined bands must not comb-filter or color the signal unless intentionally directed to.

**Why APVTS (AudioProcessorValueTreeState)?**  
JUCE's APVTS provides thread-safe parameter access between the audio and message threads, serialization for save/recall, and native DAW automation mapping — all critical for a production plugin.

**Real-time constraints**  
All heap allocation occurs during plugin initialization. The `processBlock()` path is entirely allocation-free, using pre-allocated audio buffers per band to avoid priority inversion on the audio thread.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | C++17 |
| Framework | JUCE 7 |
| Plugin Formats | VST3, AU |
| Build System | CMake |
| Platforms | macOS (ARM + Intel), Windows 10+ |

---

## Build Instructions

### Prerequisites

- JUCE 7.x ([download](https://juce.com/get-juce/))
- CMake 3.22+
- Xcode 14+ (macOS) or Visual Studio 2022 (Windows)

### Clone & Build

```bash
git clone https://github.com/YOUR_USERNAME/ToneLab.git
cd ToneLab

# Generate project files
cmake -B build -DCMAKE_BUILD_TYPE=Release

# Build
cmake --build build --config Release
```

Plugin binaries will be output to `build/ToneLab_artefacts/Release/`.

### Install

Copy the built `.vst3` or `.component` to your system plugin directory:

- **macOS VST3:** `~/Library/Audio/Plug-Ins/VST3/`
- **macOS AU:** `~/Library/Audio/Plug-Ins/Components/`
- **Windows VST3:** `C:\Program Files\Common Files\VST3\`

---

## Project Structure

```
ToneLab/
├── Source/
│   ├── PluginProcessor.h / .cpp     # Core AudioProcessor, processBlock
│   ├── PluginEditor.h / .cpp        # JUCE UI component tree
│   ├── SpectrumSplitter.h / .cpp    # Crossover filter bank
│   ├── BandProcessor.h / .cpp       # Per-band FX chain
│   └── ParallelMixer.h / .cpp       # Band recombination
├── CMakeLists.txt
└── README.md
```

---

## Development Highlights

This project demonstrates applied skills in:

- **Real-time audio programming** — thread-safe, allocation-free audio processing adhering to real-time constraints
- **DSP algorithm implementation** — custom filter design, spectral decomposition, parallel signal routing
- **JUCE plugin architecture** — full VST3/AU plugin lifecycle, APVTS parameter management, editor/processor separation
- **C++ systems design** — efficient buffer management, modular component architecture, RAII resource handling

---

## Process Documentation

Full development process, design decisions, and iteration log available at:  
**[ToneLab Development Log →](https://YOUR_USERNAME.github.io/ToneLab/process)**

---

## About

Developed as a capstone project for ATLAS 4010 at the University of Colorado Boulder.  
Exploring the intersection of signal processing, interface design, and creative toolmaking.

**[View Project Page →](https://YOUR_USERNAME.github.io/ToneLab)**

---

*Built with JUCE. Designed for precision.*

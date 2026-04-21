# ToneLab

> **Spectrally Controlled Parallel Multi-FX Plugin**  
> Built with JUCE · C++ · Real-Time DSP

---

## Overview

ToneLab is a VST3/AU audio effects plugin built around a spectral send architecture. Rather than applying effects to the full audio signal, each of the five effect lanes has its own multi-band EQ shaping layer — `SendMask5` — that defines which frequencies are routed into that lane's effect engine. Each lane processes its shaped signal independently and all five are recombined in parallel into the output.

The result is frequency-specific effect placement without external routing: distortion that only saturates the low-mids, reverb that lives only in the high-mids, delay feedback that sits in the presence band.

---

## Architecture

### Signal Flow

```
Input Signal
     │
     ├── Input Gain
     │
     ├──► Lane 1: Chorus     [SendMask5 EQ → MicroPitch engine]  ─┐
     ├──► Lane 2: Delay      [SendMask5 EQ → Delay line]          ├──► Wet Bus
     ├──► Lane 3: Saturation [SendMask5 EQ → Waveshaper]          │
     ├──► Lane 4: Reverb     [SendMask5 EQ → Algo/IR Reverb]      │
     └──► Lane 5: Distortion [SendMask5 EQ → Distortion/Crush]   ─┘
                                                                    │
     Dry ──────────────────────────────────────────────────────────┤
                                                          Equal-power crossfade
                                                                    │
                                                               Output Gain
```

### SendMask5 — Per-Lane Spectral Shaping

Each lane's send is shaped by `SendMask5`: a five-band EQ with cascaded IIR filters (3-cascade per band) and a notch filter per band. Each band supports multiple filter types (peak, low shelf, high shelf, band-pass, high-pass, low-pass, notch) and is independently enabled/disabled. The shaped signal is what feeds the lane's effect engine — the raw dry signal is never sent directly.

---

## Effect Engines

### Chorus
Multi-voice micro-pitch chorus implemented as a custom `MicroPitch` class. Uses up to 6 voices, each with an independent modulated delay line (Linear interpolation). Voice phases are distributed evenly across the LFO cycle. LFO uses a fast polynomial sine approximation. Output is normalized by voice count and soft-clipped stereo-linked above 0 dBFS.

Parameters: Rate (Hz), Depth (cents), Voices (1–6)

### Delay
Stereo delay with three modes:

- **Digital** — clean, transparent repeats
- **Tape** — 3 kHz LPF on feedback path + subtle wow/flutter (±1% pitch modulation at 0.5 Hz via LFO on delay time)
- **Analog** — 6 kHz LPF on feedback path, softer than digital

Supports BPM sync (reads host playhead position), ping-pong mode with variable stereo width (M/S), feedback pan control, and up to ~4 seconds of delay (192,000 sample delay line at 48 kHz).

Parameters: Time (ms), Feedback, Mode, BPM Sync, Beat Division, Ping-Pong, Width, Feedback Pan

### Saturation
Three waveshaper modes:

- **Soft Clip** — `x / (1 + |x|)`, symmetric, gentle even-order harmonics
- **Diode** — asymmetric half-wave blend, harsh odd/even harmonic character
- **Tape** — bias-shifted soft saturation with warm harmonic envelope

Post-saturation low-pass tone filter (500 Hz–20 kHz). Drive parameter scales pre-gain before waveshaping.

Parameters: Drive, Tone (Hz), Mode

### Reverb
Dual-engine reverb with runtime level matching between paths:

- **Algorithmic** — `juce::dsp::Reverb` with five room mode presets: Hall, Plate, Room, Spring, Chamber. Each mode shapes size, damping, and stereo width differently.
- **IR Convolution** — user-loadable impulse response via `juce::dsp::Convolution`. IR is normalized to peak 0 dBFS on load. A smoothed EMA of both paths' RMS output is maintained, and `irOutputGain` converges toward the ratio at runtime for perceptual level-matching between IR and algorithmic modes.

Pre-delay supported on both paths (up to 200 ms). A low-pass IR damping filter shapes the pre-convolution signal.

Parameters: Size, Damping, Pre-Delay, Decay, Mode (Hall/Plate/Room/Spring/Chamber), Use IR

### Distortion
Hard clipping distortion with two additional processing stages:

- Pre high-shelf filter adds grit to the input before clipping
- Post presence peak (tone filter) shapes the output
- M/S mid-channel tone shaping

Bit crusher (`bitCrush`) quantizes the signal to 2–32 bits, applied post-clip.

Parameters: Drive, Tone (Hz), Crush (bits)

---

## Key Implementation Details

**Silence gate** — A block-level RMS gate (`kSilenceThresholdRMS = 1e-6f`) skips all lane processing after 50 consecutive silent blocks, preventing unnecessary computation and avoiding reverb tail artifacts on silence.

**Equal-power wet/dry crossfade** — The global mix control uses a quarter-cosine crossfade (`cos(angle)` dry, `sin(angle)` wet) rather than a linear blend, keeping the combined output level consistent and preventing stereo image narrowing at intermediate mix positions.

**RTA** — A 2048-point (2^11) stereo FIFO feeds a lock-free block handoff to the editor's spectrum display. The audio thread writes via `rtaBlockReady` atomic; the editor reads via `pullRtaBlockStereo()`.

**Peak metering** — Input and output peak levels are written by the audio thread via `std::atomic<float>` and polled by the editor on a timer, with no locks on the audio path.

**Parameter caching** — All APVTS parameter values are accessed via pre-cached `std::atomic<float>*` pointers (`cacheParamPointers()`), loaded once per `processBlock()` call. Last-value caches on filter coefficients (tone, reverb, delay mode) skip coefficient recalculation when parameters have not changed.

**Session persistence** — IR file path and editor UI state are serialized as XML attributes alongside the APVTS state tree in `getStateInformation()` / `setStateInformation()`.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | C++17 |
| Framework | JUCE 7 |
| Build | Projucer → Xcode |
| Plugin Formats | VST3, AU |
| Platforms | macOS (Intel), Windows |

---

## Dev Environment

Primary development on a MacBook Pro 15-inch 2019, 2.4 GHz 8-Core Intel Core i9, 32 GB DDR4, macOS Sequoia 15.7.4. Project configured in Projucer and built through Xcode. Tested in Logic Pro as the primary host. A Windows VST3 build was produced later using the same Projucer project targeting Visual Studio.

---

## Links

- **Project page:** [kaihansen22.github.io/ToneLabDevelopment](https://kaihansen22.github.io/ToneLabDevelopment)
- **Product page:** [vector-dsp.com/tonelab](https://vector-dsp.com/tonelab)
- **Source (private):** [github.com/KaiHansen22/ToneLab-src](https://github.com/KaiHansen22/ToneLab-src)

---

*Built with JUCE. Developed for ATLAS 4010 at the University of Colorado Boulder.*

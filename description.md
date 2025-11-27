# Infinite Jazz Generator: Concept & Specification

## 1. Project Overview
The **Infinite Jazz Generator** is a browser-based algorithmic music engine. It generates an endless stream of jazz improvisation over a fixed chord progression. It utilizes **Web Audio API** (via Tone.js) for high-precision timing and audio synthesis, ensuring that the swing feel and groove remain consistent indefinitely.

The system is designed to simulate a "trio" setting:
1.  **Lead Voice:** Improvises melodies (FM Synth with vibraphone-like bell tones).
2.  **Comping:** Plays rhythmic background chords (Rhodes-like FM Synth with drop-2 voicings).
3.  **Bass:** Plays walking bass lines with chromatic approach notes (MonoSynth through lowpass filter).

---

## 2. Musical Configuration

### Global Settings (Configurable via UI)
*   **Tempo:** 70–160 BPM (default: 110).
*   **Time Signature:** 4/4.
*   **Groove/Feel:** Swing 0–50% (default: 30%, applied via Tone.js Transport).
*   **Key Center:** C Major (with harmonic deviations for the VI chord).

### Harmonic Progression (The "Turnaround")
The generator loops a standard 4-bar Jazz Turnaround (ii–V–I–VI).

| Bar | Chord | Type | Root MIDI | Scale Source | Notes |
| :-- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Dm7** | Minor 7 | D (62) | C Major | D, F, A, C |
| **2** | **G7** | Dominant 7 | G (55) | C Major | G, B, D, F |
| **3** | **Cmaj7** | Major 7 | C (60) | C Major | C, E, G, B |
| **4** | **A7** | Dominant 7 | A (57) | **A Phrygian Dom** | A, C#, E, G |

> **Note on A7:** While the key center is C Major, the A7 chord acts as a secondary dominant pointing back to Dm7. To avoid harmonic clashes, the scale changes for this bar to include **C#** (Critical Harmonic Fix) and **Bb** (implied b9).

### Chord Voicings
Chords use **Drop-2 voicing** for professional, open sound:
- Root in bass register
- 3rd dropped down an octave
- 5th and 7th remain in original position

---

## 3. Algorithmic Parameters

### Comping Rhythm Patterns
The comping instrument plays chords using configurable rhythm patterns:

| Style | Description | Character |
| :--- | :--- | :--- |
| **Whole Notes** | Sustained pad | Ambient, legato |
| **Charleston** | Beat 1 + "and" of 2 | Classic jazz standard |
| **Bossa Nova** | Anticipated syncopation | Latin, flowing |
| **Driving Quarters** | Four-to-the-floor | Energetic, propulsive |
| **Sparse** | Minimal accents | Space, breathing room |
| **Busy Bebop** | Rapid chord stabs | Active, complex |

### Melody Rhythm & Phrasing
Melodies are constructed using a library of **Rhythm Patterns** grouped by density. Each pattern consists of 8 slots (eighth notes).

| Density | Character | Note Count |
| :--- | :--- | :--- |
| **Sparse** | Open, breathing | 2–3 notes/bar |
| **Moderate** | Balanced phrasing | 4–5 notes/bar |
| **Busy** | Bebop runs | 6–7 notes/bar |

### Walking Bass
The bass plays a **4-beat walking pattern** per bar:
*   **Beat 1:** Root note
*   **Beat 2:** 3rd or 5th (randomized)
*   **Beat 3:** 5th
*   **Beat 4:** Chromatic approach to next chord's root (±1 semitone)

### Pitch Selection Probabilities
When a note event is triggered, the pitch is determined by a weighted decision tree:

*   **Strong Beats (1 & 3):**
    *   High probability (70%) of landing on a **Chord Tone**.
    *   Specific bias towards **Guide Tones** (3rds and 7ths) to define the harmony.
*   **Weak Beats (2 & 4 & "ands"):**
    *   Allowed to use **Scale Tones** (passing notes).
    *   Low probability (10%) of **Chromatic Approach** notes (semitone neighbors).
*   **Interval Logic:**
    *   **Stepwise:** 85% probability (move to nearest scale degree).
    *   **Leaps:** 15% probability (jumps of a 4th or 5th) to break monotony.
    *   **Register Lock:** Notes are forced to stay within a playable MIDI range (60–80).

---

## 4. Audio Engine

### Signal Routing
```
Lead Synth ──┬── Dry Gain (0.7) ──────────────────┬── Destination
             └── Delay (8n. dotted) ── Reverb ────┘

Chord Synth ──── Reverb ── Destination

Bass Synth ──── Lowpass Filter (800Hz) ── Destination (DRY)
```

### Instrument Definitions

| Voice | Synth Type | Characteristics |
| :--- | :--- | :--- |
| **Lead** | PolySynth (FMSynth) | Harmonicity 3.01, bell-like vibraphone tone |
| **Chords** | PolySynth (FMSynth) | Harmonicity 2, warm Rhodes-like tone |
| **Bass** | MonoSynth | Triangle wave, 24dB lowpass filter, no reverb |

### Effects
*   **Reverb:** Decay 2.5s, wet 35% (configurable 0–80%)
*   **Delay:** Dotted eighth note, 15% feedback, 12% wet

---

## 5. User Interface Controls

All parameters update in **real-time** while playing:

| Control | Range | Default | Description |
| :--- | :--- | :--- | :--- |
| **Tempo** | 70–160 BPM | 110 | Playback speed |
| **Swing** | 0–50% | 30% | Eighth-note swing amount |
| **Comping Style** | 6 options | Charleston | Chord rhythm pattern |
| **Melody Density** | 3 options | Moderate | Note frequency in lead |
| **Lead Volume** | 0–100% | 70% | Lead synth level |
| **Bass Volume** | 0–100% | 65% | Bass synth level |
| **Reverb Mix** | 0–80% | 35% | Wet/dry reverb blend |

---

## 6. Assumptions & Constraints

1.  **Infinite Loop:** The chord progression never changes or modulates. The variety comes entirely from the melodic improvisation and rhythm variations.
2.  **Voice Leading:** The algorithm assumes that "good" jazz sounds smooth. It calculates the nearest neighbor in the next scale rather than picking random notes.
3.  **No Micro-Timing:** While the audio engine applies a global "swing," individual notes are quantized to the grid. There is no humanization of start-times (rubato), only velocity.
4.  **Register:** The melody instrument is assumed to be monophonic (one note at a time).
5.  **Lookahead:** The system uses Tone.js scheduling to ensure audio does not glitch, meaning changes to parameters might have a slight delay before being audible.
6.  **Bass Separation:** Bass is kept completely dry (no reverb) to maintain tight low-end definition.

---

## 7. Technical Stack

*   **HTML5/CSS3:** UI and Layout (modern dark theme with gradient background).
*   **JavaScript (ES6):** Logic and Control.
*   **Tone.js:**
    *   Wrapper for Web Audio API.
    *   Handles precise scheduling (`Tone.Transport`).
    *   Provides FM Synths and MonoSynths for authentic jazz tones.
    *   Effects chain: Reverb, FeedbackDelay, Filter, Gain.
    *   Manages the Global Swing logic.

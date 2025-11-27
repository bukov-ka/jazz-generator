# Infinite Jazz Generator: Concept & Specification

## 1. Project Overview
The **Infinite Jazz Generator** is a browser-based algorithmic music engine. It generates an endless stream of jazz improvisation over a fixed chord progression. It utilizes **Web Audio API** (via Tone.js) for high-precision timing and audio synthesis, ensuring that the swing feel and groove remain consistent indefinitely.

The system is designed to simulate a "trio" setting:
1.  **Lead Voice:** Improvises melodies (Vibraphone/Electric Piano tone).
2.  **Comping:** Plays background chords (Soft Pad).
3.  **Bass:** Plays root notes to ground the harmony (Upright Bass emulation).

---

## 2. Musical Configuration

### Global Settings
*   **Tempo:** 110 BPM (Configurable).
*   **Time Signature:** 4/4.
*   **Groove/Feel:** Swing (30% swing factor applied via Tone.js Transport).
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

---

## 3. Algorithmic Parameters

### Rhythm & Phrasing
Melodies are constructed using a library of **Rhythm Patterns**. Each pattern consists of 8 slots (eighth notes).
*   **Resolution:** 1/8th note.
*   **Pattern Types:**
    *   *Walking:* Continuous notes.
    *   *Syncopated:* Emphasis on off-beats.
    *   *Sparse:* High density of rests (breathing room).
    *   *Busy:* Rapid runs.

### Pitch Selection Probabilities
When a note event is triggered, the pitch is determined by a weighted decision tree:

*   **Strong Beats (1 & 3):**
    *   High probability (70%) of landing on a **Chord Tone**.
    *   Specific bias towards **Guide Tones** (3rds and 7ths) to define the harmony.
*   **Weak Beats (2 & 4 & "ands"):**
    *   Allowed to use **Scale Tones** (passing notes).
    *   Low probability (10%) of **Chromatic Approach** notes (semitone neighbors).
*   **Interval Logic:**
    *   **Stepwise:** 70% probability (move +/- 1 or 2 scale degrees).
    *   **Leaps:** 30% probability (jumps of a 4th or 5th) to break monotony.
    *   **Register Lock:** Notes are forced to stay within a playable MIDI range (60–80).

---

## 4. Assumptions & Constraints

1.  **Infinite Loop:** The chord progression never changes or modulates. The variety comes entirely from the melodic improvisation.
2.  **Voice Leading:** The algorithm assumes that "good" jazz sounds smooth. It calculates the nearest neighbor in the next scale rather than picking random notes.
3.  **No Micro-Timing:** While the audio engine applies a global "swing," individual notes are quantized to the grid. There is no humanization of start-times (rubato), only velocity.
4.  **Register:** The melody instrument is assumed to be monophonic (one note at a time).
5.  **Lookahead:** The system uses a scheduling lookahead of ~1.0 seconds to ensure audio does not glitch, meaning changes to parameters might have a slight delay before being audible.

---

## 5. Technical Stack

*   **HTML5/CSS3:** UI and Layout.
*   **JavaScript (ES6):** Logic and Control.
*   **Tone.js:**
    *   Wrapper for Web Audio API.
    *   Handles precise scheduling (`Tone.Transport`).
    *   Provides PolySynths and Effects (Reverb, Delay).
    *   Manages the Global Swing logic.
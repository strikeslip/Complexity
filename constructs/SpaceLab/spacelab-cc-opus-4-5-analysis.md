This detailed analysis covers the **SPACELAB** seismic wave synthesizer, a sophisticated Web Audio API application that performs real-time sonification of tectonic data.

---

### 1. Conceptual Overview
**SPACELAB** is a "Seismic Sonification Engine." It transforms raw geological data into a musical composition. It uses a **generative approach** where the earthquake's magnitude dictates the tempo, and the actual physical displacement of the earth (seismic waveforms) modulates synthesizer parameters like filter cutoffs, resonance, and oscillator detuning.

The aesthetic is explicitly "Kraftwerk-inspired," utilizing clean subtractive synthesis, robotic rhythms, and a mathematical approach to melody (D Dorian scale).

---

### 2. Data Acquisition & Processing
The script implements a dual-stage data pipeline:

*   **USGS GeoJSON API:** The script first queries the USGS for the most recent **Magnitude 6.0+** earthquake. It extracts the magnitude, depth, location, and timestamp.
*   **EarthScope MiniSEED API:** Using the timestamp from the USGS, it fetches 60 seconds of raw binary seismic data (MiniSEED format) from a specific station (`IU.ANMO`).
*   **Parsing & Normalization:**
    *   `parseMiniSEED`: Manually decodes the binary buffer into 32-bit integers.
    *   `normalize`: Scales the seismic peaks to a range of `-1.0 to 1.0`.
    *   `resample`: Stretches the 1Hz–40Hz seismic data into an 8-second audio-rate buffer (44.1kHz), making the "inaudible" earth movements audible through time-compression.

---

### 3. The Audio Architecture
The Web Audio API graph is structured for professional signal flow:

1.  **Input Bus (`busIn`):** Sums all synthesizers.
2.  **DC Offset Filter (`dc`):** A High-Pass filter at 20Hz to remove sub-sonic rumble that could damage speakers or clip the output.
3.  **Dynamics Compressor (`limiter`):** Acts as a master limiter (Threshold -8dB) to prevent digital clipping when multiple instruments layer.
4.  **Ducking Gain (`duck`):** A side-chain-style gain stage that lowers the volume of the music slightly when the raw seismic "thump" is played.

---

### 4. Synthesis Engine (The "Instruments")
Each instrument is a custom function utilizing `OscillatorNodes` and `GainNodes`:

*   **Bass Drum (BD):** Uses a sine wave with a rapid exponential pitch drop. The `seisVal` (seismic magnitude) modulates the start frequency, making larger quakes have "harder" kicks.
*   **Resonant Bass:** A combination of Sawtooth and Square waves. The seismic data drives the **Filter Q (Resonance)**, creating a "squelchy" acid-bass sound that reacts to the earth's movement.
*   **Poly Pad:** A lush square-wave pad. Seismic data modulates **detuning**, creating a "shimmer" or "instability" effect corresponding to seismic activity.
*   **Solo Lead:** A classic sawtooth lead. Seismic data modulates the **brightness** (filter frequency).

---

### 5. Generative Composition Logic
The script functions like a DAW (Digital Audio Workstation) within the browser:

*   **Tempo Mapping:** The BPM is dynamic: `tempo = 110 + (mag - 6) * 8`. A Magnitude 6 quake plays at 110 BPM, while a Magnitude 9 would play at 134 BPM.
*   **Scale Quantization:** All melodies are constrained to **D Dorian** (`[0, 2, 3, 5, 7, 8, 10]`). This ensures that no matter how chaotic the seismic data, the result remains harmonic and "Kraftwerkian."
*   **Song Structure:** The `songStructure` array defines a professional arrangement (Intro -> Chorus -> Verse -> Break -> Vamp). Each section calls different patterns, creating a sense of progression rather than a simple loop.

---

### 6. Seismic Modulation (The "Soul" of the Script)
The most advanced feature is the real-time modulation system:
```javascript
function getSeismicValue() {
  seismicIndex = (seismicIndex + 1) % seismicSamples.length;
  return seismicSamples[seismicIndex];
}
```
This function "scans" the seismic waveform while the music plays. 
*   If the earth was moving violently at a specific millisecond of the quake, the **Filter Q** will spike, or the **Hi-Hat decay** will lengthen. 
*   This creates a literal "duet" between the JavaScript sequencer and the tectonic plates.

---

### 7. Technical Strengths
1.  **Memory Management:** It uses `activeNodes.push()` to track oscillators and stops them properly to prevent memory leaks and "hanging" notes.
2.  **Keep-Alive Hack:** It uses a `ConstantSourceNode` with a tiny offset (`1e-5`) connected to the output. This prevents Chrome from putting the AudioContext to sleep during quiet sections.
3.  **UI/UX:** It provides a clean, dark-mode interface with direct links to the USGS event page for scientific verification.

---

### 8. Potential Enhancements
*   **Visualizer:** Adding a `<canvas>` element to visualize the `seismicSamples` would provide a visual link to the sound.
*   **Station Selection:** Currently, it only pulls from station `ANMO` (Albuquerque, New Mexico). It could be modified to pull from the station closest to the earthquake's epicenter.
*   **Reverb/Delay:** Adding a `ConvolverNode` (reverb) would enhance the "Spacelab" space-age atmosphere.

### Summary
The **SPACELAB** script is a masterclass in combining **Earth Science** with **Algorithmic Music**. It turns a terrifying natural event into a structured, avant-garde musical experience, effectively using the Web Audio API as a bridge between the physical world and digital synthesis.

<i> Claude Opus 4.5 Analysis Jan 5, 2026

# SOS — ShadowZone 
### Seismic Waveform Electronica & Acoustic Tomography

> **"The Earth is not silent; it is a low-frequency oscillator of planetary proportions. SOS: ShadowZone is the interface through which we eavesdrop on the lithosphere."**

---

## 1. Executive Summary
**SOS (Sounds Of Seismic) - ShadowZone Edition** is a browser-based generative synthesizer and visualizer that sonifies real-time global seismic activity. Utilizing three-layer granular synthesis, it transforms the raw displacement data of the Earth's crust into a harmonic soundscape. 

The "ShadowZone" edition specifically focuses on the internal geometry of the planet, visualizing how P-waves refract through the Core-Mantle Boundary (CMB), creating silence in specific angular distances (103°–142°) while generating complex "textures" of sound elsewhere.

---

## 2. Technical Architecture

### 2.1 Data Acquisition (USGS & IRIS)
The engine utilizes the **FDSN (International Federation of Digital Seismograph Networks)** web services. 
1.  **Event Discovery:** Queries the USGS GeoJSON API for events exceeding Magnitude 6.0 within the last 30 days.
2.  **Waveform Fetch:** Requests MiniSEED data from the IRIS (Incorporated Research Institutions for Seismology) `dataselect` service, specifically targeting the **IU.ANMO** (Albuquerque, New Mexico) station.
3.  **Normalization:** Raw 32-bit integer seismic counts are normalized to a Float32 range [-1.0, 1.0] to serve as the synthesis grain source.

### 2.2 The Triple-Layer Granular Engine
Unlike standard playback, SOS uses **Seismic-Driven Granulation**. The waveform acts as both the *timbral source* and the *modulator*.

| Layer | Musical Role | Logic | Technical Characteristic |
| :--- | :--- | :--- | :--- |
| **Texture** | Shimmer/Highs | Dense, short grains (35ms). Density modulated by seismic amplitude. | High-pass filtered, wide stereo spread. |
| **Tone** | Harmonic Base | Sparse, long grains (300ms). Quantized to selected musical scales. | Fundamental frequency = Root; Saw/Tri mix. |
| **Rhythm** | Percussive Pulse | Triggered by **Zero-Crossing Detection** of the seismic wave. | High Q-factor filters, transient-heavy. |

### 2.3 Modulation Matrix
The engine calculates two real-time descriptors from the seismic stream:
*   **RMS (Root Mean Square):** Controls global amplitude and filter cutoff (the "energy" of the quake).
*   **First Derivative ($\Delta y/\Delta x$):** Measures the rate of change in ground motion, mapped to grain frequency jitter and "Texture" density.

---

## 3. Scientific Visualization: The Shadow Zone
The visualizer implements a real-time **P-wave Ray Tracing** algorithm on a 2D HTML5 Canvas.

*   **Refraction Physics:** Simulates wave paths from the epicenter through the Mantle and Outer Core.
*   **The Discontinuity:** Visualizes the "Shadow Zone," where the refractive index change at the Core-Mantle Boundary prevents direct P-wave arrival between 103° and 142° from the epicenter.
*   **Depth Weighting:** The depth of the earthquake (in km) is mapped to the **Reverb Decay Time** and **Delay Feedback**, simulating the "acoustic volume" of the subduction zone or fault line.

---

## 4. Conceptual & Philosophical Analysis

### 4.1 The Earth as an Oscillator
SOS treats the Earth as a monumental musical instrument. By shifting the sub-audible frequencies of seismic waves (typically 0.01Hz to 20Hz) into the human audible spectrum through granular transposition, the user experiences **Acoustic Tomography**. We are not just hearing noise; we are hearing the *density and elasticity* of the planet's interior.

### 4.2 The Aesthetics of the Shadow Zone
In seismology, the Shadow Zone is a region of "missing information." Philosophically, this edition of SOS explores the **presence of absence**. In the software, the Shadow Zone is represented by a darker visual void, yet in the audio, this is where the "Texture" layer often becomes most ghost-like—representing the scattered waves (PKP phases) that manage to bypass the core's occlusion.

### 4.3 "Muzak" (Frozen Time)
The "Muzak" (Freeze) function represents a departure from linear time. It traps the granular engine in a specific micro-moment of a tectonic shift. It transforms a catastrophic, high-energy event into a static, ambient meditation—a "tectonic wallpaper" that forces the listener to confront the scale of planetary movement at a human, domestic pace.

---

## 5. User Interface & Experience (UX)
The UI is designed with a "Deep-Time" aesthetic:
*   **Cybernetic Brutalism:** High-contrast, monospace typography reminiscent of early seismic monitoring stations.
*   **Dynamic Metrics:** Real-time feedback of grain counts and filter frequencies provides a sense of "scientific observation."
*   **Scale Quantization:** By allowing users to force the Earth's "chaos" into Pentatonic or Dorian scales, the app explores the tension between **Natural Entropy** and **Human Order**.

---

## 6. Project Metadata
*   **Version:** SHADOWZONE Edition (2025)
*   **Author:** SHOOK aka D.V.R.
*   **License:** MIT
*   **Technologies:** JavaScript (ES6+), Web Audio API, HTML5 Canvas, FDSN/IRIS Web Services.
*   **Deployment:** [https://sos.allshookup.org/](https://sos.allshookup.org/)

---

> *"Listening to a Magnitude 7.8 quake via SOS is not just an auditory experience; it is a realization of our precarious position atop a shifting, vibrating, and living sphere of liquid metal and stone."*

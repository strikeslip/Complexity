# 🌍 SOS: ALGORITHMIC IMPLEMENTATION SCHEMA

## 1. SYSTEM ARCHITECTURE OVERVIEW

The system operates as a **Hybrid Musification Engine**. It does not merely play data (Audification); it interprets data complexity to generate musical complexity.

**Core Logic:** `Geological Complexity ≈ Musical Complexity`
**Fitness Function:** Normalized Information Distance (NCD) & Kolmogorov Complexity ($K(x)$).

---

## 2. PHASE I: DATA INGESTION & ANALYSIS ("The Ear")

**Input:** Real-time MiniSeed streams (Channels: HHZ, HHE, HNN).

| Analysis Module | Target Metric | Method/Algorithm |
| :--- | :--- | :--- |
| **Complexity Analyzer** | **Kolmogorov Complexity ($K(x)$)** | Lempel-Ziv compression on 5s sliding windows. High compression ratio = Low $K(x)$; Low compression = High $K(x)$. |
| **Event Detector** | **Phase Identification** | STA/LTA (Short Term Average/Long Term Average) algorithms to distinguish P-waves (primary) from S-waves (secondary). |
| **Signal Profiler** | **Signal-to-Noise Ratio (SNR)** | Calculates variance to distinguish "Quiet Earth" (Ambient) from "Active Event" (Rhythmic/Chaotic). |
| **Spectral Scanner** | **Infrasonic Content** | FFT analysis focusing on 0.01 Hz – 20 Hz range for Trans-Sonic mapping. |

---

## 3. PHASE II: THE COMPOSITIONAL BRAIN ("The Composer")

This layer determines *what* happens based on the data analysis. It uses a hybrid of **Stochastic (Xenakis)** and **Generative (Eno)** logic.

### A. State Selection (The Macro Structure)
 The system selects the musical "Mode" based on the **$K(x)$ Score**.

*   **State 1: The Minimalist (Low $K(x)$)**
    *   *Data Condition:* Repetitive tremors, background noise, high compressibility.
    *   *Musical Output:* **Eno Generative Systems.** Incommensurable tape loops, drones, slow evolution.
    *   *Reference:* Brian Eno, Tangerine Dream.
*   **State 2: The Process (Medium $K(x)$)**
    *   *Data Condition:* Surface wave dispersion, evolving signal.
    *   *Musical Output:* **Reich Phasing.** Pattern shifting, polyrhythms based on Vp/Vs velocity ratios (~1.73).
    *   *Reference:* Steve Reich, Philip Glass.
*   **State 3: The Stochastic Glitch (High $K(x)$)**
    *   *Data Condition:* Earthquake rupture, chaotic S-waves, low compressibility.
    *   *Musical Output:* **Xenakis/IDM.** Granular bursting, stochastic probability distribution, non-integer rhythms.
    *   *Reference:* Autechre, Aphex Twin, Xenakis.

### B. The Narrative Engine (Solomonoff Induction)
*   **Prediction Error:** The algorithm attempts to predict the next data packet.
*   **Result:**
    *   *High Accuracy:* Maintains current musical state (Stability).
    *   *High Error (Surprise Event):* Triggers a "Drop" or abrupt timbral shift (Subversion).

---

## 4. PHASE III: PARAMETER MAPPING ("The Translator")

This phase translates geological physics into synthesizer parameters.

| Seismic Parameter | Musical Parameter | Mapping Logic |
| :--- | :--- | :--- |
| **Station Metadata (ID)** | **Scale/Harmony** | The FDSN code (e.g., `GEAQ`) is hashed to select a fixed musical scale/mode, giving each station a unique "voice." |
| **Waveform Depth** | **Register/Pitch** | Deeper source = Lower pitch register. |
| **Event Magnitude** | **Gain/Distortion** | Maps to amplitude and saturation. Higher magnitude drives signal clipping/bit-crushing. |
| **P-Wave/S-Wave Gap** | **Rhythmic Phasing** | The time delta determines the delay time between rhythmic layers (Reichian phasing). |
| **Waveform Zero-Crossings** | **Rhythmic Density** | Controls the clock speed of the sequencer. High crossing rate = IDM drill-n-bass speeds. |

---

## 5. PHASE IV: SOUND SYNTHESIS ENGINES ("The Orchestra")

The system utilizes three distinct synthesis engines to cover the aesthetic spectrum of Electronica.

### Engine A: The Trans-Sonic Shifter (Spectral)
*   *Concept:* "The Finite Nature of Music" / Expanding the vocabulary.
*   *Method:* **Frequency Shifting.**
    *   **Infrasound (Earth):** 0.01–5 Hz signals are **Multiplied** (Up-shifted) into Sub-bass (30–60 Hz).
    *   **Micro-tremor (Noise):** High-freq noise is **Divided** (Down-shifted) into crystalline ultrasonic melodies.

### Engine B: The Granular Collider (Texture)
*   *Concept:* Microsound/Geological Grits.
*   *Method:* **Asynchronous Granular Synthesis.**
    *   Seismic data is the *grain source*.
    *   Grain density is controlled by **Seismic Amplitude**.
    *   Grain position is controlled by **Time-to-Event**.

### Engine C: The Stochastic Synth (Melody)
*   *Concept:* Mathematics of Nature.
*   *Method:* **GENDYN (Xenakis) approach.**
    *   Direct synthesis where the seismic waveform *is* the control voltage (CV) for the oscillator shape.
    *   Uses **Electronica Attenuation**: Envelopes follow exponential decay curves mirroring earth strata absorption.

---

## 6. IMPLEMENTATION ROADMAP

### Step 1: The Prototype (Max/MSP)
*   Build the **Kolmogorov Analyzer** in Max/MSP or Python.
*   Route MiniSeed data to control a simple FM synth.
*   *Goal:* Verify that data complexity creates audible musical differentiation.

### Step 2: The "Finite Nature" Expansion
*   Implement the **Trans-Sonic** frequency shifters.
*   Create the "Station ID" harmonic constraints to ensure musical coherence (preventing pure noise).

### Step 3: The Autonomous Loop
*   Deploy the **Solomonoff Induction** observer.
*   Allow the system to run 24/7, transitioning between "Eno" (Quiet) and "Autechre" (Active) states without human intervention.

---

## 7. AESTHETIC GUARDRAILS

To ensure the output remains "Electronica" and not just scientific noise, the following rules apply:

1.  **The 1 Trillion Song Limit:** We actively break this by using frequencies outside the piano roll (The Trans-Sonic approach).
2.  **Attenuation:** All sounds must have long release times (coda) mimicking the Earth's "Q-factor" (ringing).
3.  **Human Range Bridge:** While we use infra/ultrasound concepts, the final mix must contain a mid-range "Bridge" element (pads/chords) to anchor human perception.

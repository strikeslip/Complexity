# 🎛️ Electronica Earthwork: Algorithmic Composition from MiniSeed Data

**Project Goal:** To design an autonomous musical compositional algorithm that converts MiniSeed seismic waveform data into Electronica Music (Sonification), inspired by artists like Aphex Twin, Autechre, and Iannis Xenakis. The core challenge is defining the algorithms that determine musical output from waveform data, rooted in Algorithmic Information Theory (AIT).

---

## I. Foundational Theory: The Algorithmic Compositional Space

Algorithmic Information Theory (AIT) provides the intellectual constraint required for the compositional 'art', by relating the complexity of the data to the complexity of the music.

* **Kolmogorov Complexity ($K(x)$):** The length of the shortest program that generates the data string $x$.
    * **Low $K(x)$ (Compressible Data):** Represents high pattern, repetition, and order (e.g., a constant tremor). Musically, this translates to **minimalism, loops, drones, and simple ostinatos** (e.g., Steve Reich, early Kraftwerk).
    * **High $K(x)$ (Incompressible/Random Data):** Represents low predictability and high randomness (e.g., major earthquake spikes, signal glitches). Musically, this translates to **maximalism, noise, aperiodic rhythms, stochastic textures, and rapid timbral shifts** (e.g., Iannis Xenakis, Autechre, Richard Devine).

The seismic data stream's continuous movement between these two states provides the inherent dynamic range for the composition.

---

## II. Algorithmic Options and Methods

Three distinct methods are proposed for mapping seismic data parameters to the four key compositional domains: Form, Rhythm, Pitch, and Timbre.

### Option 1: The Kolmogorov Complexity Engine (AIT-Driven Stochasticism)

This method directly maps the calculated compressibility of the real-time seismic waveform to structural complexity in the music.

| Compositional Domain | MiniSeed Data Parameter Mapping | Algorithmic Logic |
| :--- | :--- | :--- |
| **I. Form & Structure** | **Short-term $K(x)$ Score:** Calculated over sliding 5-second windows of the waveform's amplitude stream. | **High $K(x)$ (Near-random):** Triggers *Maximalist* sections (dense polyrhythms, rapid granular synthesis). **Low $K(x)$ (Highly repetitive):** Triggers *Minimalist* sections (extended drones, stable harmonic fields). |
| **II. Rhythm & Tempo** | **Waveform Variance / Signal-to-Noise Ratio (SNR).** | **High SNR (Clear Signal):** Engages fixed-rate sequencers based on prime numbers. **Low SNR (Noise/Ambient):** Engages **non-integer time signatures** or completely **a-periodic rhythmic generation** (stochastic music). |
| **III. Pitch & Harmony** | **Waveform Amplitude and Velocity ($V(t)$)** (First derivative of the displacement waveform). | **Low Amplitude:** Constrained to small, microtonal intervals, creating dense, dark clusters. **High Amplitude (Event Trigger):** Pitch selection is determined stochastically, constrained by an *inversion* of the event's dominant frequency. |
| **IV. Timbre & Texture** | **Channel Metadata (HHZ, HHE, HNN) & Sample Rate.** | **Timbral Control:** Each sensor channel (Z, E, N) is mapped to a different sound synthesis type (Subtractive, Granular, Physical Modeling), with modulation rates governed by sample rate *variations*. |

### Option 2: The Solomonoff Induction Observer (Pattern-Based Narrative)

This method uses algorithmic probability, rewarding or punishing the machine's prediction accuracy to create a narrative of expectation and subversion.

| Compositional Domain | MiniSeed Data Parameter Mapping | Algorithmic Logic |
| :--- | :--- | :--- |
| **I. Form & Structure** | **Algorithmic Probability/Prediction Error** (Difference between predicted and actual sample value). | **Narrative Driver:** **Low Error (Predictable data):** Music remains in an **Ambient/Consonant** state. **High Error (Unexpected Event, e.g., P-wave arrival):** Triggers a state change to **Dissonance/Tension** (aggressive distortion, abrupt rhythmic introduction, or silence). |
| **II. Rhythm & Tempo** | **Time-to-Event Duration** (time since the last significant 'trigger'). | **Temporal Density:** Long calm periods result in **slow, wide-open cinematic tempos**. Tempo and rhythmic density build up as the prediction error increases. |
| **III. Pitch & Harmony** | **Network and Station ID (Extrinsic Metadata).** | **Harmonic Palette:** The FDSN identification (`GEAQ`) is converted into a base-12 numerical sequence to define a **custom mode or scale** for the entire composition, ensuring station-specific sonic character. |
| **IV. Timbre & Texture** | **Waveform Peak Amplitude and Frequency Ratios.** | **Resonance Body:** Uses the seismic fundamental frequency to calculate **harmonic series overtone content**. Stronger amplitude results in louder and higher-order overtones, creating rich, evolving textural pads. |

### Option 3: The Multi-Scale Compression/Decompression (Minimalist Looping)

This method uses signal processing techniques (compression and granulation) as the compositional logic, drawing from looping and phasing techniques.

| Compositional Domain | MiniSeed Data Parameter Mapping | Algorithmic Logic |
| :--- | :--- | :--- |
| **I. Form & Structure** | **Data Block Size and Compression Efficiency.** | **Phasing/Loop Length:** High self-similarity blocks are converted into **Phasing Loops**, played simultaneously at slightly different playback speeds to create gradual de-synchronization (Reichian phasing). |
| **II. Rhythm & Tempo** | **Zero-Crossing Rate (ZCR) of the Waveform.** | **Gate/Trigger Generation:** The ZCR (how often the waveform crosses zero amplitude) is mapped to the probability of a percussion event. **Low ZCR:** Sparse, industrial percussion. **High ZCR:** Dense, complex rhythmic patterns generated by a Markov chain. |
| **III. Pitch & Harmony** | **The Waveform Segment Itself (Transposed/Audified).** | **Direct Waveform Synthesis:** The actual compressed seismic waveform is time-stretched and pitch-shifted into the audible range, forming the core "lead voice" (audification). **Octave Control:** Total energy dictates the octave range. |
| **IV. Timbre & Texture** | **Event Glitches and State-of-Health Flags.** | **Controlled Artifacts:** Data imperfections (missing samples, clock drifts) are *amplified* and mapped to key synthesis parameters (e.g., filter saturation, bit-crushing) to generate intentional noise and industrial grit. |

---

## III. The Inaudible Spectrum Engine: Escaping the Creative Plateau

The compositional strategy is augmented by the concept of "The Finite Nature of Music", which suggests exploiting inaudible frequencies to maintain novelty ("Trans-Sonic Music"). MiniSeed data inherently contains infrasonic and high-frequency components that can be manipulated to "escape the limit" of traditional music.

### Integration Method: Spectral Up/Down-Shift Mapping

| Algorithmic Domain | Seismic Mapping (The Inaudible Source) | Compositional Translation (The Breakthrough) |
| :--- | :--- | :--- |
| **Infrasonic Bass** (Visceral Pulse) | **Long-Period Amplitude/Displacement** (0.01 Hz – 5 Hz). | **Up-Shift:** These extremely low frequencies are **up-shifted** by a non-integer factor (e.g., $f' = f \times 5$) into the **Sub-Bass/Low Midrange** (20 Hz–100 Hz), generating the "deep, visceral pulses". |
| **Ultrasonic Melodies** (Ethereal Tones) | **High-Frequency Noise/Tremor** (50 Hz – 100 Hz). | **Down-Conversion:** These components are **down-converted** by an inverse factor (e.g., $f' = f / 4$) to occupy the **High-Frequency Crystalline Range** (8 kHz–15 kHz). This generates **microtonal, ethereal melodies**. |
| **Traditional Notes** (The Bridge) | **P/S Wave Arrivals & Event Triggers.** | **Fixed/Tonal Mapping:** Mapped to the traditional 20 Hz–20 kHz range, constrained to the custom scale defined in Option 2, serving as the "familiar range, now a bridge between extremes". |

## Conclusion: The Art and the Truth

The algorithm **needs to be built.** The integration of Algorithmic Information Theory ($K(x)$) with the concepts of Trans-Sonic Music transforms the project from mere sonification into a form of computational artistry. The algorithm uses the Earth's geophysical truth to generate music that is both structurally rigorous and aesthetically novel.


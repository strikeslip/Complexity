The **ShadowZone** seismic waveform synthesizer is a sophisticated Web Audio-based sonification engine that transforms real-time geological data into a multi-layered granular soundscape. It bridges the gap between scientific data and musical synthesis by mapping seismic characteristics (amplitude, frequency, and wave arrival times) to synthesis parameters.

REF: https://sos.allshookup.org/ShadowZone.html

Here is a detailed technical breakdown of how the system operates:

---

### 1. Data Acquisition and Preprocessing
The engine begins by fetching real-world data from two primary sources:
*   **Event Metadata:** It queries the **USGS Earthquake Catalog** for the latest global seismic event with a magnitude $\ge 6.0$.
*   **Raw Waveform:** It then requests a 60-second **MiniSEED** (Standard for the Exchange of Earthquake Data) buffer from the **EarthScope** (formerly IRIS) data center for a specific station (e.g., `IU.ANMO`).
*   **Normalization:** The raw binary data is parsed into a `Float32Array`. Since seismic sensors (seismometers) record massive integers, the engine normalizes the data to a range of `-1.0 to 1.0` so it can be used as a modulation source.

### 2. The Granular Synthesis Engine
The core of the sound is generated using **Granular Synthesis**, where sounds are broken into "grains" (millisecond-long micro-samples). The system uses three distinct "Clouds" to create a rich texture:

#### A. The `SynthGrain` Class
Every sound you hear is a `SynthGrain` object. Each grain is an individual Web Audio `OscillatorNode` with its own:
*   **Envelope:** A linear ramp (`GainNode`) for attack, sustain, and release to prevent clicking.
*   **Filter:** A `BiquadFilterNode` (lowpass) to shape the timbre of that specific grain.
*   **Panner:** A `StereoPannerNode` that distributes grains across the stereo field to create width.

#### B. Multi-Layered Clouds
The engine runs three concurrent granular generators, each with a different musical role:
1.  **Texture Layer:** High-frequency, high-density grains (45 grains/sec) using Sine waves. It creates the "hiss" and "shimmer" of the seismic background.
2.  **Tone Layer:** Low-frequency, low-density grains (6 grains/sec) using Triangle waves. This provides the melodic "weight" and harmonic foundation.
3.  **Rhythm Layer:** A specialized **`RhythmCloud`** that uses a Sawtooth waveform. Instead of random timing, it uses **Zero-Crossing Detection** from the seismic data to trigger events, effectively "playing" the pulses of the earthquake.

### 3. Seismic-to-Audio Modulation Logic
The "intelligence" of the synthesizer lies in its `modulateFromSeismic()` loop, which runs at 60fps via `requestAnimationFrame`. It extracts three features from the seismic waveform to drive the synthesis:

*   **Instantaneous Amplitude:** Maps to the **Filter Cutoff** and **Grain Density**. When a large seismic wave (like a P or S wave) arrives, the filter opens up (getting brighter) and the number of grains increases (getting louder/thicker).
*   **Derivative (Rate of Change):** Calculated by comparing adjacent samples. This measures the "jaggedness" or frequency of the seismic wave. High derivatives increase **Frequency Spread** and **Pan Spread**, making the sound more chaotic and wide during violent shaking.
*   **RMS (Root Mean Square):** A rolling average of energy. This controls the **Trigger Probability** for the rhythm layer and the overall volume, providing a "swell" effect that follows the earthquake’s intensity envelope.

### 4. Harmonic Quantization
To ensure the output is musical rather than pure noise, the engine employs a `quantizeToScale` function:
*   **Scales:** It holds arrays for Pentatonic, Minor, Dorian, and Harmonic scales.
*   **Logic:** When a grain is about to be born with a "seismic frequency," the engine calculates the nearest frequency that fits the selected musical scale and root note. 
*   **Stretch:** A `stretchFactor` parameter allows the user to slow down the playback of the seismic data without changing the pitch, acting like a "seismic magnifying glass."

### 5. Digital Signal Processing (DSP) Chain
The grains are routed through a master effects chain to provide a sense of space:
*   **Convolution Reverb:** Uses a custom-generated impulse response (a buffer of decaying noise) to simulate a vast underground space. Interestingly, the **wetness of the reverb is modulated by the earthquake's depth**: deeper quakes produce a more "reverberant/distant" sound.
*   **Feedback Delay:** A synchronized delay line (`DelayNode` + `GainNode` feedback) adds rhythmic complexity.
*   **Dynamics Compressor:** Prevents the audio from clipping when multiple granular layers overlap during peak seismic activity.

### 6. The "Muzak" (Freeze) Mode
When the user activates "Muzak" mode:
*   The `playhead` stops moving through the seismic data.
*   The engine enters a **stationary state** where grain density is increased and duration is lengthened.
*   The visualizer begins spawning musical note particles, and the audio transforms from a dynamic, data-driven "storm" into a stable, ambient drone based on the current harmonic state of the earthquake.

### 7. Ray-Traced Visualization
The "ShadowZone" visualizer (the central Earth graphic) isn't just an image; it is a mathematical simulation of seismic wave propagation:
*   **Ray Tracing:** It calculates the paths of seismic waves from the epicenter, accounting for refraction as waves enter the Earth's Core.
*   **Shadow Zones:** It highlights the "Shadow Zone" ($103^\circ$ to $142^\circ$ from epicenter) where S-waves cannot travel, visually representing why certain seismic stations "miss" parts of the signal—a direct nod to the physics that gives the tool its name.

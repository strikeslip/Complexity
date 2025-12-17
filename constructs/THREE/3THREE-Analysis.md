# 3THREE.html — Complete Technical Analysis

## Granular Seismic Synthesis Engine

---

## 1. SYSTEM OVERVIEW

**3THREE** is a real-time granular synthesis engine that transforms seismic earthquake data into generative electronic music. Unlike traditional granular synthesis (which chops audio into grains), this system uses **synthesized oscillator grains** whose parameters are **modulated by seismic waveform analysis**.

### Core Architecture

```
┌───────────────────────────────────────────────────────────────────────┐
│                        3THREE SIGNAL FLOW                             │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────────┐       │
│  │  USGS API    │────►│  IRIS/FDSN   │────►│  MiniSEED Parser │       │
│  │  (Events)    │     │  (Waveforms) │     │  (Normalize)     │       │
│  └──────────────┘     └──────────────┘     └────────┬─────────┘       │
│                                                      │                │
│                                    ┌─────────────────┘                │
│                                    ▼                                  │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │                    SEISMIC ANALYSIS                             |  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │  │
│  │  │  Amplitude  │  │  Derivative │  │     Zero-Crossings      │  │  │
│  │  │  |sample|   │  │  Δsample/Δt │  │  sign(s[i]) ≠ sign[i-1] │  │  │
│  │  └──────┬──────┘  └──────┬──────┘  └────────────┬────────────┘  │  │
│  │         │                │                      │               │  │
│  │         │                │                      │               │  │
│  │  ┌──────┴──────┐  ┌──────┴──────┐  ┌───────────┴───────────┐    │  │
│  │  │   RMS       │  │             │  │                       │    │  │
│  │  │  Energy     │  │             │  │                       │    │  │
│  │  └──────┬──────┘  │             │  │                       │    │  │
│  └─────────┼─────────┼─────────────┼──┼───────────────────────┘    │  │
│            │         │             │  │                            │  │
│            ▼         ▼             ▼  ▼                            │  │
│  ┌─────────────────────────────────────────────────────────────┐   │  │
│  │              GRANULAR SEISMIC ENGINE                        │   │  │
│  │                                                             │   │  │
│  │  ┌──────────────────────────────────────────────────────┐   │   │  │
│  │  │  TEXTURE CLOUD (High-frequency shimmer)              │   │   │  │
│  │  │  • Density: 30-100 g/s ◄── Amplitude                 │   │   │  │
│  │  │  • Freq Spread: 0.3-1.5 oct ◄── Derivative           │   │   │  │
│  │  │  • Base Freq: root×4 ◄── Sample value                │   │   │  │
│  │  │  • Pan Spread: 0.4-1.0 ◄── Derivative                │   │   │  │
│  │  └──────────────────────────────────────────────────────┘   │   │  │
│  │                                                             │   │  │
│  │  ┌──────────────────────────────────────────────────────┐   │   │  │
│  │  │  TONE CLOUD (Warm, sustained tones)                  │   │   │  │
│  │  │  • Density: 5-13 g/s ◄── RMS Energy                  │   │   │  │
│  │  │  • Freq Spread: 0.2-0.8 oct ◄── Derivative           │   │   │  │
│  │  │  • Base Freq: root                                   │   │   │  │
│  │  │  • Long grains (250ms)                               │   │   │  │
│  │  └──────────────────────────────────────────────────────┘   │   │  │
│  │                                                             │   │  │
│  │  ┌──────────────────────────────────────────────────────┐   │   │  │
│  │  │  RHYTHM CLOUD (Percussive hits)                      │   │   │  │
│  │  │  • Trigger Prob: 0.3-0.9 ◄── RMS Energy              │   │   │  │
│  │  │  • Base Freq: root×2 × (1 + RMS×0.5)                 │   │   │  │
│  │  │  • Zero-crossing sync for rhythmic feel              │   │   │  │
│  │  └──────────────────────────────────────────────────────┘   │   │  │
│  │                           │                                 │   │  │
│  │                           ▼                                 │   │  │
│  │                    ┌─────────────┐                          │   │  │
│  │                    │   FILTER    │ ◄── Amplitude (800-10kHz)│   │  │
│  │                    │  (Lowpass)  │                          │   │  │
│  │                    └──────┬──────┘                          │   │  │
│  │                           │                                 │   │  │
│  │                           ▼                                 │   │  │
│  │                    ┌─────────────┐                          │   │  │
│  │                    │Engine Master│                          │   │  │
│  │                    │    Gain     │                          │   │  │
│  │                    └──────┬──────┘                          │   │  │
│  └───────────────────────────┼─────────────────────────────────┘   │  │
│                              │                                     │  │
│         ┌────────────────────┼────────────────────┐                │  │
│         │                    │                    │                │  │
│         ▼                    ▼                    ▼                │  │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐         │  │
│  │   REVERB    │      │   DELAY     │      │  SUB-BASS   │         │  │
│  │ (Convolver) │      │ (Feedback)  │      │   DRONE     │         │  │
│  │  3.5s tail  │      │  375ms      │      │  root/2 Hz  │         │  │
│  └──────┬──────┘      └──────┬──────┘      └──────┬──────┘         │  │
│         │                    │                    │                │  │
│         └────────────────────┼────────────────────┘                │  │
│                              ▼                                     │  │
│                       ┌─────────────┐                              │  │
│                       │ COMPRESSOR  │                              │  │
│                       │  -18dB, 4:1 │                              │  │
│                       └──────┬──────┘                              │  │
│                              ▼                                     │  │
│                       ┌─────────────┐                              │  │
│                       │MASTER GAIN  │                              │  │
│                       └──────┬──────┘                              │  │
│                              ▼                                     │  │
│                         [SPEAKERS]                                 │  │
│                                                                    │  │
└────────────────────────────────────────────────────────────────────┘   
```

---

## 2. DATA ACQUISITION LAYER

### 2.1 Configuration Constants

```javascript
const SEISMIC_DATA_DURATION_S = 60;      // 60 seconds of waveform data
const SAMPLE_RATE = 44100;                // Audio sample rate
const MIN_MAGNITUDE = 6.0;                // Only M6.0+ earthquakes
const FETCH_TIMESPAN_DAYS = 30;           // Search window
const STATION_INFO = { 
    net: "IU",      // Global Seismographic Network
    sta: "ANMO",    // Albuquerque, New Mexico
    loc: "00",      // Location code
    cha: "BHZ"      // Broadband High-gain Vertical (20 sps)
};
const CYCLE_DURATION_S = 31;              // New parameters every 31 seconds
```

### 2.2 USGS Event Query

```javascript
async function fetchLatestEvent() {
    const url = `https://earthquake.usgs.gov/fdsnws/event/1/query
        ?format=geojson
        &starttime=${start}
        &endtime=${end}
        &minmagnitude=${MIN_MAGNITUDE}
        &orderby=time`;
    // Returns: { mag, place, depth, time, coordinates }
}
```

### 2.3 IRIS MiniSEED Retrieval

```javascript
async function fetchMiniSEED(eventTime) {
    const url = `https://service.iris.edu/fdsnws/dataselect/1/query
        ?net=${net}&sta=${sta}&loc=${loc}&cha=${cha}
        &starttime=${start}&endtime=${end}
        &format=miniseed`;
    // Returns: ArrayBuffer of seismic waveform
}
```

### 2.4 MiniSEED Parser

```javascript
function parseMiniSEED(buffer) {
    const view = new DataView(buffer);
    const numSamples = view.getInt16(46, false);  // Big-endian at offset 46
    const dataOffset = 64;                         // Fixed header size
    
    // Extract 32-bit integers, normalize to [-1, 1]
    const rawData = new Int32Array(buffer.slice(dataOffset, dataOffset + numSamples * 4));
    const maxVal = rawData.reduce((m, v) => Math.max(m, Math.abs(v)), 0);
    const normalized = new Float32Array(rawData.length);
    for (let i = 0; i < rawData.length; i++) {
        normalized[i] = rawData[i] / maxVal;
    }
    return normalized;
}
```

**Fallback:** If parsing fails, generates synthetic exponentially-decaying noise.

---

## 3. SEISMIC ANALYSIS FUNCTIONS

### 3.1 Zero-Crossing Detection

```javascript
function detectZeroCrossings(data) {
    const crossings = [];
    for (let i = 1; i < data.length; i++) {
        if (data[i-1] * data[i] < 0) {  // Sign change
            crossings.push(i / data.length);  // Normalized position [0,1]
        }
    }
    return crossings;
}
```

**Purpose:** Provides rhythmic trigger points for the RhythmCloud — grains are more likely to fire at zero-crossings, creating a pulse synchronized to seismic activity patterns.

### 3.2 Derivative Calculation (Rate of Change)

```javascript
function calculateDerivative(data, idx, win = 5) {
    if (idx < win || idx >= data.length - win) return 0;
    let sum = 0;
    for (let i = 1; i <= win; i++) {
        sum += Math.abs(data[idx + i] - data[idx - i]);
    }
    return sum / (2 * win);  // Average absolute change
}
```

**Purpose:** High derivative = rapid seismic movement = wider pitch spread and stereo width (more chaotic sound).

### 3.3 RMS Energy Calculation

```javascript
function calculateRMS(data, idx, win = 50) {
    const start = Math.max(0, idx - win);
    const end = Math.min(data.length, idx + win);
    let sum = 0;
    for (let i = start; i < end; i++) {
        sum += data[i] * data[i];
    }
    return Math.sqrt(sum / (end - start));
}
```

**Purpose:** Measures local energy level. High RMS = more rhythm triggers and denser tone cloud.

---

## 4. PITCH QUANTIZATION SYSTEM

### 4.1 Scale Definitions

```javascript
const SCALES = {
    chromatic: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11],  // All 12 semitones
    pentatonic: [0, 2, 4, 7, 9],                         // Major pentatonic
    minor: [0, 2, 3, 5, 7, 8, 10],                       // Natural minor
    major: [0, 2, 4, 5, 7, 9, 11],                       // Major scale
    fifths: [0, 7, 14, 21],                              // Stacked fifths
    harmonic: [0, 12, 19, 24, 28, 31, 34, 36]           // Harmonic series
};
```

### 4.2 Quantization Algorithm

```javascript
function quantizeToScale(freq, scale, rootFreq) {
    const intervals = SCALES[scale];
    const freqs = [];
    
    // Build frequency table across 7 octaves (-3 to +4)
    for (let oct = -3; oct <= 4; oct++) {
        for (const iv of intervals) {
            const f = rootFreq * Math.pow(2, oct) * semitonesToRatio(iv);
            if (f >= 20 && f <= 10000) freqs.push(f);
        }
    }
    
    // Find nearest scale frequency
    let nearest = freqs[0], minDist = Math.abs(freq - nearest);
    for (const f of freqs) {
        const dist = Math.abs(freq - f);
        if (dist < minDist) { minDist = dist; nearest = f; }
    }
    return nearest;
}

const semitonesToRatio = (st) => Math.pow(2, st / 12);
```

**Result:** Random frequencies are snapped to the nearest note in the selected scale, ensuring harmonic coherence.

---

## 5. GRAIN SYNTHESIS ENGINE

### 5.1 SynthGrain Class — The Atomic Sound Unit

```javascript
class SynthGrain {
    constructor(ctx, params) { 
        this.ctx = ctx; 
        this.params = params; 
    }
    
    play(dest, startTime) {
        const { frequency, duration, waveform, pan, amplitude, attack, release, detune } = this.params;
        
        // OSCILLATOR — actual sound source
        const osc = this.ctx.createOscillator();
        osc.type = waveform || 'sine';      // sine, triangle, sawtooth, square
        osc.frequency.value = frequency;
        osc.detune.value = detune || 0;     // ±cents for thickness
        
        // ENVELOPE — ASR (Attack-Sustain-Release)
        const env = this.ctx.createGain();
        const att = duration * attack;
        const rel = duration * release;
        const sus = Math.max(0, duration - att - rel);
        
        env.gain.setValueAtTime(0, startTime);
        env.gain.linearRampToValueAtTime(amplitude, startTime + att);
        if (sus > 0) env.gain.setValueAtTime(amplitude, startTime + att + sus);
        env.gain.linearRampToValueAtTime(0, startTime + duration);
        
        // PANNING — stereo placement
        const panner = this.ctx.createStereoPanner();
        panner.pan.value = clamp(pan, -1, 1);
        
        // ROUTING
        osc.connect(env).connect(panner).connect(dest);
        
        // LIFECYCLE
        osc.start(startTime);
        osc.stop(startTime + duration + 0.01);
        
        return osc;
    }
}
```

**Key Design Decision:** Grains use **oscillators**, not buffer playback. This is because seismic data is sampled at ~20Hz (too slow to be audible). The seismic data instead **modulates synthesis parameters**.

### 5.2 Grain Envelope Visualization

```
Amplitude
    │
 amp├───────────────┐
    │              ╱ ╲
    │             ╱   ╲
    │            ╱     ╲
    │           ╱       ╲
    │          ╱         ╲
  0 ├─────────╱───────────╲─────────
    │        │             │
    └────────┴─────────────┴────────► Time
             │  attack  │sustain│release│
             ◄──────────duration──────────►
```

---

## 6. GRANULAR CLOUD SYSTEM

### 6.1 Base GranularCloud Class

```javascript
class GranularCloud {
    constructor(ctx, dest, type = 'texture') {
        this.ctx = ctx;
        this.destination = dest;
        this.type = type;
        this.isActive = false;
        this.schedulerId = null;
        
        // Cloud parameters (defaults)
        this.density = 30;              // Grains per second
        this.grainDuration = 0.05;      // 50ms
        this.durationSpread = 0.5;      // ±50% randomization
        this.baseFrequency = 220;       // Hz
        this.frequencySpread = 0.5;     // Octaves
        this.panSpread = 0.8;           // Stereo width
        this.amplitude = 0.15;          // Grain volume
        this.attackRatio = 0.3;         // 30% of duration
        this.releaseRatio = 0.5;        // 50% of duration
        this.waveform = 'sine';
        
        // Quantization
        this.quantizeEnabled = true;
        this.scale = 'pentatonic';
        this.rootFreq = 110;
        
        // Output stage
        this.outputGain = this.ctx.createGain();
        this.outputGain.gain.value = 0.6;
        this.outputGain.connect(this.destination);
    }
    
    scheduleGrain() {
        if (!this.isActive) return;
        
        const now = this.ctx.currentTime;
        
        // Randomize frequency within spread (in octaves)
        let freq = this.baseFrequency * Math.pow(2, (Math.random() - 0.5) * this.frequencySpread * 2);
        
        // Snap to scale
        if (this.quantizeEnabled) {
            freq = quantizeToScale(freq, this.scale, this.rootFreq);
        }
        
        // Randomize duration
        const dur = this.grainDuration * (1 + (Math.random() - 0.5) * this.durationSpread);
        
        // Create and play grain
        const params = {
            frequency: freq,
            duration: dur,
            waveform: this.waveform,
            pan: (Math.random() - 0.5) * 2 * this.panSpread,
            amplitude: this.amplitude * (0.7 + Math.random() * 0.3),
            attack: this.attackRatio,
            release: this.releaseRatio,
            detune: (Math.random() - 0.5) * 20  // ±10 cents
        };
        
        new SynthGrain(this.ctx, params).play(this.outputGain, now);
        
        // Schedule next grain with jitter
        const interval = 1000 / this.density;
        const jitter = interval * 0.2 * (Math.random() - 0.5);
        this.schedulerId = setTimeout(() => this.scheduleGrain(), Math.max(10, interval + jitter));
    }
}
```

### 6.2 RhythmCloud — Zero-Crossing Synchronized

```javascript
class RhythmCloud extends GranularCloud {
    constructor(ctx, dest, zeroCrossings) {
        super(ctx, dest, 'rhythm');
        this.zeroCrossings = zeroCrossings;  // Array of normalized positions
        this.crossingIndex = 0;
        this.triggerProbability = 0.7;       // 70% chance to fire
        this.waveform = 'triangle';
    }
    
    scheduleGrain() {
        if (!this.isActive) return;
        
        // Advance through zero-crossings cyclically
        this.crossingIndex = (this.crossingIndex + 1) % Math.max(1, this.zeroCrossings.length);
        
        // Probabilistic triggering (controlled by RMS energy)
        if (Math.random() < this.triggerProbability) {
            const now = this.ctx.currentTime;
            let freq = this.baseFrequency;
            if (this.quantizeEnabled) freq = quantizeToScale(freq, this.scale, this.rootFreq);
            
            const params = {
                frequency: freq,
                duration: this.grainDuration,
                waveform: this.waveform,
                pan: (Math.random() - 0.5) * 2 * this.panSpread,
                amplitude: this.amplitude,
                attack: 0.05,   // Sharp attack (percussive)
                release: 0.7,   // Long release (resonant)
                detune: 0
            };
            
            new SynthGrain(this.ctx, params).play(this.outputGain, now);
        }
        
        this.schedulerId = setTimeout(() => this.scheduleGrain(), 1000 / this.density);
    }
}
```

### 6.3 Three-Layer Cloud Architecture

| Layer | Purpose | Density | Grain Duration | Frequency | Waveform |
|-------|---------|---------|----------------|-----------|----------|
| **Texture** | High-frequency shimmer | 30-100 g/s | 40ms | root × 4 | sine |
| **Tone** | Warm sustained bed | 5-13 g/s | 250ms | root | triangle |
| **Rhythm** | Percussive accents | 12 g/s | 60ms | root × 2 | sawtooth |

---

## 7. SEISMIC MODULATION ENGINE

### 7.1 GranularSeismicEngine Class

```javascript
class GranularSeismicEngine {
    constructor(ctx, dest) {
        this.ctx = ctx;
        this.destination = dest;
        this.seismicData = [];
        this.zeroCrossings = [];
        this.playhead = 0;           // Current position [0,1]
        this.isActive = false;
        this.isFrozen = false;
        this.stretchFactor = 1.0;    // Time stretch
        this.playbackSpeed = 0.0005; // Base scan rate
        
        // Clouds
        this.textureCloud = null;
        this.toneCloud = null;
        this.rhythmCloud = null;
        
        // Master filter (seismic-controlled)
        this.filter = this.ctx.createBiquadFilter();
        this.filter.type = 'lowpass';
        this.filter.frequency.value = 4000;
        this.filter.Q.value = 2;
        
        // Master gain
        this.masterGain = this.ctx.createGain();
        this.masterGain.gain.value = 0;
        
        this.filter.connect(this.masterGain);
        this.masterGain.connect(this.destination);
    }
}
```

### 7.2 Real-Time Modulation Loop

```javascript
modulateFromSeismic() {
    if (!this.isActive) return;
    
    // Get current seismic values
    const idx = Math.floor(this.playhead * this.seismicData.length);
    const sample = this.seismicData[idx] || 0;
    const deriv = calculateDerivative(this.seismicData, idx);
    const rms = calculateRMS(this.seismicData, idx);
    const now = this.ctx.currentTime;
    const rootFreq = parseFloat(rootSelect.value);
    
    // ═══════════════════════════════════════════════════════════
    // MAPPING 1: Amplitude → Grain Density & Filter Cutoff
    // ═══════════════════════════════════════════════════════════
    const amp = Math.abs(sample);
    const targetFilter = 1500 + amp * 6000;  // 1.5kHz - 7.5kHz
    
    if (this.textureCloud?.isActive) {
        this.textureCloud.density = 30 + amp * 70;      // 30-100 g/s
        this.textureCloud.amplitude = 0.05 + amp * 0.1; // Louder when active
    }
    this.filter.frequency.setTargetAtTime(clamp(targetFilter, 800, 10000), now, 0.03);
    
    // ═══════════════════════════════════════════════════════════
    // MAPPING 2: Derivative → Pitch Spread & Stereo Width
    // ═══════════════════════════════════════════════════════════
    const df = Math.min(1, deriv * 15);  // Normalize to [0,1]
    
    if (this.textureCloud?.isActive) {
        this.textureCloud.frequencySpread = 0.3 + df * 1.2;  // 0.3-1.5 octaves
        this.textureCloud.panSpread = 0.4 + df * 0.6;        // 40-100% stereo
    }
    if (this.toneCloud?.isActive) {
        this.toneCloud.frequencySpread = 0.2 + df * 0.6;     // 0.2-0.8 octaves
    }
    
    // ═══════════════════════════════════════════════════════════
    // MAPPING 3: RMS Energy → Rhythm Probability & Tone Density
    // ═══════════════════════════════════════════════════════════
    if (this.rhythmCloud?.isActive) {
        this.rhythmCloud.triggerProbability = 0.3 + rms * 0.6;  // 30-90%
        this.rhythmCloud.baseFrequency = rootFreq * 2 * (1 + rms * 0.5);  // Pitch up with energy
    }
    if (this.toneCloud?.isActive) {
        this.toneCloud.density = 5 + rms * 8;  // 5-13 g/s
    }
    
    // ═══════════════════════════════════════════════════════════
    // MAPPING 4: Sample Value → Base Frequency (bipolar)
    // ═══════════════════════════════════════════════════════════
    if (this.textureCloud?.isActive) {
        // Positive samples = higher pitch, negative = lower
        this.textureCloud.baseFrequency = rootFreq * 4 * (1 + sample * 0.3);
    }
    
    // ═══════════════════════════════════════════════════════════
    // PLAYHEAD ADVANCEMENT
    // ═══════════════════════════════════════════════════════════
    if (!this.isFrozen) {
        this.playhead += this.playbackSpeed / this.stretchFactor;
        if (this.playhead >= 1) this.playhead = 0;  // Loop
    }
    
    // Update visualization
    drawWaveform(this.seismicData, this.playhead);
    
    // Continue loop at ~60fps
    this.animFrameId = requestAnimationFrame(() => this.modulateFromSeismic());
}
```

### 7.3 Seismic-to-Sound Mapping Summary

| Seismic Feature | Analysis Function | Target Parameter | Range |
|-----------------|-------------------|------------------|-------|
| **Amplitude** | `\|sample\|` | Texture density | 30-100 g/s |
| **Amplitude** | `\|sample\|` | Filter cutoff | 800-10000 Hz |
| **Amplitude** | `\|sample\|` | Texture amplitude | 0.05-0.15 |
| **Derivative** | `calculateDerivative()` | Freq spread (texture) | 0.3-1.5 oct |
| **Derivative** | `calculateDerivative()` | Pan spread | 0.4-1.0 |
| **Derivative** | `calculateDerivative()` | Freq spread (tone) | 0.2-0.8 oct |
| **RMS Energy** | `calculateRMS()` | Rhythm probability | 0.3-0.9 |
| **RMS Energy** | `calculateRMS()` | Tone density | 5-13 g/s |
| **RMS Energy** | `calculateRMS()` | Rhythm pitch | root×2 to root×3 |
| **Sample Value** | `sample` (bipolar) | Texture base freq | ±30% |
| **Zero-Crossings** | `detectZeroCrossings()` | Rhythm trigger sync | Position array |

---

## 8. FREEZE SYSTEM (Spectral Sustain)

### 8.1 Freeze Activation

```javascript
freeze() {
    this.isFrozen = true;  // Stop playhead advancement
    
    if (this.textureCloud) {
        this.textureCloud.density = 80;           // Very dense
        this.textureCloud.grainDuration = 0.12;   // Longer grains
        this.textureCloud.frequencySpread = 0.2;  // Narrow (more tonal)
    }
    if (this.toneCloud) {
        this.toneCloud.density = 12;              // Denser
        this.toneCloud.grainDuration = 0.4;       // Much longer
    }
}
```

### 8.2 Unfreeze Restoration

```javascript
unfreeze() {
    this.isFrozen = false;  // Resume playhead
    
    if (this.textureCloud) {
        this.textureCloud.density = 40;
        this.textureCloud.grainDuration = 0.04;
        this.textureCloud.frequencySpread = 1.0;
    }
    if (this.toneCloud) {
        this.toneCloud.density = 6;
        this.toneCloud.grainDuration = 0.25;
    }
}
```

**Effect:** Freezing locks the playhead at a seismic peak, increasing grain density and duration to create a sustained, shimmering spectral texture — like holding a moment of the earthquake in time.

---

## 9. EFFECTS PROCESSING

### 9.1 Algorithmic Convolution Reverb

```javascript
async function createReverb() {
    const len = 3.5 * audioCtx.sampleRate;  // 3.5 second tail
    const buf = audioCtx.createBuffer(2, len, audioCtx.sampleRate);
    
    for (let ch = 0; ch < 2; ch++) {
        const data = buf.getChannelData(ch);
        for (let i = 0; i < len; i++) {
            // Exponential decay with noise (synthetic impulse response)
            data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / len, 2.2);
        }
    }
    
    reverb = audioCtx.createConvolver();
    reverb.buffer = buf;
    reverbWetGain = audioCtx.createGain();
    reverbWetGain.gain.value = 0.4;  // 40% wet
    reverb.connect(reverbWetGain).connect(masterGain);
}
```

**The 2.2 exponent:** Creates a decay curve that's fast at first, then elongates — characteristic of large hall reverbs.

### 9.2 Feedback Delay

```javascript
function createDelay() {
    delay = audioCtx.createDelay(2.0);
    delay.delayTime.value = 0.375;        // 375ms (dotted eighth at ~80bpm)
    
    delayFeedbackGain = audioCtx.createGain();
    delayFeedbackGain.gain.value = 0.4;   // 40% feedback
    
    delayWetGain = audioCtx.createGain();
    delayWetGain.gain.value = 0.25;       // 25% wet
    
    // Feedback loop
    delay.connect(delayFeedbackGain).connect(delay);
    delay.connect(delayWetGain).connect(masterGain);
}
```

### 9.3 Depth-Scaled Effects

```javascript
// Earthquake depth affects effect parameters
const df = Math.min(1, currentQuakeInfo.depth / 600);  // Normalize to [0,1]

reverbWetGain.gain.setTargetAtTime(0.3 + df * 0.4, ...);      // 30-70% reverb
delay.delayTime.setTargetAtTime(0.25 + df * 0.35, ...);       // 250-600ms delay
delayFeedbackGain.gain.setTargetAtTime(0.35 + df * 0.25, ...); // 35-60% feedback
```

**Deeper earthquakes** = more reverb, longer delay, more feedback = more spacious, cavernous sound.

### 9.4 Sub-Bass Drone

```javascript
function createSubBass(rootFreq) {
    const osc = audioCtx.createOscillator();
    osc.type = 'sine';
    osc.frequency.value = rootFreq / 2;  // One octave below root
    
    // Slow LFO for subtle pitch wobble
    const lfo = audioCtx.createOscillator();
    lfo.type = 'sine';
    lfo.frequency.value = 0.04 + Math.random() * 0.02;  // 0.04-0.06 Hz
    
    const lfoGain = audioCtx.createGain();
    lfoGain.gain.value = rootFreq / 2 * 0.02;  // ±2% pitch modulation
    
    lfo.connect(lfoGain).connect(osc.frequency);
    osc.connect(gain).connect(masterGain);
    
    gain.gain.setTargetAtTime(0.15, now, 2);  // Slow fade-in
}
```

---

## 10. CYCLE MANAGEMENT SYSTEM

### 10.1 Deterministic Pseudo-Random Generation

```javascript
function startNewCycle() {
    cycleCount++;
    
    // Seed from cycle number + earthquake magnitude
    const seed = cycleCount * 31 + Math.floor(currentQuakeInfo.mag * 10);
    
    // Sine-based PRNG (deterministic for same seed)
    const pr = n => {
        const x = Math.sin(seed + n * 9999) * 10000;
        return x - Math.floor(x);  // Fractional part [0,1)
    };
    
    // Parameter pools
    const waveforms = ['sine', 'triangle', 'sawtooth'];
    const texDens = [30, 40, 50, 60, 80];
    const tonDens = [5, 8, 10, 12, 15];
    const grDurs = [0.02, 0.03, 0.05, 0.08, 0.1, 0.15, 0.2];
    const pans = [0.3, 0.5, 0.7, 0.9, 1.0];
    
    // Generate parameters
    const params = {
        textureDensity: texDens[Math.floor(pr(1) * texDens.length)],
        toneDensity: tonDens[Math.floor(pr(2) * tonDens.length)],
        textureWaveform: waveforms[Math.floor(pr(3) * waveforms.length)],
        toneWaveform: waveforms[Math.floor(pr(4) * waveforms.length)],
        grainDuration: grDurs[Math.floor(pr(5) * grDurs.length)],
        panSpread: pans[Math.floor(pr(6) * pans.length)]
    };
    
    // Apply to clouds
    if (granularEngine.textureCloud) {
        granularEngine.textureCloud.density = params.textureDensity;
        granularEngine.textureCloud.waveform = params.textureWaveform;
        granularEngine.textureCloud.panSpread = params.panSpread;
    }
    if (granularEngine.toneCloud) {
        granularEngine.toneCloud.density = params.toneDensity;
        granularEngine.toneCloud.waveform = params.toneWaveform;
        granularEngine.toneCloud.grainDuration = params.grainDuration * 3;
    }
}
```

### 10.2 Cycle Timing

```javascript
const cycleInt = setInterval(() => {
    if (isPlaying) startNewCycle();
}, CYCLE_DURATION_S * 1000);  // Every 31 seconds
```

**Why 31 seconds?** Prime number avoids phase-locking with musical patterns, ensuring continuous variation.

---

## 11. USER INTERFACE CONTROLS

### 11.1 Real-Time Parameter Sliders

| Control | Range | Target |
|---------|-------|--------|
| **Time Stretch** | 0.25x - 4x | `stretchFactor` (playhead speed) |
| **Density** | 5-100 g/s | `textureCloud.density` |
| **Grain Size** | 10-200ms | `textureCloud.grainDuration` |
| **Reverb** | 0-100% | `reverbWetGain.gain` |

### 11.2 Scale & Root Selection

```javascript
scaleSelect.onchange = function() {
    granularEngine?.setScale(this.value);  // Updates all clouds
};

rootSelect.onchange = function() {
    const f = parseFloat(this.value);
    granularEngine?.setRootFreq(f);        // Updates all clouds
    if (subBassNodes) {
        subBassNodes.osc.frequency.setTargetAtTime(f / 2, audioCtx.currentTime, 0.5);
    }
};
```

### 11.3 Layer Toggle System

```javascript
function toggleLayer(layer) {
    layerStates[layer] = !layerStates[layer];
    const btn = document.getElementById('toggle' + layer.charAt(0).toUpperCase() + layer.slice(1));
    btn.classList.toggle('inactive', !layerStates[layer]);
    granularEngine?.setLayerActive(layer, layerStates[layer]);
}
```

---

## 12. WAVEFORM VISUALIZATION

```javascript
function drawWaveform(data, playheadPos = 0) {
    const w = waveformCanvas.width / devicePixelRatio;
    const h = waveformCanvas.height / devicePixelRatio;
    
    // Clear
    waveformCtx.fillStyle = '#0a0a0f';
    waveformCtx.fillRect(0, 0, w, h);
    
    // Draw waveform (min/max per pixel column)
    waveformCtx.strokeStyle = '#ff6f00';
    waveformCtx.beginPath();
    const step = Math.ceil(data.length / w);
    for (let i = 0; i < w; i++) {
        const idx = Math.floor(i * data.length / w);
        let min = 1, max = -1;
        for (let j = 0; j < step && idx + j < data.length; j++) {
            const v = data[idx + j];
            if (v < min) min = v;
            if (v > max) max = v;
        }
        const yMin = (1 - min) * h / 2;
        const yMax = (1 - max) * h / 2;
        waveformCtx.lineTo(i, yMin);
        waveformCtx.lineTo(i, yMax);
    }
    waveformCtx.stroke();
    
    // Update playhead position
    playheadEl.style.left = `${playheadPos * 100}%`;
}
```

---

## 13. PERFORMANCE CHARACTERISTICS

### 13.1 Audio Node Count (Typical)

| Component | Nodes Created | Nodes Active |
|-----------|---------------|--------------|
| Texture Cloud | ~40/sec | ~2-4 at once |
| Tone Cloud | ~6/sec | ~1-2 at once |
| Rhythm Cloud | ~12/sec | ~1 at once |
| Sub-Bass | 3 (osc, lfo, gain) | 3 |
| Effects | 6 (reverb, delays, gains) | 6 |
| **Total** | ~60/sec created | ~15-20 active |

### 13.2 CPU Optimization

- Grains self-terminate via `osc.stop()`
- `setTimeout` scheduling with jitter prevents synchronization
- `requestAnimationFrame` for modulation loop (60fps cap)
- Filter smoothing via `setTargetAtTime` (avoids zipper noise)

---

## 14. CREATIVE APPLICATIONS

### 14.1 Suggested Presets

**"Tectonic Ambient"**
- Scale: Pentatonic, Root: A2
- Time Stretch: 2x
- Density: 30 g/s
- Reverb: 60%

**"Aftershock Pulse"**
- Scale: Fifths, Root: E2
- Time Stretch: 0.5x
- Density: 80 g/s
- Reverb: 20%

**"Subduction Drone"**
- Scale: Harmonic, Root: A1
- Time Stretch: 4x
- Density: 15 g/s
- Reverb: 80%
- Freeze on peak

---

## 15. SUMMARY

**3THREE** transforms earthquake data into generative music through:

1. **Data Pipeline:** USGS → IRIS → MiniSEED → Normalized Float32Array
2. **Analysis:** Amplitude, Derivative, RMS, Zero-Crossings extracted in real-time
3. **Synthesis:** Oscillator-based grains (not buffer playback) organized into 3 clouds
4. **Modulation:** Seismic features continuously modulate grain parameters
5. **Quantization:** Pitches snapped to musical scales for harmonic coherence
6. **Effects:** Depth-scaled reverb/delay + sub-bass drone
7. **Cycles:** Every 31 seconds, pseudo-random parameter evolution

The result is a **living sonic portrait** of Earth's seismic activity — each earthquake creating unique, musically coherent textures that evolve over time.

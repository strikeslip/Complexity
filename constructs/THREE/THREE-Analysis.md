# THREE.html — Technical Analysis & Granular Synthesis Transformation Guide

## PART ONE: COMPLETE TECHNICAL BREAKDOWN

---

### 1. SYSTEM ARCHITECTURE OVERVIEW

THREE.html is a **real-time FM synthesis sonification engine** that transforms live seismic waveform data into generative electronic music through 31-second algorithmic composition cycles.

```
┌─────────────────────────────────────────────────────────────────┐
│                    THREE.html SIGNAL FLOW                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [USGS API] ──► [IRIS/MiniSEED] ──► [Parser] ──► [Float32Array] │
│                                                        │        │
│                                                        ▼        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              FMSeismicEngine (Core DSP)                 │   │
│  │  ┌─────────┐    ┌─────────────┐    ┌────────────┐      │   │
│  │  │Modulator├───►│ Mod Index   ├───►│  Carrier   │      │   │
│  │  │   OSC   │    │   Gain      │    │    OSC     │      │   │
│  │  └─────────┘    └─────────────┘    └─────┬──────┘      │   │
│  │                                          │             │   │
│  │  ┌─────────┐    ┌─────────────┐          ▼             │   │
│  │  │  LFO    ├───►│  LFO Gain   ├───►[carrier.freq]      │   │
│  │  └─────────┘    └─────────────┘                        │   │
│  │                                          │             │   │
│  │                      ┌───────────────────┘             │   │
│  │                      ▼                                 │   │
│  │               ┌─────────────┐                          │   │
│  │               │Carrier Gain │ ◄── Seismic Modulation   │   │
│  │               └──────┬──────┘                          │   │
│  │                      │                                 │   │
│  │                      ▼                                 │   │
│  │               ┌─────────────┐                          │   │
│  │               │StereoPanner │                          │   │
│  │               └──────┬──────┘                          │   │
│  └──────────────────────┼──────────────────────────────────┘   │
│                         │                                      │
│                         ▼                                      │
│                  ┌─────────────┐                               │
│                  │ LPF Filter  │ ◄── Seismic filterMod         │
│                  └──────┬──────┘                               │
│                         │                                      │
│           ┌─────────────┼─────────────┐                        │
│           ▼             ▼             ▼                        │
│    ┌──────────┐  ┌──────────┐  ┌──────────┐                    │
│    │ SubBass  │  │ Reverb   │  │  Delay   │                    │
│    │  Drone   │  │(Convolv) │  │(Feedback)│                    │
│    └────┬─────┘  └────┬─────┘  └────┬─────┘                    │
│         │             │             │                          │
│         └─────────────┼─────────────┘                          │
│                       ▼                                        │
│                ┌─────────────┐                                 │
│                │ Master Gain │                                 │
│                └──────┬──────┘                                 │
│                       ▼                                        │
│                  [SPEAKERS]                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 2. DATA ACQUISITION LAYER

#### 2.1 Earthquake Event Fetching

```javascript
// Configuration
const MIN_MAGNITUDE = 6.0;
const FETCH_TIMESPAN_DAYS = 30;

// USGS FDSN Event Query
const url = `https://earthquake.usgs.gov/fdsnws/event/1/query
  ?format=geojson
  &starttime=${start}
  &endtime=${end}
  &minmagnitude=${MIN_MAGNITUDE}
  &orderby=time`;
```

**What it does:** Queries USGS for M6.0+ earthquakes from the past 30 days, ordered by recency.

**Data returned:** GeoJSON with magnitude, depth, location, and event ID.

#### 2.2 MiniSEED Waveform Retrieval

```javascript
const STATION_INFO = { net: "IU", sta: "ANMO", loc: "00", cha: "BHZ" };
const SEISMIC_DATA_DURATION_S = 60;

// IRIS FDSN DataSelect Query
const url = `https://service.iris.edu/fdsnws/dataselect/1/query
  ?net=${net}&sta=${sta}&loc=${loc}&cha=${cha}
  &starttime=${startTimeISO}
  &endtime=${endTimeISO}
  &format=miniseed`;
```

**Station Details:**
- **IU.ANMO**: Albuquerque, New Mexico — Global Seismographic Network station
- **BHZ**: Broadband High-gain Vertical component (20 samples/sec)
- **Duration**: 60 seconds of waveform data, starting 5 seconds before event time

#### 2.3 MiniSEED Parser

```javascript
function parseMiniSEED(buffer) {
    const view = new DataView(buffer);
    const numSamples = view.getInt16(46, false);  // Big-endian
    const dataOffset = 64;                         // Fixed header size
    
    // Extract 32-bit integer samples
    const rawData = new Int32Array(buffer.slice(dataOffset, dataOffset + numSamples * 4));
    
    // Normalize to [-1, 1] range
    const maxVal = rawData.reduce((max, val) => Math.max(max, Math.abs(val)), 0);
    const factor = 1.0 / maxVal;
    const normalized = new Float32Array(rawData.length);
    for (let i = 0; i < rawData.length; i++) {
        normalized[i] = rawData[i] * factor;
    }
    return normalized;
}
```

**Critical Insight:** The parser reads a simplified MiniSEED structure — real MiniSEED has variable-length headers and Steim compression. This parser assumes uncompressed 32-bit integer data.

---

### 3. FM SYNTHESIS ENGINE DEEP DIVE

#### 3.1 FM Presets Bank

```javascript
const FM_PRESETS = {
    carrierFreqs: [55, 82.5, 110, 147, 165, 220, 330, 440],  // Sub-bass to mid
    modIndices: [2, 5, 10, 15, 20, 30, 50, 80],              // Subtle to extreme
    harmonicRatios: [
        [1, 2, 3, 4],           // Natural harmonic series
        [1, 1.5, 2, 3],         // Fifth-heavy
        [1, 2, 4, 6],           // Octave + sixth
        [1, 2.5, 3.5, 5],       // Complex
        [1, 1.414, 2, 2.828],   // √2 based (bell-like)
        [1, 3, 5, 7],           // Odd harmonics (square-wave timbre)
        [1, 2, 3, 5],           // Natural + fifth
        [1, 1.618, 2.618, 4.236] // Golden ratio series
    ],
    waveforms: ['sine', 'triangle', 'sawtooth', 'square'],
    attackTimes: [0.01, 0.05, 0.1, 0.2, 0.5, 1.0],
    releaseTimes: [0.5, 1.0, 2.0, 3.0, 5.0, 8.0],
    filterFreqs: [200, 400, 800, 1200, 2000, 4000, 8000],
    filterQs: [0.5, 1, 2, 4, 8, 12],
    lfoRates: [0.05, 0.1, 0.2, 0.5, 1, 2, 4],
    lfoDepths: [0.01, 0.02, 0.05, 0.1, 0.2]
};
```

#### 3.2 Pseudo-Random Parameter Generation

```javascript
generateNewParams(cycleNum, magnitude, depth) {
    // Deterministic seed from cycle + seismic metadata
    const seed = cycleNum * 31 + Math.floor(magnitude * 10);
    
    // Sine-based PRNG
    const pseudoRandom = (n) => {
        const x = Math.sin(seed + n * 9999) * 10000;
        return x - Math.floor(x);  // Fractional part
    };
    
    // Seismic-influenced scaling
    const params = {
        modIndexScale: 1 + (magnitude - 6) * 0.5,  // M7 = 1.5x, M8 = 2x
        filterMod: Math.min(1, depth / 500),       // Deep quakes = more filter mod
        detuneSpread: 5 + pseudoRandom(12) * 15,   // 5-20 cents
        panSpread: 0.3 + pseudoRandom(13) * 0.7    // 30-100% stereo width
    };
}
```

**Key Insight:** Higher magnitude = more aggressive modulation. Deeper earthquakes = more filter movement.

#### 3.3 FM Operator Chain Construction

For each harmonic ratio in the preset:

```javascript
for (let i = 0; i < numCarriers; i++) {
    const ratio = params.harmonics[i];
    const freq = params.carrierFreq * ratio;
    
    // CARRIER: tone generator
    const carrier = ctx.createOscillator();
    carrier.type = params.carrierWave;
    carrier.frequency.value = freq;
    carrier.detune.value = (Math.random() - 0.5) * params.detuneSpread;
    
    // MODULATOR: modulates carrier frequency
    const modulator = ctx.createOscillator();
    modulator.type = params.modWave;
    modulator.frequency.value = freq * (1 + Math.random() * 0.5);  // Slight ratio offset
    
    // MODULATION DEPTH
    const modGain = ctx.createGain();
    modGain.gain.value = freq * params.modIndex * params.modIndexScale / (i + 1);
    // ↑ Decreases mod depth for higher harmonics
    
    // FM ROUTING: modulator → modGain → carrier.frequency
    modulator.connect(modGain);
    modGain.connect(carrier.frequency);
}
```

**FM Formula:** `output = sin(2πfc*t + I*sin(2πfm*t))`

Where:
- `fc` = carrier frequency
- `fm` = modulator frequency  
- `I` = modulation index (β)

#### 3.4 Real-Time Seismic Modulation Loop

```javascript
modulateFromSeismic() {
    const sample = this.seismicData[this.playhead];  // Current seismic sample
    
    // FILTER MODULATION
    const filterMod = this.params.filterFreq * (1 + sample * this.params.filterMod * 2);
    this.filter.frequency.setTargetAtTime(
        Math.max(100, Math.min(filterMod, 10000)),
        now, 0.05
    );
    
    // MOD INDEX MODULATION
    this.modulators.forEach((mod, i) => {
        const baseFreq = this.params.carrierFreq * this.params.harmonics[i];
        const newModGain = baseFreq * this.params.modIndex * 
                          Math.abs(sample + 0.5) * this.params.modIndexScale;
        mod.gain.gain.setTargetAtTime(newModGain, now, 0.02);
    });
    
    // PITCH MODULATION
    this.carriers.forEach(car => {
        car.osc.detune.setTargetAtTime(
            (sample * 20) + (Math.random() - 0.5) * this.params.detuneSpread,
            now, 0.05
        );
    });
    
    // Advance playhead
    this.playhead = (this.playhead + 1) % this.seismicData.length;
    
    // Loop at ~60fps
    requestAnimationFrame(() => this.modulateFromSeismic());
}
```

**Seismic ↔ Sound Mapping:**
| Seismic Feature | Audio Parameter |
|----------------|-----------------|
| Amplitude spike | Filter opens, mod index increases |
| Low amplitude | Darker, calmer tone |
| Rapid oscillation | Pitch wobble, timbral shimmer |
| DC offset | Asymmetric harmonic content |

---

### 4. EFFECTS PROCESSING

#### 4.1 Algorithmic Reverb (Convolution)

```javascript
async function createReverb() {
    const impulseLength = 3.0 * audioCtx.sampleRate;  // 3-second tail
    const impulseBuffer = audioCtx.createBuffer(2, impulseLength, audioCtx.sampleRate);
    
    for (let channel = 0; channel < 2; channel++) {
        const data = impulseBuffer.getChannelData(channel);
        for (let i = 0; i < impulseLength; i++) {
            // Exponential decay with randomized amplitude (synthetic IR)
            data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / impulseLength, 2.5);
        }
    }
    
    reverb = audioCtx.createConvolver();
    reverb.buffer = impulseBuffer;
}
```

**The `2.5` exponent:** Creates a quick initial decay followed by a long tail — more "hall" than "room."

#### 4.2 Feedback Delay Network

```javascript
function createDelay() {
    delay = audioCtx.createDelay(2.0);        // Max 2 seconds
    delay.delayTime.value = 0.4;              // 400ms default
    
    delayFeedbackGain = audioCtx.createGain();
    delayFeedbackGain.gain.value = 0.45;      // 45% feedback
    
    // Feedback loop: delay → feedbackGain → delay (recursive)
    delay.connect(delayFeedbackGain).connect(delay);
}
```

**Depth-Scaled Effects:**
```javascript
const depthFactor = Math.min(1, currentQuakeInfo.depth / 600);
reverbWetGain.gain.setTargetAtTime(0.25 + depthFactor * 0.4, ...);  // 25-65%
delay.delayTime.setTargetAtTime(0.2 + depthFactor * 0.4, ...);      // 200-600ms
delayFeedbackGain.gain.setTargetAtTime(0.35 + depthFactor * 0.25, ...);  // 35-60%
```

#### 4.3 Sub-Bass Drone Layer

```javascript
function createSubBass(rootFreq) {
    const osc = audioCtx.createOscillator();
    osc.type = 'sine';
    osc.frequency.value = rootFreq / 4;  // Two octaves below carrier
    
    const lfo = audioCtx.createOscillator();
    lfo.frequency.value = 0.03 + Math.random() * 0.02;  // 0.03-0.05 Hz
    
    const lfoGain = audioCtx.createGain();
    lfoGain.gain.value = rootFreq / 4 * 0.02;  // ±2% pitch wobble
    
    lfo.connect(lfoGain).connect(osc.frequency);
}
```

---

### 5. CYCLE MANAGEMENT SYSTEM

```javascript
const CYCLE_DURATION_S = 31;  // Prime number — avoids phase alignment

function startNewCycle() {
    cycleCount++;
    
    // Generate new parameters seeded by cycle # + earthquake metadata
    const params = fmEngine.generateNewParams(cycleCount, currentQuakeInfo.mag, currentQuakeInfo.depth);
    
    // Crossfade between cycles
    if (cycleCount > 1) {
        fmEngine.fadeOut(1.5);         // 1.5s fade
        setTimeout(() => {
            fmEngine.start(params);    // Start new after fade
        }, 1500);
    } else {
        fmEngine.start(params);
    }
}

// Schedule at 31-second intervals
setInterval(() => startNewCycle(), CYCLE_DURATION_S * 1000);
```

---

## PART TWO: GRANULAR SYNTHESIS TRANSFORMATION

---

### 6. GRANULAR FUNDAMENTALS

**What granular synthesis does:** Chops audio into tiny "grains" (1-100ms) and reassembles them with independent control over pitch, time, density, and spatialization.

**Why seismic data is perfect for granular:**
- Already irregular, textured waveforms
- Natural amplitude variations suggest grain trigger points
- Low-frequency content benefits from time-stretching
- Unpredictable patterns create organic textures

---

### 7. GRANULAR ENGINE DESIGN

#### 7.1 Core Grain Class

```javascript
class SeismicGrain {
    constructor(audioCtx, seismicBuffer, params) {
        this.ctx = audioCtx;
        this.buffer = seismicBuffer;
        
        // Grain parameters
        this.position = params.position;      // 0-1 normalized position in buffer
        this.duration = params.duration;      // Grain length in seconds
        this.pitch = params.pitch;            // Playback rate (1 = original)
        this.pan = params.pan;                // Stereo position
        this.amplitude = params.amplitude;    // Grain volume
        this.attack = params.attack;          // Envelope attack (0-0.5)
        this.release = params.release;        // Envelope release (0-0.5)
    }
    
    play(startTime) {
        const source = this.ctx.createBufferSource();
        source.buffer = this.buffer;
        source.playbackRate.value = this.pitch;
        
        // Gaussian-ish envelope via gain automation
        const env = this.ctx.createGain();
        env.gain.setValueAtTime(0, startTime);
        env.gain.linearRampToValueAtTime(
            this.amplitude, 
            startTime + this.duration * this.attack
        );
        env.gain.setValueAtTime(
            this.amplitude, 
            startTime + this.duration * (1 - this.release)
        );
        env.gain.linearRampToValueAtTime(0, startTime + this.duration);
        
        // Panning
        const panner = this.ctx.createStereoPanner();
        panner.pan.value = this.pan;
        
        // Routing
        source.connect(env).connect(panner).connect(this.ctx.destination);
        
        // Calculate buffer offset
        const startSample = Math.floor(this.position * this.buffer.length);
        source.start(startTime, startSample / this.buffer.sampleRate, this.duration);
    }
}
```

#### 7.2 Granular Cloud Scheduler

```javascript
class GranularCloud {
    constructor(audioCtx, seismicBuffer) {
        this.ctx = audioCtx;
        this.buffer = seismicBuffer;
        this.isPlaying = false;
        
        // Cloud parameters
        this.density = 20;           // Grains per second
        this.grainDuration = 0.05;   // 50ms default
        this.durationSpread = 0.5;   // ±50% randomization
        this.pitchBase = 1.0;
        this.pitchSpread = 0.1;      // ±10% pitch variation
        this.positionBase = 0;       // Playhead position
        this.positionSpread = 0.1;   // Scan window width
        this.panSpread = 0.8;        // Stereo width
        
        // Envelope shape
        this.attackRatio = 0.3;
        this.releaseRatio = 0.5;
    }
    
    start() {
        this.isPlaying = true;
        this.scheduleGrains();
    }
    
    scheduleGrains() {
        if (!this.isPlaying) return;
        
        const now = this.ctx.currentTime;
        const intervalMs = 1000 / this.density;
        
        // Generate grain with randomized parameters
        const grain = new SeismicGrain(this.ctx, this.buffer, {
            position: this.positionBase + (Math.random() - 0.5) * this.positionSpread,
            duration: this.grainDuration * (1 + (Math.random() - 0.5) * this.durationSpread),
            pitch: this.pitchBase * Math.pow(2, (Math.random() - 0.5) * this.pitchSpread),
            pan: (Math.random() - 0.5) * 2 * this.panSpread,
            amplitude: 0.3 + Math.random() * 0.2,
            attack: this.attackRatio,
            release: this.releaseRatio
        });
        
        grain.play(now);
        
        // Schedule next grain
        setTimeout(() => this.scheduleGrains(), intervalMs);
    }
    
    // Smoothly advance playhead through seismic data
    advancePlayhead(speed = 0.001) {
        this.positionBase = (this.positionBase + speed) % 1;
    }
}
```

---

### 8. SEISMIC-REACTIVE GRANULAR PARAMETERS

#### 8.1 Amplitude-to-Density Mapping

```javascript
function seismicToDensity(sample, baseValue = 20) {
    // Louder seismic = denser cloud
    const normalizedAmp = Math.abs(sample);
    return baseValue * (1 + normalizedAmp * 4);  // 20-100 grains/sec
}
```

#### 8.2 Waveform Derivative to Pitch Spread

```javascript
function seismicToSpread(currentSample, previousSample) {
    // Rate of change = pitch chaos
    const derivative = Math.abs(currentSample - previousSample);
    return 0.05 + derivative * 2;  // 5% to 205% pitch spread
}
```

#### 8.3 Zero-Crossing Detection for Grain Triggers

```javascript
function detectZeroCrossings(seismicData) {
    const triggers = [];
    for (let i = 1; i < seismicData.length; i++) {
        if (seismicData[i-1] * seismicData[i] < 0) {
            triggers.push(i / seismicData.length);  // Normalized position
        }
    }
    return triggers;  // Use as grain start points for rhythmic patterns
}
```

#### 8.4 Spectral Centroid to Filter Cutoff

```javascript
function estimateSpectralBrightness(chunk) {
    // Simple brightness estimate from sample values
    let sum = 0, weightedSum = 0;
    for (let i = 1; i < chunk.length; i++) {
        const diff = Math.abs(chunk[i] - chunk[i-1]);  // High-freq indicator
        sum += Math.abs(chunk[i]);
        weightedSum += diff * i;
    }
    return weightedSum / (sum + 0.001);  // Higher = brighter
}
```

---

### 9. ADVANCED GRANULAR TECHNIQUES

#### 9.1 Grain Freezing (Spectral Freeze)

```javascript
class FreezeEngine {
    constructor(cloud) {
        this.cloud = cloud;
        this.frozenPosition = null;
    }
    
    freeze() {
        // Lock playhead, increase density for sustained tone
        this.frozenPosition = this.cloud.positionBase;
        this.cloud.positionSpread = 0.01;  // Tiny window
        this.cloud.density = 100;          // Dense cloud
        this.cloud.grainDuration = 0.1;    // Longer grains for smoother tone
    }
    
    unfreeze() {
        this.cloud.positionSpread = 0.1;
        this.cloud.density = 20;
        this.cloud.grainDuration = 0.05;
    }
}
```

#### 9.2 Time-Stretching via Grain Overlap

```javascript
function timeStretch(cloud, stretchFactor) {
    // stretchFactor > 1 = slower playback
    // Increase grain overlap to maintain continuity
    
    cloud.grainDuration = 0.05 * stretchFactor;
    cloud.density = 20 * stretchFactor;
    cloud.attackRatio = 0.4;   // Smoother overlap
    cloud.releaseRatio = 0.4;
    
    // Slow down playhead advancement
    const advanceSpeed = 0.001 / stretchFactor;
    setInterval(() => cloud.advancePlayhead(advanceSpeed), 16);
}
```

#### 9.3 Pitch Quantization (Harmonic Locking)

```javascript
const SCALE_RATIOS = {
    chromatic: [1, 1.059, 1.122, 1.189, 1.26, 1.335, 1.414, 1.498, 1.587, 1.682, 1.782, 1.888],
    pentatonic: [1, 1.122, 1.26, 1.498, 1.682],
    fifths: [1, 1.5, 2.25, 3.375],
    harmonic: [1, 2, 3, 4, 5, 6, 7, 8]
};

function quantizePitch(rawPitch, scale = 'pentatonic') {
    const ratios = SCALE_RATIOS[scale];
    // Find nearest ratio
    let nearest = ratios[0];
    let minDist = Math.abs(rawPitch - nearest);
    
    for (const ratio of ratios) {
        const dist = Math.abs(rawPitch - ratio);
        if (dist < minDist) {
            minDist = dist;
            nearest = ratio;
        }
    }
    return nearest;
}
```

#### 9.4 Probability-Based Grain Triggering

```javascript
class ProbabilisticCloud extends GranularCloud {
    constructor(audioCtx, seismicBuffer) {
        super(audioCtx, seismicBuffer);
        this.triggerProbability = 0.7;  // 70% chance per scheduled grain
    }
    
    scheduleGrains() {
        if (!this.isPlaying) return;
        
        // Seismic amplitude influences probability
        const currentSample = this.buffer.getChannelData(0)[
            Math.floor(this.positionBase * this.buffer.length)
        ];
        const dynProb = this.triggerProbability + Math.abs(currentSample) * 0.3;
        
        if (Math.random() < dynProb) {
            // Trigger grain
            super.scheduleGrains();
        } else {
            // Skip grain, schedule next check
            setTimeout(() => this.scheduleGrains(), 1000 / this.density);
        }
    }
}
```

---

### 10. COMPLETE SEISMIC ELECTRONICA MACHINE

#### 10.1 Full System Integration

```javascript
class SeismicElectronicaMachine {
    constructor(audioCtx) {
        this.ctx = audioCtx;
        this.seismicBuffer = null;
        this.seismicData = [];
        
        // Multiple granular voices
        this.clouds = [];
        this.fmEngine = null;
        this.subBass = null;
        
        // Effects chain
        this.reverb = null;
        this.delay = null;
        this.compressor = null;
        this.masterGain = null;
        
        // Modulation sources
        this.seismicLFO = { rate: 0, depth: 0 };
        this.playhead = 0;
    }
    
    async initialize(seismicBuffer) {
        this.seismicBuffer = seismicBuffer;
        this.seismicData = Array.from(seismicBuffer.getChannelData(0));
        
        // Create effects chain
        this.masterGain = this.ctx.createGain();
        this.masterGain.gain.value = 0.8;
        
        this.compressor = this.ctx.createDynamicsCompressor();
        this.compressor.threshold.value = -24;
        this.compressor.ratio.value = 4;
        this.compressor.attack.value = 0.003;
        this.compressor.release.value = 0.25;
        
        await this.createReverb();
        this.createDelay();
        
        // Routing
        this.compressor.connect(this.masterGain);
        this.masterGain.connect(this.ctx.destination);
        
        // Create synthesis layers
        this.createGranularLayers();
        this.createFMLayer();
        this.createSubBassLayer();
    }
    
    createGranularLayers() {
        // Layer 1: Texture cloud (high density, short grains)
        const textureCloud = new GranularCloud(this.ctx, this.seismicBuffer);
        textureCloud.density = 40;
        textureCloud.grainDuration = 0.03;
        textureCloud.pitchBase = 2;  // Octave up
        textureCloud.panSpread = 1.0;
        
        // Layer 2: Tone cloud (low density, long grains)
        const toneCloud = new GranularCloud(this.ctx, this.seismicBuffer);
        toneCloud.density = 8;
        toneCloud.grainDuration = 0.2;
        toneCloud.pitchBase = 0.5;  // Octave down
        toneCloud.panSpread = 0.5;
        
        // Layer 3: Rhythm cloud (triggered by zero-crossings)
        const rhythmCloud = new ProbabilisticCloud(this.ctx, this.seismicBuffer);
        rhythmCloud.density = 16;
        rhythmCloud.grainDuration = 0.08;
        rhythmCloud.pitchBase = 1;
        
        this.clouds = [textureCloud, toneCloud, rhythmCloud];
    }
    
    startSeismicModulation() {
        const modulate = () => {
            if (!this.isPlaying) return;
            
            const sample = this.seismicData[this.playhead];
            
            // Modulate granular parameters
            this.clouds[0].density = 30 + Math.abs(sample) * 50;
            this.clouds[1].pitchSpread = 0.1 + Math.abs(sample) * 0.5;
            this.clouds[2].triggerProbability = 0.5 + sample * 0.3;
            
            // Modulate effects
            const filterMod = 2000 + sample * 3000;
            // Apply to filter...
            
            this.playhead = (this.playhead + 1) % this.seismicData.length;
            requestAnimationFrame(modulate);
        };
        modulate();
    }
    
    start() {
        this.isPlaying = true;
        this.clouds.forEach(cloud => cloud.start());
        this.startSeismicModulation();
    }
    
    stop() {
        this.isPlaying = false;
        this.clouds.forEach(cloud => cloud.isPlaying = false);
    }
}
```

---

### 11. PARAMETER MAPPING CHEAT SHEET

| Seismic Feature | Granular Parameter | Musical Effect |
|-----------------|-------------------|----------------|
| **Amplitude** | Grain density | Texture thickness |
| **Amplitude** | Master volume | Dynamic swells |
| **Rate of change** | Pitch spread | Harmonic chaos |
| **Zero-crossings** | Grain triggers | Rhythmic pulses |
| **Low-frequency content** | Grain duration | Drone vs. texture |
| **Peak detection** | Freeze events | Spectral holds |
| **RMS energy** | Filter cutoff | Brightness |
| **Spectral centroid** | Reverb wet mix | Spatial depth |
| **Crest factor** | Compressor threshold | Punch vs. sustain |

---

### 12. SUGGESTED UI EXTENSIONS

```
┌─────────────────────────────────────────────────────┐
│  SEISMIC GRANULAR CONTROL PANEL                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  [WAVEFORM DISPLAY — Seismic data with playhead]   │
│  ▔▔▔▔▔▔▁▁▁▁▔▔▔▔▔▁▁▁▔▔▔▔▔▔▔▔▔▔▁▁▁▁▁▔▔▔▔▔▔▁▁▁▁▁▁▔▔   │
│                    ▲ playhead                       │
│                                                     │
│  DENSITY ████████░░░░░ 45 g/s    DURATION ███░░ 80ms│
│  PITCH   ████░░░░░░░░░ 0.8x      SPREAD   █████ 40% │
│  FREEZE  [OFF]  QUANTIZE [PENT]  STRETCH  [2.0x]   │
│                                                     │
│  LAYERS:  [✓] Texture  [✓] Tone  [ ] Rhythm        │
│                                                     │
│  SEISMIC MAPPING:                                   │
│  Amp→Density [████░░]   Δ→Spread [██░░░░]          │
│  ZeroCross→Rhythm [ON]  RMS→Filter [███░░]         │
│                                                     │
│  [START] [STOP] [FETCH NEW QUAKE] [EXPORT WAV]     │
└─────────────────────────────────────────────────────┘
```

---

### 13. CREATIVE PRESETS

**PRESET 1: "Tectonic Drone"**
```javascript
{ density: 60, duration: 0.15, pitch: 0.25, spread: 0.05, freeze: true }
```

**PRESET 2: "Aftershock Rhythm"**
```javascript
{ density: 16, duration: 0.04, pitch: 1, spread: 0.3, quantize: 'pentatonic' }
```

**PRESET 3: "Subduction Zone"**
```javascript
{ density: 100, duration: 0.008, pitch: 4, spread: 0.8, panSpread: 1 }
```

**PRESET 4: "P-Wave Pulse"**
```javascript
{ density: 8, duration: 0.5, pitch: 0.5, stretch: 4, reverb: 0.7 }
```

---

## SUMMARY

The original THREE.html uses **FM synthesis** with seismic data modulating operator parameters in real-time. To transform it into a **granular electronica machine**:

1. **Replace FM operators** with grain schedulers
2. **Map seismic amplitude** to grain density and filter cutoff
3. **Map seismic derivative** to pitch spread and stereo width
4. **Use zero-crossings** as rhythmic grain triggers
5. **Add time-stretching** for slow-motion seismic exploration
6. **Implement freeze** for spectral sustain on earthquake peaks
7. **Layer multiple clouds** at different pitch/density ratios
8. **Quantize pitches** to musical scales for harmonic coherence

The result: **living, breathing textures** that capture the sonic character of Earth's tectonic activity while remaining musically compelling.
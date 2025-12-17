# SOS Algorithmic Composition Design
## Granular & FM Synthesis for Seismic Sonification
### JavaScript / Web Audio API Only

---
**MiniSEED seismic data** (0.01–20 Hz) → **Audible sound** (20–20,000 Hz)

Two synthesis engines. Pure Web Audio. No dependencies.

Claude OPUS 4.5 Analysis
Dec 17, 2025

---

## 1. GRANULAR SYNTHESIS

### Concept
Slice seismic waveform into micro-fragments ("grains"), scatter and layer them to create texture.

```
seismic_stream → [grain][grain][grain]... → cloud of sound
```

### Why It Works for Seismic
- Preserves the *texture* of earth vibration
- Pitch-shifts without losing character
- Natural for "rumble" and "drone" aesthetics
- Grain density can mirror earthquake energy decay

### Parameters

| Parameter | Seismic Mapping | Range |
|-----------|-----------------|-------|
| Grain Size | Wave period | 10–100ms |
| Grain Density | Event intensity | 1–100 grains/sec |
| Pitch Shift | Frequency transpose | 0.5–4.0× |
| Pan Spread | Station location | -1 to +1 |
| Amplitude | Waveform magnitude | 0–1 |

### JavaScript Implementation

```javascript
class GranularEngine {
  constructor(audioContext) {
    this.ctx = audioContext;
    this.grainSize = 0.05;      // 50ms default
    this.grainDensity = 20;     // grains per second
    this.pitchShift = 2.0;      // octave up
    this.spread = 0.5;          // stereo spread
    this.buffer = null;
    this.isPlaying = false;
  }

  // Load seismic data as audio buffer
  async loadSeismicData(seismicArray, sampleRate = 44100) {
    const buffer = this.ctx.createBuffer(1, seismicArray.length, sampleRate);
    const channelData = buffer.getChannelData(0);
    
    // Normalize seismic data to [-1, 1]
    const max = Math.max(...seismicArray.map(Math.abs));
    for (let i = 0; i < seismicArray.length; i++) {
      channelData[i] = seismicArray[i] / max;
    }
    
    this.buffer = buffer;
  }

  // Trigger single grain
  triggerGrain(time) {
    if (!this.buffer) return;

    const source = this.ctx.createBufferSource();
    const gain = this.ctx.createGain();
    const panner = this.ctx.createStereoPanner();

    // Random position in buffer
    const startPos = Math.random() * (this.buffer.duration - this.grainSize);
    
    source.buffer = this.buffer;
    source.playbackRate.value = this.pitchShift;

    // Grain envelope (attack-sustain-release)
    const attack = this.grainSize * 0.3;
    const release = this.grainSize * 0.3;
    
    gain.gain.setValueAtTime(0, time);
    gain.gain.linearRampToValueAtTime(0.8, time + attack);
    gain.gain.linearRampToValueAtTime(0, time + this.grainSize);

    // Random pan within spread
    panner.pan.value = (Math.random() * 2 - 1) * this.spread;

    // Connect
    source.connect(gain);
    gain.connect(panner);
    panner.connect(this.ctx.destination);

    source.start(time, startPos, this.grainSize);
  }

  // Continuous grain stream
  start() {
    this.isPlaying = true;
    this.scheduleGrains();
  }

  scheduleGrains() {
    if (!this.isPlaying) return;

    const now = this.ctx.currentTime;
    const interval = 1 / this.grainDensity;

    // Schedule ahead
    for (let i = 0; i < 10; i++) {
      this.triggerGrain(now + i * interval);
    }

    setTimeout(() => this.scheduleGrains(), interval * 10 * 1000);
  }

  stop() {
    this.isPlaying = false;
  }

  // Electronica Attenuation: decay grain density over time
  applyAttenuation(decayTime = 10) {
    const startDensity = this.grainDensity;
    const startTime = this.ctx.currentTime;

    const decay = () => {
      if (!this.isPlaying) return;
      
      const elapsed = this.ctx.currentTime - startTime;
      this.grainDensity = startDensity * Math.exp(-elapsed / decayTime);
      
      if (this.grainDensity > 0.5) {
        requestAnimationFrame(decay);
      }
    };
    
    decay();
  }
}
```

### Usage

```javascript
const ctx = new AudioContext();
const granular = new GranularEngine(ctx);

// seismicData = array of floats from MiniSEED parse
await granular.loadSeismicData(seismicData);

granular.grainSize = 0.03;    // 30ms grains
granular.grainDensity = 40;   // dense texture
granular.pitchShift = 4.0;    // 2 octaves up

granular.start();
granular.applyAttenuation(15); // 15 second decay
```

---

## 2. FM SYNTHESIS

### Concept
Use seismic waveform as **modulator** to bend the frequency of a **carrier** oscillator.

```
carrier_freq × (1 + seismic_sample × mod_index) → complex timbre
```

### Why It Works for Seismic
- Creates metallic, bell-like, grinding timbres
- Seismic complexity → harmonic complexity
- Real-time responsive to live data
- Low CPU, high expressiveness

### Parameters

| Parameter | Seismic Mapping | Range |
|-----------|-----------------|-------|
| Carrier Freq | Base pitch | 50–2000 Hz |
| Mod Index | Seismic intensity | 0–100 |
| Mod Freq | Derived from seismic rate | 0.1–50 Hz |
| Envelope | Amplitude shape | ADSR |

### JavaScript Implementation

```javascript
class FMSeismicSynth {
  constructor(audioContext) {
    this.ctx = audioContext;
    this.carrierFreq = 220;     // A3
    this.modIndex = 10;         // modulation depth
    this.seismicData = [];
    this.playhead = 0;
    this.isPlaying = false;

    // Create nodes
    this.carrier = this.ctx.createOscillator();
    this.modulator = this.ctx.createOscillator();
    this.modGain = this.ctx.createGain();
    this.outputGain = this.ctx.createGain();

    // FM routing
    this.modulator.connect(this.modGain);
    this.modGain.connect(this.carrier.frequency);
    this.carrier.connect(this.outputGain);
    this.outputGain.connect(this.ctx.destination);

    this.carrier.type = 'sine';
    this.modulator.type = 'sine';
  }

  // Load seismic array
  loadSeismicData(seismicArray) {
    // Normalize to [-1, 1]
    const max = Math.max(...seismicArray.map(Math.abs));
    this.seismicData = seismicArray.map(s => s / max);
  }

  // Set base frequency
  setCarrierFreq(freq) {
    this.carrierFreq = freq;
    this.carrier.frequency.value = freq;
  }

  // Set modulation intensity
  setModIndex(index) {
    this.modIndex = index;
  }

  start() {
    this.carrier.frequency.value = this.carrierFreq;
    this.modulator.frequency.value = 1; // Will be controlled by seismic rate
    this.modGain.gain.value = this.carrierFreq * this.modIndex;
    this.outputGain.gain.value = 0.5;

    this.carrier.start();
    this.modulator.start();
    this.isPlaying = true;

    this.processSeismic();
  }

  // Real-time seismic modulation
  processSeismic() {
    if (!this.isPlaying || this.seismicData.length === 0) return;

    const sample = this.seismicData[this.playhead];
    
    // Modulate carrier frequency based on seismic sample
    const modAmount = this.carrierFreq * this.modIndex * sample;
    this.modGain.gain.setTargetAtTime(
      Math.abs(modAmount),
      this.ctx.currentTime,
      0.01
    );

    // Advance playhead (loop)
    this.playhead = (this.playhead + 1) % this.seismicData.length;

    // Schedule next update (~60fps)
    if (this.isPlaying) {
      requestAnimationFrame(() => this.processSeismic());
    }
  }

  stop() {
    this.isPlaying = false;
    this.outputGain.gain.setTargetAtTime(0, this.ctx.currentTime, 0.1);
  }
}
```

### Advanced: Multi-Carrier FM

```javascript
class MultiFMSeismic {
  constructor(audioContext) {
    this.ctx = audioContext;
    this.carriers = [];
    this.ratios = [1, 2, 3, 4.5, 6]; // harmonic ratios
    this.baseFreq = 110;
    
    this.masterGain = this.ctx.createGain();
    this.masterGain.connect(this.ctx.destination);
  }

  init(seismicData) {
    this.ratios.forEach((ratio, i) => {
      const osc = this.ctx.createOscillator();
      const gain = this.ctx.createGain();
      
      osc.frequency.value = this.baseFreq * ratio;
      osc.type = 'sine';
      gain.gain.value = 0.2 / (i + 1); // decreasing amplitude
      
      osc.connect(gain);
      gain.connect(this.masterGain);
      
      this.carriers.push({ osc, gain, ratio });
    });

    this.seismicData = this.normalize(seismicData);
  }

  normalize(data) {
    const max = Math.max(...data.map(Math.abs));
    return data.map(d => d / max);
  }

  start() {
    this.carriers.forEach(c => c.osc.start());
    this.isPlaying = true;
    this.playhead = 0;
    this.modulate();
  }

  modulate() {
    if (!this.isPlaying) return;

    const sample = this.seismicData[this.playhead];
    const modFactor = 1 + sample * 0.5; // ±50% frequency deviation

    this.carriers.forEach(c => {
      c.osc.frequency.setTargetAtTime(
        this.baseFreq * c.ratio * modFactor,
        this.ctx.currentTime,
        0.02
      );
    });

    this.playhead = (this.playhead + 1) % this.seismicData.length;
    
    if (this.isPlaying) {
      setTimeout(() => this.modulate(), 16); // ~60fps
    }
  }

  stop() {
    this.isPlaying = false;
    this.masterGain.gain.setTargetAtTime(0, this.ctx.currentTime, 0.5);
  }
}
```

---

## 3. HYBRID: Granular + FM

Combine both for maximum seismic expression.

```javascript
class SeismicHybridEngine {
  constructor(audioContext) {
    this.ctx = audioContext;
    this.granular = new GranularEngine(audioContext);
    this.fm = new FMSeismicSynth(audioContext);
    
    // Crossfade control
    this.granularGain = this.ctx.createGain();
    this.fmGain = this.ctx.createGain();
    this.master = this.ctx.createGain();
    
    this.granularGain.connect(this.master);
    this.fmGain.connect(this.master);
    this.master.connect(this.ctx.destination);
  }

  async load(seismicData) {
    await this.granular.loadSeismicData(seismicData);
    this.fm.loadSeismicData(seismicData);
  }

  // Crossfade between granular and FM based on magnitude
  setMix(granularAmount) {
    // 0 = all FM, 1 = all granular
    this.granularGain.gain.value = granularAmount;
    this.fmGain.gain.value = 1 - granularAmount;
  }

  // Auto-mix based on seismic intensity
  autoMix(seismicData) {
    const rms = Math.sqrt(
      seismicData.reduce((sum, s) => sum + s * s, 0) / seismicData.length
    );
    
    // High energy = more FM, low energy = more granular
    const mix = 1 - Math.min(rms * 10, 1);
    this.setMix(mix);
  }

  start() {
    this.granular.start();
    this.fm.start();
  }

  stop() {
    this.granular.stop();
    this.fm.stop();
  }
}
```

---

## 4. Quick Reference

### Granular Parameters by Seismic Event

| Event Type | Grain Size | Density | Pitch | Character |
|------------|------------|---------|-------|-----------|
| Microseism | 80ms | 5/sec | 1.5× | Slow drone |
| Local quake | 30ms | 40/sec | 3× | Dense rumble |
| Teleseism | 50ms | 15/sec | 2× | Filtered wash |
| Volcanic | 20ms | 60/sec | 4× | Aggressive texture |

### FM Parameters by Magnitude

| Magnitude | Carrier | Mod Index | Character |
|-----------|---------|-----------|-----------|
| M2–3 | 440 Hz | 2 | Gentle shimmer |
| M4–5 | 220 Hz | 10 | Metallic tone |
| M6–7 | 110 Hz | 30 | Bell/gong |
| M8+ | 55 Hz | 100 | Grinding chaos |

---

## 5. MiniSEED to Array (Minimal)

```javascript
// Simplified MiniSEED parse (BHZ channel)
// For production, use a proper library

async function fetchSeismicData(url) {
  const response = await fetch(url);
  const buffer = await response.arrayBuffer();
  const view = new DataView(buffer);
  
  const samples = [];
  const headerSize = 64; // simplified
  const dataStart = headerSize;
  
  // Assuming 32-bit float samples
  for (let i = dataStart; i < buffer.byteLength; i += 4) {
    samples.push(view.getFloat32(i, true));
  }
  
  return samples;
}
```

---

## Summary

| Engine | Best For | CPU | Complexity |
|--------|----------|-----|------------|
| Granular | Texture, ambient, decay | Medium | Medium |
| FM | Tonal, reactive, timbral | Low | Low |
| Hybrid | Full expression | Medium | High |

**Start simple**: FM for tonal response, Granular for texture. Layer them.

---

*Pure Web Audio. No dependencies. No MIDI. Just seismic → sound.*

# ShadowZone.html — Technical Analysis

## System Overview

ShadowZone is a browser-based seismic waveform synthesizer that transforms real-time earthquake data into generative electronica music. The system fetches live seismic data from USGS and EarthScope services, parses MiniSEED format waveforms, and uses the data to modulate a three-layer granular synthesis engine.

---

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           DATA PIPELINE                                  │
├─────────────────────────────────────────────────────────────────────────┤
│  USGS Earthquake API  →  EarthScope MiniSEED  →  parseMiniSEED()       │
│       (GeoJSON)              (Binary)              (Float32Array)       │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                         SYNTHESIS ENGINE                                 │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐                │
│  │ TextureCloud │   │  ToneCloud   │   │ RhythmCloud  │                │
│  │   (Sine)     │   │ (Triangle)   │   │ (Sawtooth)   │                │
│  │ 45 grains/s  │   │  6 grains/s  │   │ 14 grains/s  │                │
│  └──────────────┘   └──────────────┘   └──────────────┘                │
│          ↓                  ↓                  ↓                         │
│     ┌─────────────────────────────────────────────────────┐             │
│     │              GranularEngine Master Filter            │             │
│     │                 (Lowpass 1-10kHz)                    │             │
│     └─────────────────────────────────────────────────────┘             │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                          EFFECTS CHAIN                                   │
├─────────────────────────────────────────────────────────────────────────┤
│   DynamicsCompressor → ConvolutionReverb → StereoDelay → MasterGain    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## MiniSEED Data Handling

### Compression Answer: **Neither Steim 1 nor Steim 2**

The `parseMiniSEED()` function (lines 1328-1354) implements a **simplified, non-decompressing parser** that assumes uncompressed 32-bit integer data. Here's the critical analysis:

### Parser Implementation

```javascript
function parseMiniSEED(buffer) {
    const view = new DataView(buffer);
    if (buffer.byteLength < 64) throw new Error("Buffer too small");
    
    // Read sample count from byte offset 46 (big-endian Int16)
    const numSamples = view.getInt16(46, false);
    
    // Fixed data offset assumption
    const dataOffset = 64;
    
    // Direct Int32 read - NO DECOMPRESSION
    const rawData = new Int32Array(buffer.slice(dataOffset, dataOffset + numSamples * 4));
    
    // Normalize to Float32
    const maxVal = rawData.reduce((m, v) => Math.max(m, Math.abs(v)), 0);
    const normalized = new Float32Array(rawData.length);
    for (let i = 0; i < rawData.length; i++) 
        normalized[i] = rawData[i] / maxVal;
    
    return normalized;
}
```

### What This Parser Does

| Operation | Implementation | Notes |
|-----------|---------------|-------|
| Header Size | Fixed 64 bytes | Standard MiniSEED fixed header |
| Sample Count | Byte 46-47 (Int16 BE) | Correct location per SEED spec |
| Data Format | Raw Int32Array | **No compression handling** |
| Encoding Check | None | Ignores byte 39 (encoding format) |
| Blockette Parsing | None | Assumes no blockettes |

### What Real Steim Compression Requires

**Steim 1** encoding (format code 10) uses:
- 64-byte frames with a control word
- Variable-width differences (8, 16, or 32 bits)
- Delta encoding from initial sample value
- Bit-packing with nibble-based decoding

**Steim 2** encoding (format code 11) adds:
- Additional difference sizes (4, 5, 6 bits per sample)
- More aggressive compression ratios
- Same frame structure with different control codes

### Parser Limitations

1. **No encoding detection** — The parser doesn't read byte 39 to determine the actual encoding format
2. **No decompression** — Simply reads bytes as Int32, which would produce garbage for Steim-compressed data
3. **Fixed offset assumption** — Doesn't account for variable-length blockettes
4. **Single record only** — Doesn't handle multiple 512-4096 byte records in a stream

### Fallback Mechanism

When parsing fails (which it likely will for Steim data), the code generates **synthetic seismic data**:

```javascript
catch (e) {
    const fb = new Float32Array(8192);
    for (let i = 0; i < fb.length; i++) {
        // P-wave arrival simulation
        const p = Math.sin(i * 0.15) * Math.exp(-i / 1500);
        // S-wave arrival (~2000 samples later)
        const s = i > 2000 ? Math.sin((i - 2000) * 0.08) * Math.exp(-(i - 2000) / 3000) * 1.5 : 0;
        // Surface wave arrival (~4000 samples later)
        const surf = i > 4000 ? (Math.sin((i - 4000) * 0.03) + 
                                  Math.sin((i - 4000) * 0.05) * 0.5) * 
                                  Math.exp(-(i - 4000) / 5000) * 0.8 : 0;
        fb[i] = p + s + surf + (Math.random() - 0.5) * 0.05;
    }
    return fb;  // Returns synthetic waveform
}
```

This fallback creates a scientifically-plausible earthquake waveform with:
- **P-wave**: Early, high-frequency arrival (0.15 rad/sample)
- **S-wave**: Delayed, lower-frequency arrival (0.08 rad/sample)  
- **Surface waves**: Late, long-period arrivals with multiple frequencies
- **Ambient noise**: ±2.5% random noise floor

---

## Data Sources

### Earthquake Event Data
```
URL: https://earthquake.usgs.gov/fdsnws/event/1/query
Parameters:
  - format=geojson
  - minmagnitude=6.0
  - orderby=time
  - starttime/endtime (30-day window)
```

### Waveform Data
```
URL: https://service.earthscope.org/fdsnws/dataselect/1/query
Station: IU.ANMO.00.BHZ (Albuquerque, New Mexico)
Format: MiniSEED
Duration: 60 seconds starting 5s before event origin
```

---

## Granular Synthesis Architecture

### Three-Layer Cloud System

| Layer | Waveform | Base Density | Grain Duration | Role |
|-------|----------|--------------|----------------|------|
| **Texture** | Sine | 45/sec | 35ms | Ambient shimmer, high-frequency sparkle |
| **Tone** | Triangle | 6/sec | 300ms | Melodic foundation, sustained pads |
| **Rhythm** | Sawtooth | 14/sec | 55ms | Percussive triggers at zero-crossings |

### Seismic-to-Audio Parameter Mapping

The `GranularEngine.modulateFromSeismic()` function maps waveform features to synthesis parameters:

```javascript
// Seismic features extracted
const sample = seismicData[idx];           // Raw amplitude
const deriv = calculateDerivative(idx);    // Local rate of change
const rms = calculateRMS(idx, 50);         // 50-sample RMS window

// Master filter modulation
targetFilter = 2000 + |sample| * 6000;     // 2-8kHz range

// Texture cloud modulation
textureCloud.density = 35 + |sample| * 60;
textureCloud.frequencySpread = 0.4 + deriv * 15;
textureCloud.panSpread = 0.5 + deriv * 8;

// Tone cloud modulation  
toneCloud.density = 5 + rms * 10;
toneCloud.frequencySpread = 0.2 + deriv * 8;

// Rhythm cloud modulation
rhythmCloud.triggerProbability = 0.35 + rms * 0.5;
```

### Scale Quantization

All grain frequencies are quantized to musical scales:

```javascript
const SCALES = {
    pentatonic: [0, 2, 4, 7, 9],
    minor: [0, 2, 3, 5, 7, 8, 10],
    dorian: [0, 2, 3, 5, 7, 9, 10],
    harmonic: [0, 12, 19, 24, 28, 31, 34, 36],  // Overtone series
    fifths: [0, 7, 14, 21, 28]                   // Circle of fifths
};
```

---

## Effects Processing

### Dynamics Compressor
- Threshold: -12dB
- Ratio: 3:1
- Attack: 5ms
- Release: 200ms

### Convolution Reverb (Algorithmic IR)
- Length: 3.5 seconds
- Early reflections: First 2000 samples with sinusoidal modulation
- Decay curve: Quadratic (power 2.3)
- Wet level: 45% (depth-modulated)

### Stereo Delay
- Time: 375ms base (depth-modulated 300-650ms)
- Feedback: 38% (depth-modulated to 60%)
- Lowpass filter: 2200Hz on feedback path
- Wet level: 28%

### Depth-Based Processing
Earthquake depth modulates atmospheric effects:
```javascript
const depthFactor = Math.min(1, quakeInfo.depth / 600);
reverbWet = 0.35 + depthFactor * 0.35;      // 35-70%
delayTime = 0.3 + depthFactor * 0.35;       // 300-650ms
delayFeedback = 0.35 + depthFactor * 0.25;  // 35-60%
```

---

## Shadow Zone Visualization

The `ShadowZoneVisualizer` class implements P-wave ray tracing through Earth's interior:

### Geometric Model
- Earth radius: 38% of canvas minimum dimension
- Outer core ratio: 54.5% of Earth radius
- Inner core ratio: 19% of Earth radius
- Shadow zone: 103° - 142° from epicenter

### Ray Tracing Algorithm
```javascript
traceRay(takeoff, coreR) {
    // 1. Cast ray from epicenter at takeoff angle
    // 2. Test intersection with core boundary
    // 3. If hits core: calculate refraction exit point
    // 4. If misses core: calculate surface emergence
    // 5. Determine if exit falls in shadow zone (103°-142°)
}
```

### Animation System
- Wavefronts advance continuously at 0.00111 units/frame
- Three staggered wavefront phases for smooth animation
- MUZAK mode adds floating musical note particles (♪♫🎵🎶)

---

## Feature Summary

| Feature | Implementation |
|---------|---------------|
| Mobile Support | Responsive CSS, touch events, iOS audio unlock |
| Scale Selection | 5 musical scales (Pentatonic, Minor, Dorian, Harmonic, Fifths) |
| Root Note | 5 options (A1, D2, E2, A2, D3) |
| Time Stretch | 0.25× to 3× playback speed |
| MUZAK Mode | Freeze playhead, increase density, display note indicators |
| Real-time Metrics | Texture/Tone/Rhythm density, filter frequency |

---

## Conclusion

ShadowZone uses a **simplified MiniSEED parser that does not implement Steim compression decoding**. The parser assumes uncompressed 32-bit integer format and falls back to synthetic seismogram generation when parsing fails. For production use with actual FDSN waveform services (which typically use Steim 1 or Steim 2 compression), a proper decompression implementation would be needed—or requesting uncompressed format explicitly from the service.

The synthesis architecture elegantly maps seismic waveform characteristics (amplitude, derivative, RMS energy) to granular synthesis parameters, creating a musically coherent representation of earthquake data through scale quantization and layered cloud structures.

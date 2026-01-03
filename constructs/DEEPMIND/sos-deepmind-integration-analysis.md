# SOS × DeepMind Live Music Models Integration Analysis

## Executive Summary

The **Live Music Models** paper from Google DeepMind introduces a paradigm perfectly aligned with seismic sonification: real-time, continuous audio generation with synchronized user control. The SOS (Sounds of Seismic) project's granular synthesis architecture, driven by live earthquake data, represents an ideal candidate for enhancement with DeepMind's codec language modeling approach.

This analysis identifies **7 DeepMind technologies** directly applicable to SOS and provides implementation pathways for each.

---

## Current SOS Architecture (ShadowZone)

Based on the ShadowZone implementation:

| Component | Current Implementation |
|-----------|----------------------|
| **Input** | Real-time seismic waveforms (USGS/EarthScope) |
| **Synthesis** | Three-layer granular synthesis (Texture/Tone/Rhythm) |
| **Control** | Scale selection, root note, stretch, reverb, filter |
| **Visualization** | Shadow zone P-wave ray tracing |
| **Output** | Browser-based Web Audio API |

---

## DeepMind Technologies Applicable to SOS

### 1. Magenta RealTime (Open-Weights Model)

**What it is:** 750M parameter autoregressive transformer generating 48kHz stereo audio in 2-second chunks, conditioned on 10 seconds of audio context + style embeddings.

**Integration with SOS:**

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│ Seismic Waveform│───▶│ MusicCoCa Embed  │───▶│ Magenta RT LM   │
│ (EarthScope)    │    │ (Style Vector)   │    │ (Audio Tokens)  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
        │                      │                       │
        ▼                      ▼                       ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│ Granular Buffer │───▶│ Weighted Blend   │───▶│ SpectroStream   │
│ (Current SOS)   │    │ + Text Prompts   │    │ Audio Decoder   │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

**How to implement:**
1. Run Magenta RT on Colab TPU (free tier) or local GPU (40GB VRAM)
2. Use seismic waveform chunks as audio prompts via MusicCoCa
3. Blend with text prompts describing desired electronica style
4. Feed output back to browser via WebSocket streaming

**Code pattern:**
```python
from magenta_rt import audio, system, musiccoca

mrt = system.MagentaRT()
style_model = musiccoca.MusicCoCa()

# Seismic waveform as audio prompt
seismic_audio = audio.Waveform.from_buffer(seismic_data, sample_rate=100)
seismic_audio_resampled = seismic_audio.resample(48000)  # Upscale from seismic Hz

weighted_styles = [
    (2.0, seismic_audio_resampled),  # Seismic "character"
    (1.5, 'ambient drone'),           # Genre foundation
    (1.0, 'deep bass'),               # Electronica element
    (0.5, earthquake_magnitude_descriptor)  # Dynamic text
]

# Generate continuous stream
for chunk in mrt.stream(style=blend_styles(weighted_styles)):
    yield chunk.samples
```

**Benefits:**
- Transforms raw seismic oscillation patterns into musically coherent output
- Preserves "character" of earthquake via audio embedding
- Text prompts allow artistic control while seismic data drives uniqueness

---

### 2. MusicCoCa (Joint Audio-Text Embedding)

**What it is:** Contrastive captioner mapping both audio and text into shared 768-dimensional embedding space. 12-layer ViT for audio, 12-layer Transformer for text.

**Integration with SOS:**

This is the **critical bridge** between seismic data and musical generation.

**Seismic-to-Embedding Pipeline:**
```
Raw Seismic      Frequency         Log-mel          MusicCoCa        Style
Waveform    ───▶ Upscaling    ───▶ Spectrogram ───▶ Audio Tower ───▶ Vector
(0.01-10Hz)      (×1000)           (128 channels)    (12-layer ViT)   (768d)
```

**Implementation approach:**

1. **Frequency transposition:** Shift seismic frequencies (typically 0.01-10 Hz) into audible range (20-20kHz) using accelerated playback or pitch shifting
2. **Spectrogram generation:** Create 10-second log-mel spectrograms from transposed waveforms
3. **Embedding extraction:** Use MusicCoCa audio tower to generate style vectors
4. **Embedding arithmetic:** Blend seismic embeddings with genre text embeddings

**Key insight from paper:** *"A weighted sum of embed('techno') and embed('flute') gives a good approximation of embed('techno flute')"*

**For SOS:**
```python
# Earthquake "personality" via embedding arithmetic
eq_style = (
    0.4 * musiccoca.embed(seismic_audio) +      # Earth's voice
    0.3 * musiccoca.embed('deep rumbling bass') + # Seismic character
    0.2 * musiccoca.embed('IDM glitch') +         # Electronica genre
    0.1 * musiccoca.embed('atmospheric drone')    # Ambient bed
)
```

**Parameters to modulate by seismic data:**
| Seismic Parameter | Embedding Weight Modulation |
|-------------------|----------------------------|
| Magnitude | Increase bass/drone weight |
| Depth | Shift toward sub-frequencies |
| P-S interval | Tempo/rhythm embedding weight |
| Distance | Reverb/space text prompts |

---

### 3. Audio Injection (Live Steering)

**What it is:** Real-time audio input mixed with model output, tokenized, and fed as context for next generation. User audio influences but is never played back directly.

**This is transformative for SOS.**

**Current SOS flow:**
```
Seismic Data ──▶ Granular Synthesis ──▶ Audio Output
```

**With Audio Injection:**
```
┌─────────────────────────────────────────────────────────┐
│                    FEEDBACK LOOP                        │
│                                                         │
│  Seismic ──▶ Granular ──┬──▶ Mix ──▶ Encode ──▶ Context │
│  Waveform    Synthesis  │      │                  │     │
│                         │      │                  ▼     │
│                         │      │         ┌──────────┐   │
│                         │      ◀─────────│ Magenta  │   │
│                         │                │ RT LM    │   │
│                         │                └──────────┘   │
│                         │                      │        │
│                         ▼                      ▼        │
│                    Final Output ◀───── Decode          │
└─────────────────────────────────────────────────────────┘
```

**How it works (from paper):**
1. Mix user input audio (seismic granular output) with model output
2. Tokenize resulting mix via SpectroStream
3. Feed tokenized mix as context for next 2-second chunk
4. Model "chooses" to repeat, transform, or be influenced by input

**Implementation for SOS:**
```python
# Audio injection loop
injection_weight = 0.3  # How much seismic "bleeds through"

for chunk_i in infinite_stream():
    # Get current seismic granular output
    seismic_grains = granular_engine.generate(2.0)  # 2 seconds
    
    # Mix with previous model output
    mixed = (1 - injection_weight) * prev_output + injection_weight * seismic_grains
    
    # Encode to SpectroStream tokens (coarse 4 RVQ levels)
    context_tokens = spectrostream.encode(mixed, depth=4)
    
    # Generate next chunk conditioned on mixed context
    output_tokens = magenta_rt.generate(
        context=context_tokens,
        style=earthquake_style_embedding
    )
    
    # Decode and output
    prev_output = spectrostream.decode(output_tokens)
    yield prev_output
```

**Classifier-Free Guidance (CFG) for injection control:**
```python
# From paper: (1 + w) · Logits_pos − w · Logits_neg
# Higher w = more influence from seismic input

cfg_weight = magnitude_to_cfg(earthquake.magnitude)  # 0.0-5.0
logits = (1 + cfg_weight) * logits_with_seismic - cfg_weight * logits_without_seismic
```

---

### 4. SpectroStream Audio Codec

**What it is:** Neural audio codec converting 48kHz stereo to discrete tokens via Residual Vector Quantization (RVQ). 64 depth levels, 1024 codebook size, 25Hz frame rate.

**Integration with SOS:**

SpectroStream provides the **tokenization layer** that bridges continuous seismic waveforms with the language model.

**Bandwidth hierarchy:**
| Level | RVQ Depth | Bandwidth | Use |
|-------|-----------|-----------|-----|
| Coarse | 1-4 | 1kbps | Context (history) |
| Medium | 5-16 | 4kbps | Live generation |
| Fine | 17-64 | 16kbps | High-fidelity output |

**For SOS implementation:**
1. Encode seismic-derived audio at full depth (64 levels) for maximum fidelity
2. Use coarse encoding (4 levels) for context window efficiency
3. Generate at medium depth (16 levels) for real-time performance
4. Optional: Train custom SpectroStream on seismic/electronica dataset

**Context window math:**
```
10 seconds history × 25Hz × 4 RVQ = 1,000 context tokens
2 seconds output × 25Hz × 16 RVQ = 800 target tokens
```

---

### 5. Chunk-Based Autoregression

**What it is:** Generate audio in 2-second chunks, each conditioned on 10-second history (5 previous chunks) under Markov assumption.

**Advantages for seismic sonification:**
1. **Error isolation:** Each chunk is independent—seismic anomalies don't cascade
2. **Stateless inference:** No generation cache needed, simplifies browser deployment
3. **Dynamic conditioning:** Style can change every 2 seconds, perfect for evolving earthquakes

**Mapping earthquake events to chunks:**
```
Earthquake Timeline:
│ P-wave │ S-wave │ Surface │ Coda │
│ arrival│ arrival│ waves   │      │
▼        ▼        ▼         ▼

Chunk Conditioning:
│ Chunk 1 │ Chunk 2 │ Chunk 3 │ Chunk 4 │ ...
│ sparse  │ attack  │ dense   │ decay   │
│ ambient │ pulse   │ rhythm  │ drone   │
```

**Implementation:**
```python
def seismic_to_chunk_style(eq_event, chunk_index, chunks_since_arrival):
    """Map earthquake phase to style conditioning"""
    
    time_since_p = chunk_index * 2.0  # seconds
    
    if time_since_p < eq_event.p_s_interval:
        # P-wave phase: subtle, building tension
        return blend(['minimal', 'atmospheric', 'anticipation'], 
                     magnitude_weight=0.3)
    
    elif time_since_p < eq_event.p_s_interval + 10:
        # S-wave arrival: maximum energy
        return blend(['intense', 'percussion', 'bass drop'],
                     magnitude_weight=eq_event.magnitude / 10.0)
    
    elif time_since_p < eq_event.total_duration * 0.7:
        # Surface waves: rhythmic, complex
        return blend(['polyrhythm', 'complex', 'evolving'],
                     magnitude_weight=0.6)
    
    else:
        # Coda: decay, return to ambient
        return blend(['reverb', 'fading', 'ambient'],
                     magnitude_weight=0.2 * (1 - chunks_since_arrival/total_chunks))
```

---

### 6. Lyria RealTime API (Extended Controls)

**What it is:** Cloud-based API with advanced controls beyond open-weights model: tempo (BPM), stem balance, brightness, density, chromas, key.

**Available via:** `g.co/magenta/lyria-realtime`

**Advanced controls for SOS:**

| Lyria Control | Seismic Parameter Mapping |
|---------------|---------------------------|
| **BPM (tempo)** | P-wave dominant frequency × multiplier |
| **Brightness** | High-frequency seismic energy content |
| **Density** | Event rate (aftershocks/minute) |
| **Key** | Geographic region hash → musical key |
| **Stems (bass/drums/other)** | Magnitude → bass, depth → other |
| **Chromas** | Spectral content of seismic signal |

**Self-conditioning (from paper):** Lyria RT can predict its own control tokens, meaning it can autonomously evolve the music while being guided by seismic parameters as soft priors.

```python
# Control priors for self-conditioning
def seismic_to_control_priors(eq_data):
    return {
        'tempo_prior': normal(mean=eq_data.dominant_freq * 60, std=10),
        'density_prior': categorical([0.2, 0.5, 0.3]),  # low/med/high
        'brightness_prior': beta(eq_data.hf_energy, 1 - eq_data.hf_energy),
        'key_prior': one_hot(region_to_key(eq_data.location))
    }
```

---

### 7. Latent Constraints (Style Embedding Refinement)

**What it is:** GAN-based model that transforms text embeddings to match high-quality audio embedding distribution, improving generation fidelity.

**Problem it solves:** Text embeddings and audio embeddings don't perfectly overlap—~90% classifiable as which modality they came from.

**For SOS:** Since seismic waveforms are neither text nor typical music, they may produce embeddings in a "no-man's land" of the latent space. Latent constraints can:

1. Train discriminator on: `[text, low-quality audio, high-quality audio, seismic-derived]`
2. Generator transforms seismic embeddings toward high-quality audio distribution
3. Regularization keeps seismic "character" via cosine similarity loss

```python
# Latent constraint for seismic embeddings
class SeismicLatentConstraint(nn.Module):
    def __init__(self):
        self.generator = MLP(768, 768, layers=4)
        self.discriminator = MLP(768, 1, layers=4)
    
    def forward(self, seismic_embedding):
        # Transform toward musical space while preserving character
        constrained = self.generator(seismic_embedding)
        
        # Regularization: stay close to original
        reg_loss = 1 - cosine_similarity(seismic_embedding, constrained)
        
        return constrained, reg_loss
```

---

## Implementation Roadmap

### Phase 1: MusicCoCa Integration (Weeks 1-2)
- [ ] Set up Magenta RT Colab environment
- [ ] Build seismic waveform → audio prompt pipeline
- [ ] Test embedding arithmetic with text style prompts
- [ ] Validate seismic "character" preservation

### Phase 2: Audio Injection Loop (Weeks 3-4)
- [ ] Implement bidirectional audio stream (browser ↔ Colab)
- [ ] Create mixing strategy for granular output + model output
- [ ] Tune CFG weights for magnitude-responsive injection
- [ ] Test "looper mode" for rhythmic seismic patterns

### Phase 3: Lyria API Integration (Weeks 5-6)
- [ ] Apply for Lyria RealTime API access
- [ ] Map seismic parameters to advanced controls
- [ ] Implement self-conditioning with seismic priors
- [ ] Compare output quality vs. open-weights model

### Phase 4: Custom Training (Weeks 7-12)
- [ ] Curate seismic + electronica paired dataset
- [ ] Fine-tune MusicCoCa on seismic audio
- [ ] Train latent constraint model for seismic embeddings
- [ ] Potentially fine-tune SpectroStream on seismic frequencies

---

## Technical Requirements

### For Magenta RealTime (Open-Weights)
- **Hardware:** TPU v2-8 (free Colab) or NVIDIA GPU with 40GB VRAM
- **OS:** Linux
- **Docker:** Required for local deployment
- **Latency:** ~1.25s per 2s chunk (RTF 1.6)

### For Lyria RealTime API
- **Access:** API key from Google
- **Latency:** Cloud-dependent, typically <500ms
- **Controls:** Full advanced control suite

### For Browser Integration
- **WebSocket:** Real-time audio streaming
- **Web Audio API:** Local granular synthesis
- **AudioWorklet:** Low-latency processing

---

## Philosophical Alignment

The Live Music Models paper's core thesis resonates deeply with SOS:

> *"Live music represents a new frontier for generative AI... placing users in a continuous perception-action loop promotes more active creation, creates higher-bandwidth interaction, fosters personalized expression and emphasizes the process as much as the product."*

For SOS, Earth itself enters the perception-action loop. The earthquake becomes performer, the algorithm becomes instrument, and the listener becomes witness to planetary music-making.

The paper's distinction between "music as a noun" (static recordings) vs. "music as a verb" (live performance) maps perfectly to seismic sonification: earthquakes are inherently live events, unrepeatable, each with unique character. DeepMind's live music models provide the technical framework to honor that liveness.

---

## References

1. Live Music Models (arXiv:2508.04651) - Google DeepMind, 2025
2. Magenta RealTime GitHub: github.com/magenta/magenta-realtime
3. Lyria RealTime API: g.co/magenta/lyria-realtime
4. SpectroStream (arXiv:2508.05207) - Li et al., 2025
5. MusicLM (arXiv:2301.11325) - Agostinelli et al., 2023

---

*Analysis prepared for SOS (Sounds of Seismic) project integration planning*
*sos.allshookup.org*

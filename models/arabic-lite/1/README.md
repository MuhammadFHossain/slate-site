---
license: cc-by-4.0
language:
- ar
library_name: coreml
pipeline_tag: automatic-speech-recognition
base_model: lemuralabs/lemura-arabic-asr-lite
tags:
- coreml
- fastconformer
- ctc
- arabic
---

# Lemura Arabic ASR lite, Core ML (4-bit)

The Arabic speech model behind Arabic dictation in
[Slate](https://slatedictation.com), converted to Core ML to run on the
Apple Neural Engine. Needs macOS 15 or later.

It is [Lemura Labs' Arabic ASR lite](https://huggingface.co/lemuralabs/lemura-arabic-asr-lite),
itself fine-tuned from
[NVIDIA's Arabic FastConformer](https://huggingface.co/nvidia/stt_ar_fastconformer_hybrid_large_pcd_v1.0):
a 109M-parameter FastConformer encoder with a CTC head. Both are licensed
under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), and so is
this conversion.

## What changed

Converted by Blue Tomato Studio. Apart from the 4-bit weights, the model
computes the same function as the original, within rounding:

- Converted from NeMo to Core ML, in the two-model layout FluidAudio's CTC
  loader (`CtcModels.loadDirect`) reads: `MelSpectrogram` (16 kHz audio and
  its length in, normalised log-mel features out, float32) and
  `AudioEncoder` (features in, CTC logits and their length out, float16).
- The spectrogram's STFT rewritten as a strided convolution against a
  windowed DFT basis, matching NeMo's features to within 0.001.
- Values in the first layer exceed float16's range (attention scores near
  80,000, the residual stream near 97,000). The Neural Engine tolerates
  that; Core ML on the CPU and GPU returns NaN. Fixed exactly by moving
  powers of two: the 1/8 attention scale applied to the queries before the
  multiply, and each layer's residual stream divided down, which the
  LayerNorm in front of every sublayer ignores.
- Encoder weights palettized to 4 bits by k-means, one 16-value table per
  eight output channels, from the float16 weights. The CTC head, the
  per-layer position tables, biases and norms stay float16.

Fixed input of 240,000 samples (15 seconds at 16 kHz); pad shorter audio and
pass its real length. Greedy CTC: the blank is token 1024, after the last
entry in `vocab.json`.

## Accuracy

428 FLEURS Egyptian Arabic test clips (78 minutes), diacritics,
punctuation and letter variants normalised before scoring, against the
8-bit conversion of the same model:

| | Size | Word error | Character error | Per 10 s of audio (M4 Pro) |
|---|---|---|---|---|
| This model, Neural Engine | 65 MB | 9.0% | 3.9% | 0.013 s |
| This model, CPU only | | 9.1% | 3.9% | 0.03 s |
| 8-bit conversion, Neural Engine | 112 MB | 8.7% | 3.7% | 0.014 s |

## Files

`MelSpectrogram.mlmodelc`, `AudioEncoder.mlmodelc`, `vocab.json`; their
SHA-256 hashes are in `SHA256SUMS`. Slate checks every file against a hash
built into the app before using it.

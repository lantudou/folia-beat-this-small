# folia-beat-this-small

The Beat This! **small0** model exported for Folia's WebGPU runtime. Folia packages it with the
desktop app for Sonnet's 智能动感模式 (smart beat mode). The weights are not kept in Folia's git
history; Folia's build downloads the file pinned here (tag and sha256) when it packages the app,
the same way it gets FFmpeg.

The file is published as a release asset:

| Release | File | Bytes | SHA-256 |
|---|---|---|---|
| `small0-v1` | `beat_this-small-wgsl.bin` | 4755823 | `b143091f1c2060169d7f72276bc84684c8bd22b19d5c6a9a60cdf4eaf20079b4` |

## Where it comes from

- Model: [Beat This!](https://github.com/CPJKU/beat_this) (CPJKU, ISMIR 2024), checkpoint `small0`
  (2.1M parameters: transformer width 128, 6 layers, stem 32), downloaded from the authors' checkpoint
  host as `beat_this-small0.ckpt` (sha256 `6074be2c4d490c5f6101fcc374a1ec72ae93456e23bb6019783b849f5dc7d47b`).
- Exported with Folia's `build/beat_this_wgsl/export_weights.py CKPT OUT.bin --f16`:
  - BatchNorms and RMSNorm gammas are folded into the layers that use them.
  - Matmul weights are stored as fp16; everything else stays fp32.
  - The tensor manifest is packed into the same file, after the tensors and before an 8-byte trailer.

## How it was checked

On an Apple M4 Pro, through Electron's WebGPU (Dawn on Metal):

- **Against a plain-PyTorch forward of the checkpoint, every stage of one 1500-frame chunk:**
  - fp32 export: at least 107 dB at every stage; logits 110 dB.
  - This file (fp16): logits 76 dB.
- **Seven whole tracks through Folia's own mel spectrogram, chunking and peak picking:** the fp16 and
  fp32 exports give identical beat and downbeat lists.
- **Against final0 (the model Folia's automix uses) on the same seven tracks:**
  - Beat F-measure 0.92–1.00 and downbeat F-measure 0.78–1.00, both at ±70 ms.
  - Roughly 1.8 times faster.

## License

The model and its weights are MIT, © 2024 Institute of Computational Perception, JKU Linz (see
`LICENSE`). This file is those weights rearranged, with no retraining.

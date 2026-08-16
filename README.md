
---

# Deterministic Kinematic Extraction - Synthetic Video Dataset

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Paper](https://img.shields.io/badge/Paper-IEEE%20Access-blue)](https://ieeeaccess.ieee.org/) (under revision) 
[![Institution](https://img.shields.io/badge/Institution-UTFPR-green)](https://www.utfpr.edu.br/)

This repository contains the open-source synthetic video dataset generated for the paper (under revision):

> **"Deterministic Kinematic Extraction from Diffusion Models for Human Activity Recognition"**
> Marcos Eduardo Pivaro Monteiro, Jamil de Araujo Farhat, João Alberto Fabro, Bogdan Tomoyuki Nassu
> (under revision) *IEEE Access*, 2026 — Federal University of Technology – Paraná (UTFPR), Brazil

---

## Overview

This dataset was designed to overcome physical IMU data scarcity for Human Activity Recognition (HAR) on edge devices. It provides synthetically generated human kinematic video sequences produced by state-of-the-art text-to-video diffusion models in a zero-shot regime.

Unlike probabilistic distillation methods, this dataset is intended for **deterministic mathematical extraction**. The videos serve as the visual foundation for a rigid signal processing pipeline: 3D surface meshes are extracted via WHAM and SLAM, smoothed via optimal Savitzky-Golay filtering, and mathematically differentiated to derive physical 3-axis inertial (acceleration) tensors.

---

## Related Dataset

A sibling corpus by the same authors accompanies the companion study, which attacks the same
data-scarcity problem through **cross-modal knowledge distillation** rather than deterministic
extraction:

> **Synthetic IMU Data Generation via Diffusion-Based Video Models**
> https://github.com/marcoseduardopm/Synthetic-IMU-Data-Generation-Diffusion-Based-Video-Models

It shares this taxonomy, prompt structure and naming convention, and was generated with the
same pipeline under different seeds — so the two corpora are directly poolable, and the seed
in each filename identifies which study a clip originated from.

---

## Dataset & File Structure

The corpus is partitioned by the generative foundation model used, and split across two generation seeds:

```text
synthetic_dataset/            # primary corpus, seed 987654 — 320 clips
├── hunyuan_1_5_720/          # HunyuanVideo 1.5 generations   (80)
├── ltxv2_720/                # LTX-Video 2.0 generations      (80)
├── ltxv2_3_720/              # LTX-Video 2.3 generations      (80)
└── wan2_2_720/               # Wan 2.2 generations            (80)

synthetic_dataset_extra/      # supplementary corpus, seed 132456078 — 220 clips
├── hunyuan_1_5_720/          # 3 classes only                 (30)
├── ltxv2_720/                # all 8 classes                  (80)
├── ltxv2_3_720/              # all 8 classes                  (80)
└── wan2_2_720/               # 3 classes only                 (30)
```

### `synthetic_dataset/` — primary corpus (seed `987654`)

The full factorial sweep: **8 prompt classes × 10 environments × 4 models = 320 clips**.

### `synthetic_dataset_extra/` — supplementary corpus (seed `132456078`)

A second generation seed, added to widen the motion prior for the classes where monocular
mesh recovery is hardest. It is **deliberately not a full sweep**. The two LTX-Video models,
which are fast enough to sweep exhaustively, cover all eight classes; for Wan 2.2 and
HunyuanVideo 1.5, whose per-clip generation cost is far higher, generation was restricted to
the three classes whose kinematics are most often lost by the downstream tracker:

| Model | Classes generated | Clips |
| --- | --- | --- |
| `ltxv2_720` | all 8 | 80 |
| `ltxv2_3_720` | all 8 | 80 |
| `wan2_2_720` | `Class_B_Fall`, `Class_D_Stairs`, `Class_F_Stumble` | 30 |
| `hunyuan_1_5_720` | `Class_B_Fall`, `Class_D_Stairs`, `Class_F_Stumble` | 30 |

Falls end prone and partially self-occluded, stair navigation is frequently framed
side-on with the far leg hidden, and a stumble is brief and asymmetric — the three cases in
which a monocular tracker most often drops the subject. Both corpora are pooled before
filtering; the seed is recoverable from every filename, so either can be isolated.

### File Naming Convention

Inside each directory, the `.mp4` files follow a strict, programmatic naming convention based on the Kinematic Class, Activity Name, Environmental Modifier, and Generation Seed:

**Format:** `Class_[Letter]_[Activity]_[Environment]_seed[Seed].mp4`

**Example:** `Class_B_Fall_on_a_grassy_park_field_seed987654.mp4`

Every clip is therefore self-describing: class, environment and seed are all recoverable
from the filename alone, and the generating model from the parent directory.

---

## Kinematic Taxonomy

The dataset generation was governed by a unified **7-class dynamic taxonomy**. This taxonomy was specifically designed to align with the physical hardware targets of the PAMAP2 and SisFall datasets.

| Class ID | Prefix | Activity | Description |
| --- | --- | --- | --- |
| **0** | `Class_A` | **Running** | Continuous high-cadence bipedal locomotion |
| **1** | `Class_B` | **Fall** | Uncontrolled total-body collapse |
| **2** | `Class_C` | **Lifting** | Bending and vertical load manipulation |
| **3** | `Class_D` | **Stairs** | Ascending and descending stair navigation |
| **4** | `Class_E` | **Jumping** | Repetitive vertical impulse movements |
| **5** | `Class_F` | **Stumble** | Partial loss of balance with inertial recovery |
| **6** | `Class_G` | **Nordic Walking** | Flat-ground locomotion with synchronized arm engagement |
| *-* | `Class_H` | *Soccer* | *Included in the video generation corpus for spatial diversity, but strictly excluded from the final training/extraction pipeline.* |

`Class_H_Soccer` is released here because it was generated, but it takes no part in the
experiment: neither PAMAP2 nor SisFall provides a corresponding activity, so there is no
physical counterpart to augment. Eight prompt classes therefore yield seven experimental ones.

---

## Prompt Templates

Every clip comes from one of eight prompt templates, instantiated once per environment. The
literal `{environment}` placeholder is replaced by each modifier in the list below.

The templates are deliberately over-specified in three respects, because each one protects a
downstream stage rather than the visual result: **a single isolated person** (a second body
makes monocular mesh recovery ambiguous about which subject to track), **full body framed
from head to toe** (the pelvis–ankle distance is the scaling anchor of the extraction
pipeline, so a cropped subject cannot be scaled), and **a static or steady camera** (camera
motion is otherwise absorbed into the recovered world trajectory and differentiated into
acceleration that the subject never experienced).

| Class | Prompt template |
| --- | --- |
| `Class_A_Running` | *A single isolated person running at a steady pace `{environment}`. The camera is positioned to keep the person completely centered. Full body completely framed in the shot from head to toe, static camera. Clear daylight, dynamic and continuous periodic running motion.* |
| `Class_B_Fall` | *A single isolated person walking `{environment}`, slipping and falling onto the ground. The camera is positioned to keep the person completely centered. Full body completely framed in the shot, steady camera. Clear vertical drop, realistic physical impact.* |
| `Class_C_Lifting` | *A single isolated person standing `{environment}`, bending at the knees to lift a heavy box from the floor, and standing back up straight. The camera is positioned to keep the person completely centered. Full body completely framed in the shot, fluid vertical motion.* |
| `Class_D_Stairs` | *A single isolated person walking up a well-lit flight of stairs `{environment}`. The camera is positioned to keep the person completely centered. Full body completely framed, side-profile tracking shot. Rhythmic vertical displacement.* |
| `Class_E_Jumping` | *A single isolated person jumping rope `{environment}`. The camera is positioned to keep the person completely centered. Full body completely framed, static camera. Feet continuously leaving the ground, clear vertical landings.* |
| `Class_F_Stumble` | *A single isolated person walking `{environment}`, tripping over an unseen obstacle, and stumbling forward to regain balance. The camera is positioned to keep the person completely centered. Full body completely framed in the shot.* |
| `Class_G_Nordic_Walking` | *A single isolated person engaged in Nordic walking `{environment}`, using walking poles with exaggerated, rhythmic arm and leg strides. The camera is positioned to keep the person completely centered. Full body completely framed in the shot, steady tracking shot.* |
| `Class_H_Soccer` | *A single isolated person playing soccer `{environment}`, making clear directional changes and foot strikes. The camera is positioned to keep the person completely centered. Full body completely framed, dynamic tracking shot.* |

### Environmental Modifiers

To ensure the 3D mesh recovery algorithms do not overfit to specific backgrounds or lighting conditions, every activity prompt was multiplied across **10 distinct environmental modifiers**. The prompt uses the spaced form; the filename uses the underscored form.

| In the prompt | In the filename |
| --- | --- |
| `in an industrial hallway` | `in_an_industrial_hallway` |
| `on an outdoor concrete track` | `on_an_outdoor_concrete_track` |
| `in a sterile laboratory` | `in_a_sterile_laboratory` |
| `in a brightly lit gymnasium` | `in_a_brightly_lit_gymnasium` |
| `on a grassy park field` | `on_a_grassy_park_field` |
| `in a dense forest clearing` | `in_a_dense_forest_clearing` |
| `on a sandy beach` | `on_a_sandy_beach` |
| `in an empty parking lot` | `in_an_empty_parking_lot` |
| `on a city sidewalk` | `on_a_city_sidewalk` |
| `in a modern office lobby` | `in_a_modern_office_lobby` |

### Negative Prompt

One negative prompt is shared by every class, environment, model and seed:

```text
multiple people, crowd, background characters, close-up, cropped, zoomed in,
out of frame, head out of frame, feet out of frame, cut off, mutated, deformed,
floating, bad anatomy, missing limbs, extra limbs, motion blur, ghosting,
smudging, trailing pixels, blurry limbs, out of focus, poor definition,
low shutter speed
```

Its three groups mirror the positive constraints: suppress additional subjects, suppress
framing that truncates the kinematic chain, and suppress temporal artifacts. The last group
matters most for this pipeline — motion blur and trailing pixels are exactly the artifacts
that survive mesh recovery as positional jitter and are then amplified by double
differentiation.

---

## Technical Specifications

| Parameter | Value |
| --- | --- |
| **Resolution** | 704 × 1280 (portrait 720p) |
| **Aspect Ratio** | 9:16 (portrait) |
| **Frame Rate** | 24 fps |
| **Frames per clip** | 81 (3.375 s) |
| **Inference steps** | 50 |
| **Orientation** | Portrait (strictly enforced via prompt engineering to ensure full kinematic chain capture from head to toe) |
| **Generation Seeds** | `987654` (primary), `132456078` (supplementary) |

### Model Checkpoints and Per-Model Settings

All four generators share the frame count, frame rate, resolution, step count, seed and
negative prompt above. Only the guidance scale differs, tuned per model so that prompt
adherence does not collapse into the over-saturated, temporally rigid output that high
guidance produces in each architecture:

| Model | Hugging Face checkpoint | Guidance scale |
| --- | --- | --- |
| **Wan 2.2** | `linoyts/Wan2.2-T2V-A14B-Diffusers-BF16` | 5.0 |
| **HunyuanVideo 1.5** | `hunyuanvideo-community/HunyuanVideo-1.5-Diffusers-720p_t2v` | 6.0 |
| **LTX-Video 2.3** | `dg845/LTX-2.3-Diffusers` | 4.5 |
| **LTX-Video 2.0** | `Lightricks/LTX-2` | 3.5 |

Generation used the `diffusers` backend with balanced device mapping. A clip is fully
reproducible from its filename plus this table: the parent directory gives the checkpoint and
guidance scale, the filename gives the class template, the environment substitution and the
seed.

---

## How This Dataset Is Used (The Extraction Pipeline)

This dataset is not meant to be processed by a Vision Encoder. Instead, it is processed via the following deterministic extraction pipeline:

1. **Quality gating:** Clips pass an automated pixel-stability gate (VBench temporal flickering, motion smoothness and subject consistency) and a manual taxonomic curation pass that discards sequences depicting the wrong activity.
2. **3D Mesh Recovery:** Videos are processed by **WHAM** to extract a 6,890-point SMPL surface mesh for every frame.
3. **Global Translation:** Monocular camera drift and foot-sliding anomalies are resolved by coupling the mesh extraction with visual-inertial **SLAM**, anchoring the subject to a world-grounded coordinate system.
4. **Proportional Skeletal Scaling:** The arbitrary monocular scale is anchored to a fixed pelvis–ankle segment length, estimated as a high percentile over all frames of the clip.
5. **Digital Signal Processing (DSP):** Spatial coordinates are upsampled via cubic splines to the 100 Hz working rate of the physical corpora and smoothed using an optimized Savitzky-Golay filter ($w=7$, $p=3$), suppressing generative pixel-jitter while preserving high-frequency biomechanical impacts.
6. **Kinematic Differentiation:** The smoothed spatial trajectories are numerically differentiated twice over time, mapped into the wearable sensor frame with gravity applied on the vertical axis, and emitted as **3-axis acceleration tensors** in m/s².

---

## Physical Datasets Reference

This synthetic dataset was explicitly designed to act as a structural prior and augment the following open-source physical HAR datasets under data-scarce conditions:

* **PAMAP2** — Reiss, A. and Stricker, D. (2012). *Introducing a New Benchmarked Dataset for Activity Monitoring.* ISWC.
* **SisFall** — Sucerquia, A., López, J. D., and Vargas-Bonilla, J. F. (2017). *SisFall: A Fall and Movement Dataset.* Sensors, 17(1), 198.

---

## Citation

If you use this dataset or the extraction methodology in your research, please cite:
(under revision)

```bibtex
@article{monteiro2026deterministic,
  title   = {Deterministic Kinematic Extraction from Diffusion Models for Human Activity Recognition},
  author  = {Monteiro, Marcos Eduardo Pivaro and Farhat, Jamil de Araujo and Fabro, Jo\~ao Alberto and Nassu, Bogdan Tomoyuki},
  journal = {IEEE Access},
  year    = {2026},
  institution = {Federal University of Technology -- Paran\'a (UTFPR), Brazil}
}
```

---

## License

This dataset is released under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license. You are free to share and adapt the material for any purpose, provided appropriate credit is given.

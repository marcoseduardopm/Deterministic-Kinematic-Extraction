
---

# Synthetic Video Dataset for Cross-Modal IMU Knowledge Distillation

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Paper](https://img.shields.io/badge/Paper-IEEE%20Access-blue)](https://ieeeaccess.ieee.org/) (under revision) 
[![Institution](https://img.shields.io/badge/Institution-UTFPR-green)](https://www.utfpr.edu.br/)

This repository contains the open-source synthetic video dataset generated for the paper (under revision):

> **"Deterministic Kinematic Extraction from Diffusion Models for Human Activity Recognition"**
> Marcos Eduardo Pivaro Monteiro, Jamil de Araujo Farhat, Bogdan Tomoyuki Nassu, João Alberto Fabro
> (under revision) *IEEE Access*, 2026 — Federal University of Technology – Paraná (UTFPR), Brazil

---

## Overview

This dataset was designed to overcome physical IMU data scarcity for Human Activity Recognition (HAR) on edge devices. It provides synthetically generated human kinematic video sequences produced by state-of-the-art text-to-video diffusion models in a zero-shot regime.

Unlike probabilistic distillation methods, this dataset is intended for **deterministic mathematical extraction**. The videos serve as the visual foundation for a rigid signal processing pipeline: 3D surface meshes are extracted via WHAM and SLAM, smoothed via optimal Savitzky-Golay filtering, and mathematically differentiated to derive physical 3-axis inertial (acceleration) tensors. 

---

## Dataset & File Structure

The dataset was generated using a fixed random seed (`987654`) to ensure reproducibility. It is partitioned by the generative foundation model used:

```text
synthetic_dataset/
├── hunyuan_1_5_720/       # HunyuanVideo 1.5 generations
├── ltxv2_720/             # LTX-Video 2.0 generations
├── ltxv2_3_720/           # LTX-Video 2.3 generations
└── wan2_2_720/            # Wan 2.2 generations

```

### File Naming Convention

Inside each directory, the `.mp4` files follow a strict, programmatic naming convention based on the Kinematic Class, Activity Name, Environmental Modifier, and Generation Seed:

**Format:** `Class_[Letter]_[Activity]_[Environment]_seed[Seed].mp4`

**Example:** `Class_B_Fall_on_a_grassy_park_field_seed987654.mp4`

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

### Environmental Modifiers

To ensure the 3D mesh recovery algorithms do not overfit to specific backgrounds or lighting conditions, every activity prompt was multiplied across **10 distinct environmental modifiers**:

1. `in_an_empty_parking_lot`
2. `in_an_industrial_hallway`
3. `in_a_brightly_lit_gymnasium`
4. `in_a_dense_forest_clearing`
5. `in_a_modern_office_lobby`
6. `in_a_sterile_laboratory`
7. `on_an_outdoor_concrete_track`
8. `on_a_city_sidewalk`
9. `on_a_grassy_park_field`
10. `on_a_sandy_beach`

---

## Technical Specifications

| Parameter | Value |
| --- | --- |
| **Resolution** | 1280 × 704 (720p) |
| **Aspect Ratio** | 9:16 (portrait) |
| **Frame Rate** | 24 fps |
| **Orientation** | Portrait (strictly enforced via prompt engineering to ensure full kinematic chain capture from head to toe) |
| **Generation Seed** | `987654` |

---

## How This Dataset Is Used (The Extraction Pipeline)

This dataset is not meant to be processed by a Vision Encoder. Instead, it is processed via the following deterministic extraction pipeline:

1. **3D Mesh Recovery:** Videos are processed by **WHAM** to extract a 6,890-point SMPL surface mesh for every frame.
2. **Global Translation:** Monocular camera drift and foot-sliding anomalies are resolved by coupling the mesh extraction with visual-inertial **SLAM**, anchoring the subject to a world-grounded coordinate system.
3. **Digital Signal Processing (DSP):** Spatial coordinates are upsampled via cubic splines and smoothed using an optimized Savitzky-Golay filter ($w=7$, $p=3$) to suppress generative pixel-jitter while preserving high-frequency biomechanical impacts (e.g., $13g$ fall peaks).
4. **Kinematic Differentiation:** The smoothed spatial trajectories are numerically differentiated over time to yield physically bounded **3-axis acceleration tensors**.

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

```

```

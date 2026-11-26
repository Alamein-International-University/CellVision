# CellVision — Cell Nuclei Image Segmentation (Data Science Bowl 2018)

Classical digital image processing and computer vision pipeline for instance-level cell nuclei segmentation across diverse microscopy modalities (fluorescence and brightfield histology) on the **2018 Data Science Bowl (BBBC038v1)** dataset.

---

## Project Roadmap
- [x] **Phase 1:** Dataset integrity verification, modality/polarity EDA, 2D FFT spectral profiling, and Otsu thresholding baselines.
- [x] **Phase 2:** Channel/polarity normalization, Gaussian denoising, CLAHE adaptive contrast enhancement, and scale-aware distance-transform Watershed instance splitting.
- [ ] **Phase 3:** Modality-specific stain separation (H&E), error analysis, and parameter ablation study.
- [ ] **Phase 4:** Test set inference (`stage1_test`) and RLE submission generation.

---

## Pipeline Overview

### Phase 1: Exploratory Data Analysis & Baseline Benchmarking
- **Dataset Validation:** Verified all `670` training samples (`stage1_train`) with `0` missing masks and `0` overlapping ground-truth instance labels.
- **Modality & Scale Profiling:** Identified two primary imaging regimes—dark-background fluorescence (`n=548`) and bright-background histology (`n=122`)—spanning resolutions from `256×256` to `1040×1388` pixels.
- **Frequency Domain Analysis:** Applied zero-mean, 2D Hann-windowed Fast Fourier Transforms (FFT) with normalized radial power spectra, confirming macro-level nuclear structure below `0.05 cycles/pixel` and high-frequency sensor noise above `0.25 cycles/pixel`.
- **Competition Metric:** Implemented the exact DSB 2018 Instance Mean Average Precision (`mAP` across IoU thresholds `0.50:0.95`) alongside semantic foreground Dice.

### Phase 2: Preprocessing & Scale-Aware Watershed Segmentation
1. **Channel & Polarity Normalization:** Extracts the green channel for RGB brightfield stains (maximizing hematoxylin nuclear contrast) and inverts bright-background images so all samples share a unified dark-background convention.
2. **Noise Suppression & Adaptive Contrast:** Applies Gaussian smoothing (`sigma=1.2`) to attenuate high-frequency sensor grain, followed by low-clip CLAHE (`clip_limit=0.008`) to recover faint nuclei without amplifying background noise.
3. **Morphological Cleanup:** Uses Otsu thresholding, hole filling (`binary_fill_holes`), resolution-scaled debris removal (`min_size = max(15, 0.02% of image area)`), and morphological opening.
4. **Scale-Aware Distance-Transform Watershed:** Computes the Euclidean Distance Transform and dynamically scales `peak_local_max` seed spacing (`adaptive_min_dist = clip(0.65 × median_equivalent_radius, 6, 22)`) to each image's median object radius, splitting clumped nuclei without shattering large single nuclei.

---

## Benchmark Results (`stage1_train`, n = 670)

| Pipeline Stage | Overall mAP (`0.50:0.95`) | Dark-BG mAP (`n=548`) | Bright-BG mAP (`n=122`) | Semantic Dice |
| :--- | :---: | :---: | :---: | :---: |
| **Phase 1 — Polarity-Aware Otsu Baseline** | `0.4027` | `0.4578` | `0.1549` | `0.8185` |
| **Phase 2 — CLAHE + Scale-Aware Watershed** | **`0.4335`** (`+0.0309`) | **`0.4849`** (`+0.0271`) | **`0.2026`** (`+0.0477`) | **`0.8617`** |

### Key Takeaways & Next Steps
- **Scale invariance matters:** Replacing a fixed `7px` peak distance (`0.3391` mAP) with an adaptive radius-scaled peak distance (`0.4335` mAP) eliminated over-segmentation on high-resolution images.
- **Bright-background opportunity:** While semantic Dice reached `0.8617` and bright-background mAP improved by `+30.8%` relative (`0.1549` $\rightarrow$ `0.2026`), histology stains remain the primary bottleneck to address via optical-density / H&E color deconvolution in **Phase 3**.

---

## Repository Structure
```text
CellVision/
├── .gitignore
├── LICENSE
├── README.md
├── requirements.txt
└── notebooks/
    └── nuclei_segmentation_pipeline.ipynb
```

---

## Setup & Usage
1. **Clone the repository and install dependencies:**
   ```bash
   git clone https://github.com/GeorgeA0/CellVision.git
   cd CellVision
   pip install -r requirements.txt
   ```
2. **Download the dataset:**
   Download `stage1_train` from the [2018 Data Science Bowl (BBBC038v1)](https://bbbc.broadinstitute.org/BBBC038) and place it in the project root directory.
3. **Run the pipeline:**
   Open and execute `notebooks/nuclei_segmentation_pipeline.ipynb` in Jupyter or Google Colab.

---

## Dataset Citation
> Caicedo, J.C., Goodman, A., Karhohs, K.W. et al. *Nucleus segmentation across imaging experiments: the 2018 Data Science Bowl.* Nature Methods 16, 1247–1253 (2019). Released under the **CC0 1.0 Universal (Public Domain)** license.

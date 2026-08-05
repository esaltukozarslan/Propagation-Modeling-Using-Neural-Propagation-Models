# Learning-Based RF Propagation Modeling in Urban Environments Using Neural Propagation Models

Master's thesis, Electrical and Computer Engineering, University of California, Santa Barbara (March 2026)

**Author:** Emir Saltuk Ozarslan
**Committee:** Prof. Upamanyu Madhow (Chair), Prof. Mahnoosh Alizadeh, Prof. Kenneth Rose

[Thesis PDF](#) &nbsp;·&nbsp; [Abstract](#abstract) &nbsp;·&nbsp; [Methods](#methods) &nbsp;·&nbsp; [Results](#results) &nbsp;·&nbsp; [Repo structure](#repository-structure)

---

## Abstract

Modeling how radio signals propagate through cities is a prerequisite for nearly everything in modern wireless network design — coverage planning, beamforming, and the deployment of next-generation systems all depend on it. Physics-based ray tracing can do this accurately, but at significant computational cost, particularly in large outdoor environments where every reflection, diffraction, and scattering interaction has to be resolved across detailed 3D geometry. Learning-based methods offer a way to sidestep this bottleneck: train on ray-tracing data once, then predict propagation behavior at a fraction of the cost.

This thesis examines two such methods — **Geo2SigMap** and **RF-3DGS** — within a common outdoor simulation environment built on **BostonTwin**, a city-scale digital twin of Boston, and the **Sionna** ray-tracing framework. Ray-tracing simulations on this environment serve as ground truth for both training and evaluation. The two methods represent distinct modeling philosophies: Geo2SigMap uses a cascaded U-Net architecture to predict dense path-gain maps from building geometry and transmitter location, while RF-3DGS takes a scene-reconstruction approach, learning a 3D Gaussian representation of the radio environment and rendering spatial RF spectra directly from it.

Both methods are evaluated against ray-tracing ground truth using metrics appropriate to each — RMSE and MAE for Geo2SigMap's map predictions, and RMSE, PSNR, SSIM, and LPIPS for RF-3DGS's spectrum reconstructions. Geo2SigMap proves effective and practical for large-scale coverage map prediction, particularly when geometric alignment between inputs and targets is maintained. RF-3DGS captures richer channel structure — including angular and delay information — but is more sensitive to the quality of its input spectra.

## Motivation

Predicting how radio signals behave in urban environments is one of the more practically difficult problems in wireless engineering. Buildings, roads, and other structures scatter, block, and reflect signals in ways that are hard to capture with simple formulas, and errors compound quickly when planning real deployments. This matters not just for basic coverage, but for systems that depend on fine spatial channel properties — beamforming, massive MIMO, and directional links all require more than a coarse path-loss estimate.

The traditional answer has been to either run expensive ray-tracing simulations for every scenario or rely on empirical models that generalize poorly to specific environments. This thesis studies the middle ground: digital twins and learning-based approaches that absorb the cost of high-fidelity simulation once and then serve efficient predictions afterward — evaluating two such approaches side by side, in the same high-fidelity urban environment, so that differences in performance can be attributed to the methods themselves rather than to differing setups.

## Contributions

- A unified **BostonTwin–Sionna** outdoor RF simulation pipeline, providing a high-fidelity digital-twin environment for learning-based wireless modeling.
- **Geo2SigMap** adapted to this environment for dense path-gain-map prediction.
- **RF-3DGS** adapted to a localized BostonTwin outdoor scene using multiple RF spectrum representations, including CBF, MVDR, ideal/MPC-based, AoD, and delay targets.
- A comparative study examining the strengths, limitations, and practical tradeoffs of map-based vs. scene-based RF learning under the same digital-twin setting.

## Methods

### Geo2SigMap — coverage map prediction
A cascaded, two-stage U-Net architecture:
- **Stage 1** predicts a coarse path-gain map from a building-height raster and a Gaussian transmitter map.
- **Stage 2** refines that estimate using the same geometric inputs together with the coarse prediction, a sparse-measurement map, and a sparse mask, producing the final path-gain map.

Evaluated with **RMSE** and **MAE** against Sionna ray-tracing ground truth, including sensitivity to the sparse-measurement budget available at inference time.

### RF-3DGS — scene-level RF reconstruction
A scene-reconstruction approach that learns a 3D Gaussian representation of the radio environment (a Radio Radiance Field) with CSI-encoded spherical harmonics, and renders spatial RF spectra directly from that representation. Adapted here to a localized, transmitter-centered outdoor scene cropped from BostonTwin, with point-cloud initialization for the Gaussian splats.

Trained and evaluated across several spectrum representations:
- Conventional Beamforming (CBF)
- Minimum Variance Distortionless Response (MVDR)
- Ideal / multipath-component (MPC)-based spectrum
- Angle-of-Departure (AoD) spectrum
- Delay spectrum

Evaluated with **RMSE, PSNR, SSIM, and LPIPS** between rendered and target spectra.

## Dataset & Simulation Pipeline

Both methods share a common data-generation pipeline built on:
- **[BostonTwin](https://github.com/wineslab/boston_twin)** — a city-scale, geometrically detailed digital twin of Boston with geolocated wireless infrastructure.
- **[Sionna RT](https://github.com/NVlabs/sionna)** — an open-source differentiable ray-tracing framework used to generate ground-truth propagation data (path gains, multipath components, angles, delays).

From this shared scene environment, two dataset variants are produced:
- **Geo2SigMap data**: tiled building-height rasters paired with Sionna-simulated reference path-gain maps on a 128×128 grid, with per-tile transmitter placement and valid-region masking.
- **RF-3DGS data**: a localized, transmitter-centered 250m × 250m micro-scene with LoS/NLoS-classified receiver locations, per-receiver ray-traced spectra (CBF/MVDR/MPC/AoD/delay), and a geometry-aware point cloud used to initialize the Gaussian splats.

## Results

**Geo2SigMap** performs consistently and responds predictably to additional sparse measurements when the geometry is well prepared and path-gain targets are aligned with the input building maps. (Note: the reported numbers rest on a single held-out test tile, so they are best read as a proof of concept for the BostonTwin adaptation rather than a statistically robust benchmark.)

**RF-3DGS** is harder to get right — sensitive to scene design, spectrum construction, and coordinate consistency — but when those are in order, it reconstructs channel structure that a 2D map cannot represent at all: angular distributions, delay spread, and transmitter-side directional information. The choice of supervision signal mattered more than almost anything else: switching from CBF to MVDR inputs changed the qualitative character of the reconstruction, not just the error numbers.

Full quantitative tables and qualitative figures are in the [thesis PDF](#).

## Key Takeaways

- **Geo2SigMap** is the practical choice for large-scale coverage-map prediction when speed and simplicity matter and 2D path gain is the target quantity.
- **RF-3DGS** is the right tool when richer channel structure (angle, delay) is needed, but its output quality is bottlenecked by the quality of the RF observations it's trained on — investing in better observation/spectrum design may matter as much as model architecture.
- **BostonTwin** works well as a shared evaluation foundation: its geometric detail and simulation-ready structure make it possible to study very different learning-based methods under realistic, consistent conditions.

## Future Work

- Combine map-based and scene-based descriptors (including AoD/delay) into a unified framework that provides both large-scale coverage estimation and detailed spatial CSI.
- Improve RF-3DGS spectrum quality via better array-processing strategies, denser path sampling for MPC-based spectra, or hybrid CBF/path-based supervision.
- Close the sim-to-real gap with hybrid pipelines that calibrate the digital twin using a small number of real field measurements.

## Repository Structure

> _Update this section once the code is added._

```
.
├── data_pipeline/          # BostonTwin + Sionna simulation & dataset generation
├── geo2sigmap/              # Geo2SigMap model, training, and evaluation code
├── rf_3dgs/                  # RF-3DGS model, training, and evaluation code
├── notebooks/                # Analysis / figure-generation notebooks
├── figures/                   # Generated figures and result plots
└── README.md
```

## Requirements

> _Add exact package versions / environment.yml or requirements.txt once available._

- Python 3.x
- [Sionna](https://github.com/NVlabs/sionna)
- [BostonTwin](https://github.com/wineslab/boston_twin)
- PyTorch
- 3D Gaussian Splatting dependencies (for RF-3DGS)

## Usage

> _To be filled in with setup and run instructions once the code is added (dataset generation, training, evaluation, and figure reproduction commands)._

## Citation

If you use this work, please cite:

```bibtex
@mastersthesis{ozarslan2026learning,
  title  = {Learning-Based RF Propagation Modeling in Urban Environments Using Neural Propagation Models},
  author = {Ozarslan, Emir Saltuk},
  school = {University of California, Santa Barbara},
  year   = {2026},
  type   = {Master's Thesis}
}
```

## References

This work builds directly on:
- Geo2SigMap: Li et al., *"GEO2SIGMAP: High-fidelity RF signal mapping using geographic databases,"* IEEE DySPAN, 2024.
- RF-3DGS: Zhang et al., *"RF-3DGS: Wireless channel modeling with radio radiance field and 3D Gaussian splatting,"* IEEE Trans. Wireless Commun., 2026.
- BostonTwin: Testolina et al., *"BostonTwin: The Boston digital twin for ray-tracing in 6G networks,"* ACM MMSys, 2024.
- Sionna: Hoydis et al., *"Sionna RT: Differentiable ray tracing for radio propagation modeling,"* arXiv:2303.11103, 2023.

(Full reference list in the thesis.)

## Acknowledgements

Thanks to my committee — Prof. Upamanyu Madhow (Chair), Prof. Mahnoosh Alizadeh, and Prof. Kenneth Rose — for their guidance, and to Burak Kekec for his invaluable help throughout this work.

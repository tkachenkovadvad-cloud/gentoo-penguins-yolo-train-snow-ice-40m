# gentoo penguins YOLO training on snow–ice surface (40 m)

This repository presents a **reproducible machine-learning pipeline** for automated detection and counting of **gentoo penguins** in UAV imagery using **YOLOv8** and **tile-based inference**.

The project focuses on detection of **very small objects** (≈15–22 pixels in height) in **high-density colonies** on **snow–ice dominated surfaces**, acquired from low-altitude drone surveys.

---

## Project scope

- Species: **Gentoo penguins (Pygoscelis papua)**
- Platform: UAV (DJI Air 2S, DJI Mavic 2)
- Nominal flight altitude: **~40 m AGL**
- Surface type: snow–ice dominated
- Number of annotated individuals: **1,048**
- Number of UAV flights: **28**
- Annotation tool: Label Studio
- Detection model: YOLOv8 (nano)
- Inference strategy: tile-based inference with global NMS

---

## Dataset overview

The training dataset consists of **1,048 manually annotated gentoo penguins**, extracted from UAV imagery acquired during **28 Ukrainian Antarctic Expeditions (UAE)**.

The imagery was collected using **DJI Air 2S** and **DJI Mavic 2** platforms at a nominal altitude of approximately **40 m above ground level**, resulting in small object sizes and strong spatial correlation between images within individual flights.

> ⚠️ **Data availability**  
> The annotated dataset is **not publicly released**.  
> Access may be granted **upon reasonable request to the author** for research purposes.

---

## Method summary

The implemented pipeline includes:

1. Manual annotation of a limited number of UAV images
2. Conversion to YOLO format
3. Automatic tiling with overlap
4. YOLOv8 model training on tiled data
5. Tile-based inference on full-resolution imagery
6. Merging detections into global coordinates
7. Global Non-Maximum Suppression
8. Final object counting

A detailed description of all engineering decisions, environment separation, Label Studio integration, and active learning strategy is provided in:

## Project Status and Outlook

This repository represents the **initial stage** of a long-term effort to
develop a robust UAV-based detection pipeline for gentoo penguins on
snow–ice dominated surfaces.

The current annotated dataset (1,048 individuals) is intended as a
**methodological proof-of-concept**. Ongoing and planned annotation efforts
are expected to expand the snow–ice training dataset to approximately
**10,000 annotated individuals**, enabling further refinement and validation
of the detection models.

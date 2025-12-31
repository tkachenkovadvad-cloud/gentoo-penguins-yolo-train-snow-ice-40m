## Materials and Methods

### Study Context and Data Acquisition

This study presents a methodological framework for automated detection and
counting of gentoo penguins (*Pygoscelis papua*) in UAV imagery acquired over
snow–ice dominated Antarctic environments.

UAV surveys were conducted using **DJI Air 2S** and **DJI Mavic 2** platforms
at a nominal flight altitude of approximately **40 m above ground level (AGL)**.
The imagery was collected during **28 Ukrainian Antarctic Expeditions (UAE)**
within the institutional framework of the **National Antarctic Scientific Center of Ukraine (NASC)**.

All field activities were carried out under an official permit issued by the
**Ministry of Education and Science of Ukraine** (Permit Series **AP No. 093-23**),
valid from **20 February 2023 to 30 April 2028**, authorizing research activities
in the Antarctic Treaty area south of 60°S.

Surveys were non-invasive and designed to minimize disturbance to wildlife.

---

### Dataset and Annotation

The dataset used in this study consists of **133 UAV images**, acquired during
multiple spatially correlated flights. From this collection, **44 images**
were manually annotated, yielding **1,048 individual gentoo penguin annotations**.

Annotations were produced using **Label Studio**, with a single object class
(`Penguin`) and rectangular bounding boxes. Each individual penguin was
annotated separately using tight bounding boxes, excluding shadows, guano,
rocks, and background features. Closely spaced individuals were annotated
as distinct objects.

Due to the nature of UAV surveys, images within the same flight were considered
spatially correlated rather than statistically independent.

---

### Dataset Preparation and Tiling

Annotated images were exported to YOLO format and used to generate a tile-based
training dataset. Images were automatically divided into **1024 × 1024 pixel**
tiles with **20% overlap** to increase the effective object scale for small
individuals.

The resulting dataset included:
- 22 positive training tiles
- 4 positive validation tiles
- 416 background-only (negative) tiles extracted from unannotated imagery

Negative tiles were included to reduce false-positive detections on visually
homogeneous snow–ice backgrounds.

---

### Model Architecture and Training

Object detection was performed using the **YOLOv8 nano (YOLOv8n)** architecture.
The model was trained from scratch on the tiled dataset using an input image
size of **1024 pixels**.

Training was conducted for **40 epochs**, with conservative data augmentation
settings. Aggressive augmentations (e.g. mosaic and mixup) were disabled to
avoid distortion of small objects, while mild geometric transformations were
retained.

Model performance was evaluated at the tile level, yielding:
- mAP@50 ≈ **0.92**
- mAP@50–95 ≈ **0.69**

---

### Tile-Based Inference

Direct full-frame inference on original UAV images produced unstable detection
results due to the small size of target objects. Therefore, **tile-based
inference** was employed for all downstream analyses.

Inference was performed on **138 image tiles** using a low confidence threshold
(confidence = **0.001**) and an IoU threshold of **0.7** to maximize recall.
The initial tile-level inference produced **3,965 detections**, representing
an upper bound prior to duplicate removal.

---

### Post-processing and Global Non-Maximum Suppression

Tile-level detections were transformed into global image coordinates and merged
across overlapping tiles. **Global Non-Maximum Suppression (NMS)** with an IoU
threshold of **0.5** was applied to remove duplicate detections caused by tile
overlap.

Following post-processing, the final count was **3,534 detections**, corresponding
to an approximate **11% reduction** relative to the uncorrected total.

---

### Data Availability

The annotated dataset is **not publicly released** due to institutional,
logistical, and environmental constraints associated with Antarctic fieldwork.
Access to the annotated data may be granted **upon reasonable request to the
author** for non-commercial research purposes and subject to permitting
conditions.

The complete software pipeline is publicly available under the **Apache
License 2.0**.

---

### Scope and Limitations

This work is presented as a methodological contribution rather than a final
population estimate. Key limitations include spatial correlation among images,
restricted environmental diversity, and sensitivity to flight altitude and
imaging geometry.

The proposed pipeline is transferable in principle but requires retraining or
fine-tuning for different species, altitudes, or surface conditions.

The present dataset size reflects the early stage of model development.
The primary objective of this work is to establish and validate the detection
pipeline. Expansion of the snow–ice dataset from approximately 1,000 to
10,000 annotated individuals is planned as part of ongoing work and is
expected to further strengthen model performance.


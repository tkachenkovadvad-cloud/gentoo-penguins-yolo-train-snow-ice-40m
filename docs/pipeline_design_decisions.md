# Pipeline Design Decisions

This document provides a detailed description of the engineering decisions,
design constraints, and methodological choices underlying the UAV-based
penguin detection pipeline implemented in this repository.

The purpose of this document is to ensure **transparency, reproducibility,
and interpretability** of the proposed method, rather than to serve as a
step-by-step installation guide.

---

## 1. Problem Definition

The target task is automated detection of **gentoo penguins (Pygoscelis papua)**
in UAV imagery acquired over snow–ice dominated surfaces.

Key characteristics of the problem:

- Individual penguins occupy approximately **15–22 pixels in height**
  in the original imagery.
- A single UAV frame may contain **tens to hundreds of individuals**.
- Images are **highly spatially correlated** within individual flights.
- Background is visually homogeneous (snow, ice, shadows, guano),
  increasing the risk of false positives.
- Standard full-frame inference with common object detectors
  yields unstable recall for such small objects.

These constraints motivated a pipeline explicitly optimized for
**small-object detection under high-density conditions**.

---

## 2. Data Acquisition Context

- Platforms: **DJI Air 2S**, **DJI Mavic 2**
- Nominal flight altitude: **~40 m AGL**
- Surface type: snow–ice dominated
- Campaign scale: **28 UAV flights**
- Institutional framework: **Ukrainian Antarctic Scientific Center (UASC)**

The imagery was acquired during multiple UAV flights, resulting in
distinct but internally correlated scenes. This correlation was explicitly
considered during dataset preparation and model evaluation.

---

## 3. Separation of Software Environments

A strict separation of Python environments was adopted to avoid dependency
conflicts between annotation tools and machine-learning frameworks.

### 3.1 Annotation Environment

The annotation environment is dedicated exclusively to **Label Studio** and
its dependencies. This environment prioritizes long-term stability over access
to the latest numerical libraries.

Design goals:
- Stable operation of Label Studio
- Compatibility with local file storage
- No heavy machine-learning dependencies

### 3.2 Machine Learning Environment

A separate environment is used for:
- YOLOv8 training and inference
- GPU-enabled PyTorch
- Optional tiling-based inference libraries

This separation prevents version conflicts (e.g. NumPy, Pillow, FastAPI)
and ensures reproducibility of both annotation and training workflows.

---

## 4. Annotation Strategy

### 4.1 Annotation Tooling

Annotations were produced using **Label Studio** with:
- a single object class: `Penguin`
- rectangular bounding boxes only
- local file storage to avoid cloud dependencies

### 4.2 Annotation Rules

Strict annotation rules were applied to ensure label quality:

- Each penguin is annotated with a **tight bounding box**
  (≤ 1 pixel margin where possible).
- Shadows, guano, rocks, and background structures are excluded.
- Closely spaced individuals are annotated **as separate objects**.
- No group or polygon annotations are used.

These rules prioritize **precision and consistency** over annotation speed.

---

## 5. Bootstrap Dataset

The initial training dataset (“bootstrap set”) consisted of:

- **1,048 manually annotated penguins**
- Images selected from multiple flights to maximize scene diversity

Despite its small size, this dataset was sufficient to initialize
a functional detector and enable an active learning workflow.

---

## 6. Model Choice and Training Configuration

### 6.1 Model Architecture

The **YOLOv8 nano (YOLOv8n)** architecture was selected due to:

- small model size
- fast iteration cycles
- sufficient capacity for single-class detection
- compatibility with limited GPU memory

### 6.2 Training Design Choices

Key training decisions included:

- Training from scratch or from lightweight pretrained weights
- Large input image sizes to improve small-object visibility
- Disabling aggressive augmentations (e.g. mosaic, mixup)
- Conservative geometric augmentations only

These choices were driven by the risk that aggressive augmentations
could distort already small objects beyond recognition.

---

## 7. Active Learning Workflow

An **iterative active learning loop** was a core design objective.

The cycle consists of:

1. Manual annotation of a small batch of new images
2. Export to YOLO format
3. Model training or fine-tuning
4. Deployment of the updated model for pre-labeling
5. Manual correction of model-generated labels
6. Repetition of the cycle

This approach significantly reduces manual annotation effort while
progressively improving model performance and robustness.

---

## 8. Tiling-Based Inference Rationale

Full-frame inference proved insufficiently stable for detecting
18–22 pixel objects.

To address this, the pipeline adopts **tile-based inference**:

- Images are split into overlapping tiles
- Each tile is processed independently by the detector
- Detections are mapped back to global image coordinates
- Duplicate detections are resolved using global NMS

This strategy effectively increases the relative size of objects
within each inference window, substantially improving recall.

---

## 9. Post-processing and Counting

Post-processing steps include:

- merging tile-level detections into global coordinates
- applying **global Non-Maximum Suppression**
- counting final detections after duplicate removal

This ensures that overlapping tiles do not inflate population estimates
and that the final count reflects unique detected individuals.

---

## 10. Scope and Limitations

This pipeline is presented as a **methodological contribution**, not as a
final population census tool.

Key limitations include:

- spatial correlation between images within flights
- limited geographic and environmental diversity
- sensitivity to flight altitude and imaging geometry

The method is transferable in principle but requires re-training
or fine-tuning for different altitudes, species, or surface conditions.

---

## 11. Current Maturity

At the current stage, the pipeline:

- is fully functional end-to-end
- is reproducible and documented
- supports active learning workflows
- is suitable for publication as a methods paper or technical reference

Future work primarily concerns dataset expansion rather than
fundamental architectural changes.

---

## 12. Design Philosophy

The pipeline emphasizes:

- transparency over black-box optimization
- reproducibility over maximal benchmark scores
- practical field constraints over idealized datasets

All major design decisions were guided by real-world limitations
of UAV-based ecological monitoring in polar environments.


## Future Work

The current implementation represents an initial methodological baseline.
While the present dataset includes 1,048 annotated individuals, ongoing and
planned annotation efforts are focused on expanding the snow–ice training
dataset to approximately **10,000 annotated penguins**.

This expansion is expected to improve model robustness, generalization
across colony densities, and stability of tile-based inference, without
fundamental changes to the pipeline architecture.

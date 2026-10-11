# AI Application - Group-9

## Install

```
pip install -r requirements.txt
```
---

## Project structure

```
primagaz-yolo/
├── notebooks/
│   ├── 01_gas_bottle_baseline.ipynb
│   ├── 02_video_tracking.ipynb
│   ├── 03_gas_cylinder_training.ipynb
│   ├── 04_gas_cylinder_evaluation.ipynb
│   └── 05_cvat_custom_dataset.ipynb
├── data/
├── videos/
├── results/
└── requirements.txt
```

---

## Notebooks

| Notebook | Purpose |
|---|---|
| 01_gas_bottle_baseline | Runs pretrained YOLO11n (COCO) on local LGP images. Shows the model has no cylinder class. |
| 02_video_tracking | Runs ByteTrack and BoT-SORT on short videos. Also re-runs tracking with the fine-tuned models. |
| 03_gas_cylinder_training | Fine-tunes YOLO11n on every available gas-cylinder dataset with the same settings. |
| 04_gas_cylinder_evaluation | Re-evaluates each `best.pt` on validation and test. Saves metrics and annotated examples. |
| 05_cvat_custom_dataset | End-to-end pipeline for the manually labelled CVAT dataset. Can be re-run as more labels are added. |

### To run the notebooks, the datasets and videos files will have to be downloaded through the given links in relevant folders.
---

## Datasets

| Dataset | Location | Classes | Link |
|---|---|---|---|
| Local LGP images | `data/all_images/` | none | local |
| CylinDeRS | `data/gas_bottles/` | `gas_cylinder` | https://universe.roboflow.com/klearchos-stavrothanasopoulos-konstantinos-gkountakos-6jwgj/cylinders-iaq6n/dataset/3 |
| Roboflow gas-cylinder | `data/gas_cylinder/` | `Cap_Close`, `Cap_open` | https://universe.roboflow.com/dl-zepmn/gas-cylinder-fn6bf/dataset/1 |
| Roboflow gas (hack) | `data/gas_hack/` | `gas` | https://universe.roboflow.com/hack-rhpxr/gas-bsckk/dataset/1 |
| CVAT batch 1 | `data/cvat_batch_1/` | `gas-bottles` | locally labelled |

**Notes:**
- CylinDeRS is the best match for whole-cylinder detection.
- The Roboflow gas-cylinder dataset labels cap states, not whole cylinders.
- `gas_hack` produced inconsistent metrics and is excluded from results.

**Video links**
- https://youtu.be/PtKnRpGvPM4
- https://youtu.be/UH-u0g3B8Pw

---

## Model

YOLO11n, initialized from COCO weights for every fine-tuning run.
Chosen for laptop-friendliness and to match the assignment hint about small models.

---

## Training settings

| Field | Value |
|---|---|
| Model | YOLO11n |
| Epochs | 5 (public datasets), 20 (CVAT) |
| Image size | 416 |
| Batch size | 4 |
| Training fraction | 0.25 (public), 1.00 (CVAT) |
| Device | CPU (GPU if available) |

---

## Results

**Baseline:** The pretrained COCO model has no cylinder class. It labels cylinder-shaped objects as vase, bottle, or fire hydrant, or misses them.

**Tracking:** ByteTrack is faster than BoT-SORT. Tracked classes are COCO classes in the pretrained section and cylinder-related classes in the fine-tuned section.

**Fine-tuning and evaluation:**

| Dataset | Classes | Split | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|---|---|
| gas_bottles | gas_cylinder | val | 0.9170 | 0.6864 | 0.8866 | 0.8421 |
| gas_bottles | gas_cylinder | test | 0.9321 | 0.6845 | 0.8952 | 0.8545 |
| gas_cylinder | Cap_Close, Cap_open | val | 0.7896 | 0.3108 | 0.3449 | 0.1263 |
| gas_cylinder | Cap_Close, Cap_open | test | 0.8344 | 0.3421 | 0.3775 | 0.1567 |

`gas_hack` is excluded. Its precision, recall, and mAP50 are internally inconsistent, indicating a label or YAML mismatch that was not resolved.

**CVAT:** The custom dataset pipeline works end-to-end. Metrics are high but inflated by scene overlap between train and val. The dataset is still being labelled; the exact size and metrics change each time Notebook 05 is re-run.

---

## Known limitations

- The COCO model has no cylinder class.
- The cap-state dataset labels cap regions, not whole cylinders.
- `gas_hack` metrics are inconsistent and excluded.
- The CVAT dataset has train/val scene overlap and is still growing.
- Most training was done on CPU.

---
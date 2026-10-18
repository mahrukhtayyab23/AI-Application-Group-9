# AI Application - Group-9

## Install
```bash
pip install -r requirements.txt
```

---

## Project structure
```text
primagaz-yolo/
├── notebooks/
│   ├── 00_data_preparation.ipynb
│   ├── 01_gas_bottle_baseline.ipynb
│   ├── 02_video_tracking.ipynb
│   ├── 03_gas_cylinder_training.ipynb
│   ├── 04_gas_cylinder_evaluation.ipynb
│   ├── 05_cvat_custom_dataset.ipynb
│   └── 06_synchronised_tracking.ipynb
├── datasets/
├── videos/
├── results/
└── requirements.txt
```

---

## Notebooks

| Notebook | Purpose |
|---|---|
| 00_data_preparation | Extracts frames from the raw videos for CVAT annotation and testing. |
| 01_gas_bottle_baseline | Runs pretrained YOLO11n (COCO) on local LGP images. Shows the model has no cylinder class. |
| 02_video_tracking | Runs ByteTrack and BoT-SORT on short videos. |
| 03_gas_cylinder_training | Fine-tunes YOLO11n on the public Roboflow datasets. |
| 04_gas_cylinder_evaluation | Re-evaluates each public dataset `best.pt` on validation and test splits. |
| 05_cvat_custom_dataset | Merges the 3 CVAT batches, performs a scene-aware train/val split (to prevent data leakage), and trains the final YOLO11n model. |
| 06_synchronised_tracking | Runs the final CVAT model on the optimized videos to assign global, monotonically increasing IDs to bottles across different camera angles. |

*(Note: The raw video and dataset files must be downloaded or extracted into their respective folders before running the notebooks).*

---

## Datasets & Resources

**Roboflow Datasets (Public):**
| Dataset | Location | Classes | Link |
|---|---|---|---|
| CylinDeRS | `datasets/gas_bottles/` | `gas_cylinder` | https://universe.roboflow.com/klearchos-stavrothanasopoulos-konstantinos-gkountakos-6jwgj/cylinders-iaq6n/dataset/3 |
| Gas-cylinder detection | `datasets/gas_cylinder/` | `gas_cylinder`, `gas cylinder head` | https://universe.roboflow.com/obj-dect/gas-cylinder-detection/dataset/3 |

**Offline CVAT Datasets (Custom Labeled):**
- `datasets/cvat_batch_1a/`: NOK bottles provided by the lecturer.
- `datasets/cvat_batch_1b/`: Mixed OK/NOK bottles from the factory site.
- `datasets/cvat_batch_2/`: Extracted from YouTube reference videos.

**Reference Videos (videos_1):**
- Bolivia: FLEXSPEED for Side-Entry Valves (2014): https://www.youtube.com/watch?v=PtKnRpGvPM4
- Portugal: Petrogal LPG Cylinder Filling Plant (2013): https://www.youtube.com/watch?v=UH-u0g3B8Pw
- Automatic LPG Cylinder filling plant: https://www.youtube.com/shorts/sdo9Pg50GNY
- LPG CYLINDER FILLING CAROUSEL (ELECTRONIC 36 sec.): https://www.youtube.com/shorts/dvBFjJub6rE
- LPG CYLINDER FILLING CAROUSEL: https://www.youtube.com/shorts/dvBFjJub6rE
- LPG Cylinder Robotic Loading System: https://www.youtube.com/watch?v=TIVIFfRqbgU

**Primagaz Videos:**
- Provided by the lecturer (front, back left, back right views for both 14:43 and 14:55). 

**Synchronised Tracking Demos:**
- Front Camera: <br>
https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_55_front_cropped.mp4
https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_43_front_cropped.mp4
- Back Left Camera: <br>
https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_55_back_left_cropped.mp4
https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_43_back_left_cropped.mp4
- Back Right Camera: <br>
https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_55_back_right_cropped.mp4
https://github.com/mahrukhtayyab23/AI-Application-Group-9/blob/main/results/synchronised_tracking/demos/sync_optimized_14_43_back_right_cropped.mp4

---

## Model & Training Settings

**Model:** YOLO11n (initialized from COCO weights).

**Settings for the final CVAT model (Notebook 5):**
| Field | Value |
|---|---|
| Epochs | 25 |
| Image size | 416 |
| Batch size | 2 (limited by CPU RAM) |
| Device | CPU |

*Note: Public dataset training (Notebook 3) used 5 epochs and a fraction of 0.25 due to the large size of the CylinDeRS dataset.*

---

## Results

**Baseline:** The pretrained COCO model has no cylinder class. It labels cylinder-shaped objects as vase, bottle, or fire hydrant, or misses them entirely.

**Tracking:** ByteTrack is faster than BoT-SORT. 

**Public Datasets (Notebook 4):**

| Dataset | Split | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|---|
| gas_bottles | val | 0.9170 | 0.6864 | 0.8866 | 0.8421 |
| gas_bottles | test | 0.9321 | 0.6845 | 0.8952 | 0.8545 |
| gas_cylinder | val | 0.7896 | 0.3108 | 0.3449 | 0.1263 |
| gas_cylinder | test | 0.8344 | 0.3421 | 0.3775 | 0.1567 |

**Custom CVAT Dataset (Final Model for WS4):**

| Dataset | Split | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|---|
| cvat_merged | val | 0.8312 | 0.6765 | 0.7345 | 0.5320 |

Training on our site-specific CVAT data improved the mAP50 from ~34% (public dataset) to 73.4%, proving the necessity of domain-specific annotations.

---

## Known limitations
- The pre-trained COCO model has no cylinder class.
- Models trained on public datasets (Roboflow) fail to generalize well to our specific factory videos.
- The CVAT dataset is currently imbalanced (more NOK than OK bottles), which will affect the future OK/NOK classification model.
- All training was done on CPU.
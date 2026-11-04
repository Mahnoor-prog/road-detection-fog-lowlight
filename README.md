# Robust Road Object Detection in Fog and Low Light (YOLOv8n)

## Problem
Object detectors trained on clear images can lose accuracy in low visibility.
This project measures that drop on a small road-scene dataset and tests whether
adding fog/dark images to training reduces it.

## Dataset
- Vishi Sapra, "Object detection", Roboflow Universe (version 10), CC BY 4.0
- 900 images, 5 classes: bus, car, person, traffic signal, truck
- https://universe.roboflow.com/vishi-sapra/object-detection-axukj/dataset/10
- The dataset is not included in this repo.
- Some label files contained polygon rows; I converted them to bounding boxes.

## Method
- Model: YOLOv8n (pretrained), 40 epochs, image size 416, batch 16
- Synthetic corruption with OpenCV/NumPy:
  - fog (strength 0.5 / 0.7 / 0.85)
  - darkening (factor 0.35 / 0.2 / 0.12)
- Baseline: trained on the original training split
- Augmented: original training split plus one fog or dark copy of each image
  (fog strength 0.3-0.9, dark factor 0.12-0.5)
- Both models are evaluated on the same corrupted copies of the validation
  and test splits

## Results (car + person mAP50, test split)
| Condition | Baseline | Augmented |
|---|---|---|
| clean | 0.974 | 0.976 |
| fog 0.5 | 0.945 | 0.967 |
| fog 0.7 | 0.914 | 0.966 |
| fog 0.85 | 0.808 | 0.979 |
| dark 0.35 | 0.976 | 0.978 |
| dark 0.2 | 0.968 | 0.980 |
| dark 0.12 | 0.934 | 0.980 |

Under the heaviest fog the baseline drops from 0.974 to 0.808. The augmented
model recovers to 0.979 with no meaningful change on clean images.
Differences below about 0.02 are within noise.

### Detection counts under heavy fog (fog 0.85, test split, car + person)
Confidence 0.25, IoU 0.5, 100 images, 148 ground-truth objects. This is a simple
per-image count, not the full mAP computation.

| Model | Hits | Misses | Extra boxes |
|---|---|---|---|
| Baseline | 92 | 56 | 18 |
| Augmented | 141 | 7 | 21 |

The augmented model finds many more objects but produces slightly more extra boxes.

## Limitations
- Fog and darkness are synthetic, and the same functions were used for training
  and evaluation. This does not show improvement on real fog.
- The synthetic fog is blocky and does not look like real atmospheric fog.
- The test split has 100 images. Bus (1), traffic signal (2) and truck (6) have
  too few test instances, so I report car + person only.
- Single training run, one seed.
- The augmented model saw about twice as many training images as the baseline.
- The best checkpoint was selected on the validation split, so validation
  numbers are slightly optimistic. The test split is the main result.

## Future work
- Evaluate on real foggy data (for example RTTS) and a larger dataset (BDD100K)
- Compare augmentation against dehazing before detection
- Try architecture-level robustness (attention modules, domain adaptation)
- Multiple seeds and confidence intervals

## Run
Open `notebook.ipynb` in Google Colab and run the cells in order. Add your own
Roboflow API key where the notebook says `YOUR_API_KEY`.

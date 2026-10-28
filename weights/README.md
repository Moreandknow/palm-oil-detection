# Model Checkpoints & Weights

Due to GitHub's file size limits, trained model weights (`.pth`, `.pt`) are hosted externally on Google Drive / Hugging Face Hub instead of being tracked directly by Git.

## 📥 Download Links

| Model Architecture | Variant | File Name | Size | Download Link |
| :--- | :--- | :--- | :--- | :--- |
| **Faster R-CNN (ResNet-50 FPN)** | Baseline Best | `best_faster_rcnn_resnet50_fpn.pth` | ~322 MB | [Download Link](#) |
| **Faster R-CNN (ResNet-50 FPN)** | Baseline Last | `last_faster_rcnn_resnet50_fpn.pth` | ~322 MB | [Download Link](#) |
| **Faster R-CNN (ResNet-50 FPN)** | Fine-Tuned Best | `best_faster_rcnn_finetuned.pth` | ~162 MB | [Download Link](#) |
| **YOLOv11** | Best Weights | `best_yolov11.pt` | -- | [Download Link](#) |

## 🛠️ Setup Instructions

1. Download the required checkpoint files from the links above.
2. Place all downloaded files directly into this directory (`weights/`).
3. Ensure the filenames match the table above before running evaluation or inference scripts.
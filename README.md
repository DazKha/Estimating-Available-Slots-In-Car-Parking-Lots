# 🅿️ Estimating Available Slots in Car Parking Lots

> A lightweight CNN-based system for real-time parking slot occupancy detection using snapshot image classification — no video streaming required.

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=flat&logo=opencv&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat&logo=pytorch&logoColor=white)
![Accuracy](https://img.shields.io/badge/Accuracy-92.74%25-22c55e?style=flat)
![Latency](https://img.shields.io/badge/Latency-≤2s-3b82f6?style=flat)

---

## 🎯 Problem

Managing parking availability manually is inefficient and error-prone — especially in large, multi-zone lots. Existing video-streaming solutions require high compute, temporal tracking, and complex motion analysis.

**Our approach:** Treat it as a periodic **per-slot image classification** task instead.

- Capture one snapshot every 5 seconds (no full video stream)
- Classify each predefined slot ROI independently as `occupied` or `empty`
- Visualize results with colored bounding boxes in real time

This works because parked vehicles are **stationary for 10–30+ minutes** — real-time tracking is unnecessary overhead.

---

## ⚙️ How It Works

```
Camera (top-down, 4–6m height)
        ↓  every 5 seconds
  Snapshot Frame (1000×750 JPEG)
        ↓
  Align Slot Layout Blueprint
  (manual calibration at setup, reused every frame)
        ↓
  Extract Slot ROIs
  (OpenCV crop — each slot → 150×150 patch)
        ↓
  CNN Binary Classifier
  (occupied / empty per slot)
        ↓
  Annotate Frame
  🟥 Red box  → occupied
  🟩 Green box → empty
        ↓
  Display: slot count + annotated image
```

This loop repeats every 5 seconds — fast enough to catch arrivals/departures, light enough to run on modest hardware.

---

## 📊 Evaluation

| Metric | Result | Requirement |
|---|---|---|
| **Accuracy** | **92.74%** | ≥ 90% (daylight / twilight) |
| **Precision (avg)** | **91.15%** | Minimize false availability claims |
| **Latency** | **≤ 2 seconds** | Timely slot status updates |
| **Frame Interval** | 5 seconds | Optimized load, near real-time |
| **Scalability** | Tested up to 40 slots | Supports large parking layouts |
| **Low-light Robustness** | ~88% | Maintains accuracy in twilight |
| **Privacy Compliance** | ✅ Verified | No license plates / faces captured |

---

## 🧠 Model

**Architecture:** Lightweight CNN trained for binary image classification (`busy` / `free`)

**Input:** 150×150 RGB patch of a single parking slot  
**Output:** Binary label — `occupied` or `empty`

**Design choice — why CNN over object detection:**
- No need to locate or segment vehicles explicitly
- Each slot is a fixed-size region — pure appearance-based classification suffices
- Significantly lighter than YOLO/RCNN pipelines
- Easily transferable to new parking lots by re-aligning slot coordinates

---

## 📁 Dataset

**CNR-EXT** (publicly available) — a standard benchmark for parking slot classification.

```
CNR-EXT_FULL_IMAGE_1000x750/     ← Full frames (multi-camera, multi-weather)
├── camera1.csv ... cameraN.csv  ← Slot layout maps
└── FULL_IMAGE_1000x750/
    ├── OVERCAST/
    ├── RAINY/
    └── SUNNY/

CNR-EXT-Patches-150x150/         ← Pre-cropped 150×150 slot patches
├── LABELS/
└── PATCHES/
    ├── OVERCAST/
    ├── RAINY/
    └── SUNNY/

CNRPark-Patches-150x150/         ← Additional split by camera zone
├── A/  ├── busy/  └── free/
└── B/  ├── busy/  └── free/
```

Images captured under varied **weather** (sunny, overcast, rainy) and **lighting** conditions across multiple fixed overhead cameras.

---

## 🔧 Technical Constraints & Scope

**Camera requirements:**
- Fixed angle, top-down view (4–6m height, pitch 75°–85°)
- Each slot must be **fully visible** in frame — partial occlusion reduces reliability
- Multi-camera support: each camera handles its own independent zone

**Operating conditions:**
- Daylight and moderate twilight ✅
- Nighttime ❌ (not supported)
- Rain / fog / harsh shadows may reduce performance

**Out of scope:**
- License plate recognition
- Vehicle type classification (cars only — no motorcycles, trucks, buses)
- Motion tracking or trajectory analysis
- Illegal/abnormal parking detection
- Dynamic layout discovery at runtime

---

## 🚀 Quick Start

```bash
git clone https://github.com/DazKha/Estimating-Available-Slots-In-Car-Parking-Lots
cd Estimating-Available-Slots-In-Car-Parking-Lots
pip install -r requirements.txt
```

**Run inference on a sample image:**
```bash
python main.py --image path/to/parking_frame.jpg --layout path/to/layout.csv
```

**Output:** Annotated image with 🟥/🟩 bounding boxes + available slot count printed to console.

---

## 🛡️ Ethics & Privacy

| Concern | How We Handle It |
|---|---|
| **Privacy** | Top-down camera angle — no faces or license plates captured |
| **Data retention** | Frames discarded immediately after classification, no storage |
| **Transparency** | All camera locations marked with visible signage |
| **Fairness** | Identical model and criteria applied uniformly to all slots |

---

## 👥 Team — CS117.P21, UIT – VNU-HCM

| No. | Name | Student ID | Role |
|---|---|---|---|
| 1 | Lê Minh Kha | 23520664 | Team Leader |
| 2 | Trần Quang Minh | 23520958 | Member |
| 3 | Đoàn Việt Hoàng | 23520535 | Member |
| 4 | Nguyễn Vũ Khang | 23520701 | Member |
| 5 | Nguyễn Hải Đăng | 23520228 | Member |

**Course:** CS117 — Computational Thinking  
**Supervisor:** Dr. Ngo Duc Thanh · UIT, VNU-HCM

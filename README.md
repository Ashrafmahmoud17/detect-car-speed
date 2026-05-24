# 🚗 Vehicle Speed Detection System
### YOLOv8 | Real-Time Speed Estimation & Violation Detection

---

## 📌 Overview

A real-time **vehicle speed detection and violation system** that uses **YOLOv8** to detect cars in a video, estimates their speed using a two-line crossing method, saves the output video with annotations, and automatically captures photos of speed limit violators.

The system evolved through **4 versions** — from basic detection to full violation logging.

---

## 🧠 How It Works

```
Video Input
     │
     ▼
YOLOv8 (Detect Cars — class_id = 2)
     │
     ▼
Draw Two Virtual Lines (Line 1 & Line 2)
     │
     ▼
Car crosses Line 1 → Record frame number / timestamp
     │
     ▼
Car crosses Line 2 → Calculate elapsed time
     │
     ▼
Speed = (Distance ÷ Time) × 3.6   →   km/h
     │
     ▼
Speed > Limit?
  ├── YES → Save violation photo + log
  └── NO  → Display speed only
     │
     ▼
Save annotated output video
```

---

## 📁 Project Structure

```
car-speed-detection/
│
├── v1_basic_detection.py        # Basic YOLOv8 object detection on video
├── v2_save_video.py             # Detection + save annotated output video
├── v3_speed_time.py             # Speed estimation using timestamps
├── v4_violations.py             # Full system: speed + violations + save
│
├── yolov8n.pt                   # YOLOv8 nano model (auto-downloaded)
│
├── speed_output.mp4             # Output video with bounding boxes & speeds
│
└── speed_violations/            # Cropped photos of speeding vehicles
    ├── car_3_85kmh.jpg
    ├── car_7_102kmh.jpg
    └── ...
```

---

## ⚙️ Requirements

### Python Version
```
Python 3.8+
```

### Install Libraries
```bash
pip install ultralytics
pip install opencv-python
pip install Pillow
```

> `yolov8n.pt` is automatically downloaded on first run.

---

## 🚀 Project Versions

### Version 1 — Basic Detection
Detects all 80 COCO objects in a video and displays results in Jupyter Notebook.
```python
# Detects: person, car, truck, bus, motorcycle, etc.
model = YOLO('yolov8n.pt')
```

### Version 2 — Save Output Video
Adds video saving with bounding boxes and confidence scores drawn on each frame.
```python
out = cv2.VideoWriter('output.mp4', fourcc, fps, (width, height))
out.write(frame)
```

### Version 3 — Speed Estimation (Time-Based)
Estimates speed using two virtual lines and real timestamps.
```python
# Car crosses Line 1 → start timer
# Car crosses Line 2 → stop timer
speed = (distance_meters / elapsed_time) * 3.6  # km/h
```

### Version 4 — Full System with Violations ✅
Complete system: speed estimation + speed limit enforcement + violation photo capture.
```python
if vehicle_speeds[idx] > speed_limit:
    cv2.imwrite(f"car_{idx}_{speed}kmh.jpg", car_crop)
```

---

## 🎛️ Configuration

All key settings are at the top of the script:

```python
# Virtual line positions (pixels from top of frame)
line1_y = 250       # First line  (green)
line2_y = 400       # Second line (red)

# Real-world distance between the two lines
distance_meters = 10    # Adjust based on actual scene calibration

# Speed limit
speed_limit = 60    # km/h — vehicles above this are flagged

# Output paths
save_folder = "speed_violations/"
output_video = "speed_output.mp4"
```

---

## 📊 Speed Calculation

```
Speed (km/h) = (Distance in meters ÷ Time in seconds) × 3.6

Example:
  Distance = 10 meters
  Time     = 0.42 seconds
  Speed    = (10 / 0.42) × 3.6 = 85.7 km/h
```

> **Important:** `distance_meters` must be calibrated to the actual real-world distance between the two lines in the video scene. Use landmarks or road markings as reference.

---

## 🔍 Detection Details

| Setting | Value |
|---|---|
| Model | YOLOv8 Nano (`yolov8n.pt`) |
| Detected Class | Car only (`class_id = 2`) |
| Detection Source | COCO dataset (80 classes) |
| Line Crossing Tolerance | ±10 pixels |
| Speed Calculation | Frame-based (v4) or Time-based (v3) |

---

## 💾 Output Files

### Annotated Video (`speed_output.mp4`)
- Green line = Line 1 (entry)
- Red line = Line 2 (exit)
- Blue boxes = detected vehicles
- Yellow text = estimated speed in km/h

### Violation Photos (`speed_violations/`)
- Cropped image of the vehicle at the moment of detection
- Filename format: `car_{id}_{speed}kmh.jpg`

---

## ⚠️ Limitations & Notes

- **No object tracking** — uses frame index as car ID, which may cause ID switches between frames. For production use, integrate **SORT** or **ByteTrack**.
- **Calibration required** — `distance_meters` must match the actual scene geometry for accurate speed readings.
- **Lighting conditions** — detection accuracy drops in very dark or overexposed footage.
- **Camera angle** — a top-down or slight overhead angle gives more accurate speed readings than a low angle.

---

## 🔧 Improving Accuracy

### Use Frame-Based Speed (More Stable)
```python
frames_passed = frame_count - car_entry_frame[car_id]
time_sec = frames_passed / fps
speed = (distance_m / time_sec) * 3.6
```

### Add Real Object Tracking
```bash
pip install lap
```
```python
# Use YOLO built-in tracker
results = model.track(frame, persist=True, verbose=False)[0]
track_ids = results.boxes.id.int().cpu().tolist()
```

---

## 👤 Author

**Ashraf Mahmoud**
Computer Sciences — New Mansoura University

---

## 📄 License

This project is for educational purposes.

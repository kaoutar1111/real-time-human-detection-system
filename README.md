# YOLO-Based Personnel Detection Desktop Application

A real-time desktop application that detects people in images from industrial Hikvision cameras. It combines YOLOv8 detection, a Tkinter interface and SQLite logging in one program.

Graduation project, Computer Science and Technology, China Jiliang University (2026).

![Application screenshot](screenshots/app.png)

## Why this project

Security staff who watch many cameras at once miss events because of attention fatigue, and most footage is only reviewed after an incident. Strong AI models and industrial cameras exist, but there is rarely a simple desktop application that connects them. This project joins them in one tool that runs on a normal Windows computer, without expensive commercial software.

## Features

- **Real-time detection:** YOLOv8 processes the camera frames, draws red boxes around people, counts them, and has an adjustable confidence threshold.
- **Camera control:** connect and disconnect Hikvision cameras, adjust exposure and gain, and switch between continuous and external (software) trigger modes.
- **Offline image mode:** load a local JPG, PNG, BMP or TIFF file and run the same detection without a camera.
- **SQLite logging:** every scan is saved in three tables: scan summary (`scans`), box coordinates and confidence (`scan_details`), and timestamped `logs`.
- **Statistics tab:** total scans, total persons, average per scan, daily count and success rate, updated after each scan.
- **Three tabs:** Camera Control, Logs and Statistics.

## Architecture

| Layer | Component |
| --- | --- |
| Presentation | Tkinter GUI with three tabs |
| Application | `HikCameraApp`, the main controller and event loop |
| Hardware and AI | `HikCamera` (Hikvision MVS SDK) and `YoloDetector` (YOLOv8) |
| Data | `DatabaseManager` (SQLite) |

**Workflow:** launch the app, enumerate cameras on the network, connect and acquire frames, run YOLO inference for the person class, draw and display the boxes, then write the result to the database.

## Requirements

- Windows
- Python 3.10
- Hikvision MVS SDK installed (only needed for live camera mode)
- A Hikvision camera on the local network (offline image mode works without one)

## Installation

```bash
git clone https://github.com/kaoutar1111/real-time-human-detection-system.git
cd real-time-human-detection-system
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

The model file `yolov8n.pt` is in the repository. If it is missing, run `python download_model.py`.

## Usage

```bash
python main.py
```

- **Live camera:** pick a camera in the list, click **Connect**, then start detection. Use the trigger mode to capture and analyse a single frame.
- **Offline images:** load an image file and run detection without a camera.
- Logs and statistics are shown in their own tabs. The database `app_data.db` is created automatically on first run.

## Project structure

| File | Purpose |
| --- | --- |
| `main.py` | Interface and application logic |
| `hik_camera.py` | Camera connection through the MVS SDK |
| `yolo_detector.py` | Person detection with YOLOv8 |
| `database.py` | SQLite storage |
| `download_model.py` | Downloads the YOLOv8 model |
| `test_camera.py`, `test_yolo_install.py` | Checks for the camera and the YOLO install |

## Results

Measured on a CPU-only computer with the YOLOv8 nano model:

| Measure | Result |
| --- | --- |
| Functional tests | 8 of 8 passed |
| Inference time | 115 to 145 ms per frame |
| Live feed, detection off | 20 to 25 fps |
| Live feed, detection on | 6 to 8 fps |
| CPU load | about 4 to 8% (camera only), about 24% (detection on) |
| Memory | about 140 MB at start, about 380 MB with the model loaded |
| Database write | under 3 ms per transaction |
| Trigger cycle (capture, detect, save) | about 280 ms |
| Stress test | 2 hours continuous, no crashes or memory leaks |

## Limitations

- On CPU only, the live feed with detection drops to 6 to 8 fps.
- Accuracy falls in poor lighting, and partly hidden people can be missed.
- Detects people only, not other object types.
- Desktop only, and no RTSP stream support (Hikvision MVS SDK cameras only).

## Future work

- RTSP support, so any camera brand can be used
- Multi-class detection (vehicles, assets)
- Custom YOLO training on a local dataset
- Heatmaps and foot-traffic analytics
- Cloud storage, and a web dashboard (Flask and React)

## Tech

Python 3.10, YOLOv8 (Ultralytics), OpenCV, NumPy, Tkinter, SQLite, Hikvision MVS SDK

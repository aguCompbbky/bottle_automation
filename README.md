# Bottle Automation

A computer vision-based bottle inspection system that automatically detects and classifies bottles using YOLO object detection and integrates with Arduino for automated quality control feedback.

## Overview

This project implements an automated bottle inspection pipeline that:
- Captures video frames from a camera (Raspberry Pi or USB camera)
- Detects bottles using YOLO object detection
- Classifies bottles as **ACCEPT**, **REJECT**, or **UNCERTAIN**
- Sends commands to Arduino for physical feedback (e.g., pneumatic rejection)
- Displays real-time preview with detection results

## Features

- **Dual-Stage Detection**:
  - Stage 1: Bottle presence detection (COCO bottle class)
  - Stage 2: Custom bottle fill-level classification (accept/reject)

- **Multi-Device Support**:
  - Raspberry Pi with libcamera
  - USB cameras on laptops/desktops

- **Arduino Integration**:
  - USB serial communication
  - Mock mode for testing without hardware

- **Real-Time Preview**:
  - Annotated video overlay with detection results
  - Color-coded status (green=accept, red=reject, orange=no bottle)
  - Confidence scores

- **Configurable Confidence Thresholds**:
  - Separate thresholds for accept/reject decisions
  - Bottle detection gate to prevent false positives

## Architecture

```
├── main.py                          # Entry point & main loop
├── config.py                        # Configuration constants
├── requirements.txt                 # Python dependencies
│
├── hardware/
│   └── camera_service.py            # Camera abstraction (Raspberry Pi / USB)
│
├── image_detection/
│   └── detector.py                  # YOLO-based bottle detector
│
├── model/
│   └── best.pt                      # Pre-trained YOLO model (custom)
│
└── communicationwardunio/
    └── arduino_client.py            # Arduino serial communication
```

## Installation

### Requirements
- Python 3.8+
- OpenCV (`opencv-python`)
- YOLO detection library (`ultralytics`)
- NumPy
- PySerial (for Arduino communication)

### Setup

1. Clone the repository:
```bash
git clone https://github.com/aguCompbbky/bottle_automation.git
cd bottle_automation
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. For Raspberry Pi, install additional camera library:
```bash
# Uncomment in requirements.txt and install
pip install picamera2
```

4. Prepare the model:
   - Place your trained YOLO model at `model/best.pt`
   - Default expects `yolov8n.pt` for bottle detection (auto-downloaded)

## Usage

### Basic Usage (Preview Mode)
```bash
python main.py --device raspberry
```

### Available Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `--device` | `raspberry` | Device type: `raspberry` or `laptop` |
| `--model` | `model/best.pt` | Path to custom YOLO model |
| `--accept-conf` | `0.95` | Confidence threshold for ACCEPT decision |
| `--reject-conf` | `0.85` | Confidence threshold for REJECT decision |
| `--conf` | `None` | Override both accept/reject thresholds |
| `--bottle-conf` | `0.45` | Confidence threshold for bottle detection |
| `--no-bottle-gate` | `False` | Disable bottle presence check (not recommended) |
| `--interval` | `0.05` | Frame processing interval in seconds |
| `--serial-port` | `/dev/ttyACM0` | Arduino serial port |
| `--baud-rate` | `115200` | Arduino baud rate |
| `--arduino-mode` | `usb` | Communication mode: `usb` or `mock` |
| `--no-preview` | `False` | Disable display preview |
| `-v, --verbose` | `False` | Enable debug logging |

### Examples

**Test with USB camera and mock Arduino:**
```bash
python main.py --device laptop --arduino-mode mock -v
```

**Custom thresholds for stricter rejection:**
```bash
python main.py --accept-conf 0.98 --reject-conf 0.80
```

**Production mode (no preview, USB Arduino):**
```bash
python main.py --device raspberry --no-preview
```

**Press `q` during preview to gracefully exit**

## Configuration

Edit `config.py` to customize:

### Camera Settings
- `CAMERA_WIDTH`, `CAMERA_HEIGHT` - Frame resolution
- `DEFAULT_FRAME_INTERVAL` - Processing delay between frames
- `CAMERA_SCAN_INTERVAL_SEC` - Camera reconnection check interval

### Detection Settings
- `ACCEPT_CONF` - Threshold for ACCEPT classification
- `REJECT_CONF` - Threshold for REJECT classification
- `BOTTLE_DETECT_CONF` - Bottle presence detection threshold
- `BOTTLE_CROP_PADDING` - Crop padding around detected bottle

### Visual Settings
- `COLOR_ACCEPT_BGR`, `COLOR_REJECT_BGR`, `COLOR_NO_BOTTLE_BGR` - Border colors
- `FRAME_BORDER_THICKNESS` - Border thickness in pixels

### Arduino Settings
- `ARDUINO_MODE` - `"usb"` or `"mock"`
- `DEFAULT_SERIAL_PORT` - Serial port path (e.g., `/dev/ttyACM0`)
- `DEFAULT_BAUD_RATE` - Serial baud rate (default: `115200`)
- `ARDUINO_REJECT_CMD` - Command sent on rejection

## Output States

The detector classifies each frame into one of four states:

| State | Description | Border Color |
|-------|-------------|--------------|
| `accept` | Bottle present and meets accept criteria | Green |
| `reject` | Bottle present but rejected; Arduino notified | Red |
| `uncertain` | Confidence between accept/reject thresholds | Orange |
| `no_bottle` | No bottle detected or below detection gate | Orange |

## Hardware Integration

### Arduino Communication
- **Protocol**: Simple serial command (`REJECT\n`)
- **Baud Rate**: 115200 (configurable)
- **Modes**:
  - `usb`: Real USB serial communication
  - `mock`: Simulated mode for testing

On rejection, the system sends:
```
REJECT\n
```

Your Arduino sketch should listen for this command and trigger the reject mechanism (e.g., pneumatic valve).

## Logging

The system logs to console with timestamps. Enable verbose output for debugging:
```bash
python main.py -v
```

Log format:
```
HH:MM:SS | LEVEL   | Message
14:23:45 | INFO    | Starting camera service...
14:23:46 | DEBUG   | Frame detected: ID=42 ...
```

## Troubleshooting

### Camera Not Detected
- Check device path: `ls /dev/video*` (USB) or test libcamera on Raspberry Pi
- Increase camera timeouts in `config.py`

### Arduino Not Communicating
- Verify serial port: `ls /dev/ttyACM0`
- Check baud rate matches your Arduino sketch
- Use `--arduino-mode mock` to test without hardware

### Low Detection Accuracy
- Adjust confidence thresholds (`--accept-conf`, `--reject-conf`)
- Retrain model with better dataset
- Ensure proper lighting conditions

### Preview Not Showing
- Verify X11 forwarding if using SSH
- Use `--no-preview` flag for production
- Install `python3-tk` if needed for display

## Language Composition
- **Python**: 81.2% (main logic)
- **Shell**: 10.9% (scripts)
- **C++**: 7.9% (hardware drivers/dependencies)

## Dependencies

```
ultralytics>=8.0.0       # YOLO detection
opencv-python>=4.8.0    # Image processing
numpy>=1.24.0           # Numerical operations
pyserial>=3.5           # Arduino communication
picamera2               # Raspberry Pi camera (optional)
```

## License

[Specify your license here]

## Contributing

Contributions are welcome! Please submit issues and pull requests to improve the system.

## Author

**aguCompbbky**

---

**Last Updated**: May 2026

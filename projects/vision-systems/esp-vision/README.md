# ESP Vision

A lightweight embedded vision project for ESP32-class boards.

ESP Vision is intended to become a compact and practical image-processing and vision library for constrained embedded hardware, starting with ESP32-CAM.

The project follows a staged engineering process: first characterize the hardware and validate the camera driver, then define the backend contracts, and only after that design the core image model and processing layers.

## Goals

- Build a lightweight and realistic vision stack for ESP32-class hardware
- Keep the architecture small, understandable, and hardware-aware
- Provide clean low-level abstractions for frame and image handling
- Focus first on practical grayscale image processing
- Validate each layer before moving upward

## Non-Goals

At the initial stage, this project is not trying to be:

- a full OpenCV replacement
- a desktop-oriented image library
- a generic GPU-style rendering framework
- an unbounded abstraction detached from embedded constraints

## Target Hardware

Initial target profile:

- ESP32 classic
- Dual-core Xtensa LX6
- 240 MHz
- ESP32-CAM class board
- PSRAM-enabled configuration
- ESP-IDF-based firmware environment

## Development Strategy

The project is developed layer by layer.

No upper layer is implemented before the lower layer is documented, validated, and accepted.

## Layer Roadmap

### 01. Hardware Characterization
Identify the exact chip, memory profile, heap behavior, PSRAM availability, and reproducible hardware limits.

### 02. Driver Validation
Validate camera driver behavior, supported formats, stable frame sizes, frame buffer behavior, and capture timing.

### 03. Backend / HAL
Define low-level contracts for memory, timing, logging, and camera-facing services required by higher layers.

### 04. Core Image Model
Define the foundational image and frame representations such as raw frame ownership, image layout, stride, ROI, and format contracts.

### 05. ImgProc
Implement basic image-processing operations with a priority on GRAY8 and embedded-friendly performance.

### 06. Drawing
Add lightweight drawing primitives such as lines, rectangles, circles, and overlays.

### 07. Vision API
Build higher-level functionality such as blob detection, motion detection, and compact perception helpers.

### 08. Applications
Provide practical demos, validation apps, and real use-case examples.

## Project Structure
```text
esp-vision/
├── docs/
│   ├── layers/
│   ├── decisions/
│   ├── hardware/
│   └── api/
├── experiments/
│   ├── hw-characterization/
│   └── driver-validation/
├── firmware/
│   └── esp-idf/
├── benchmarks/
└── examples/

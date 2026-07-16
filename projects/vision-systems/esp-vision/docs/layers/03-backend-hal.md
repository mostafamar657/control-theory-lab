# 03 — Backend / HAL

## 1. Overview

Backend / HAL is the low-level service layer of ESP Vision.  
Its purpose is to isolate the rest of the project from ESP-IDF-specific details and provide a small, stable, deterministic API for hardware-adjacent operations.

This layer does **not** perform image processing.  
It only provides foundational services needed by higher layers:

- memory management
- time and timestamp utilities
- logging
- camera lifecycle and frame access
- platform capability queries
- unified status/error handling

The design goal is to keep the interface minimal, explicit, and predictable.

---

## 2. Responsibilities

The Backend / HAL layer is responsible for:

- abstracting ESP-IDF dependencies away from upper layers
- providing a unified error model
- offering controlled memory allocation APIs
- exposing monotonic time and timing helpers
- wrapping logging behind project-owned interfaces
- managing camera initialization, capture, and release
- exposing hardware and driver capabilities in a structured way

---

## 3. Non-Goals

This layer explicitly does **not** include:

- image preprocessing
- image conversion pipelines
- resize, crop, normalize, or filter operations
- scheduling or task orchestration
- network communication
- storage/file-system logic
- multi-camera coordination
- high-level vision algorithms
- application-specific pipeline logic

These concerns belong to higher layers or separate modules.

---

## 4. Design Principles

### 4.1 C-first API
The public interface must be C-friendly and lightweight.  
The API should not require C++ features or complex ownership semantics.

### 4.2 Explicit ownership
Memory ownership and frame ownership must be clearly documented.

### 4.3 Deterministic behavior
The HAL should favor predictable runtime behavior over abstraction complexity.

### 4.4 Minimal abstraction
Only abstractions that are needed by the project should be introduced.  
Avoid general-purpose interfaces that are not justified by current use cases.

### 4.5 No ESP-IDF leakage to upper layers
Higher layers must not depend directly on:

- `esp_err_t`
- `heap_caps_*`
- `esp_timer_*`
- `ESP_LOGx`
- `esp32-camera` internals

---

## 5. Scope

### Included
- status codes and result handling
- memory allocation and capability queries
- time and delay utilities
- logging interface
- camera init/deinit
- camera capability query
- frame capture and release

### Excluded
- pixel manipulation
- frame preprocessing
- camera driver tuning logic outside validated configuration
- algorithmic processing
- application pipelines

---

## 6. Layer Dependencies

This layer depends on:

- Stage 01: Hardware Characterization
- Stage 02: Driver Validation
- ESP-IDF
- `esp32-camera`

The layer must be implemented using only validated hardware and driver assumptions.

---

## 7. Status / Error Model

The project uses a unified status type instead of exposing ESP-IDF error codes directly.

### Expected status categories
- `ESPV_OK`
- `ESPV_ERR_INVALID_ARG`
- `ESPV_ERR_NOT_SUPPORTED`
- `ESPV_ERR_NO_MEM`
- `ESPV_ERR_TIMEOUT`
- `ESPV_ERR_IO`
- `ESPV_ERR_HW`
- `ESPV_ERR_STATE`
- `ESPV_ERR_BUSY`
- `ESPV_ERR_INTERNAL`

### Requirements
- every public API must return a project status code
- status codes must be documented
- conversion from platform-specific error codes must happen inside the backend implementation

---

## 8. Memory API

The memory API provides controlled allocation with optional capability hints.

### Goals
- hide ESP-IDF allocation details from upper layers
- support internal RAM and PSRAM policies
- make memory ownership explicit
- support future profiling and diagnostics

### Expected features
- allocate/free
- capability-aware allocation
- memory info queries
- optional fallback policy control

### Notes
- the caller owns allocated memory
- free must be performed through the matching HAL function
- allocation failure must be reported with a clear status code

---

## 9. Time API

The time API provides monotonic timestamps and delay helpers.

### Goals
- provide stable timing for benchmarks and profiling
- avoid direct dependency on `esp_timer` in upper layers
- keep timing units explicit

### Expected features
- current time in microseconds
- current time in milliseconds
- elapsed time calculation
- sleep/delay helper when needed

### Notes
- time should be monotonic where possible
- microseconds are the preferred reference unit for profiling

---

## 10. Logging API

The logging API provides project-owned logging macros and a minimal runtime interface.

### Goals
- keep logging consistent across layers
- avoid direct use of ESP-IDF logging APIs in upper layers
- support log level control

### Expected features
- set log level
- write log messages by level and tag
- convenience macros for common levels

### Notes
- logging must remain lightweight
- hot-path functions should not depend on heavy formatting work

---

## 11. Camera API

The camera API is the most important part of Backend / HAL.

### Goals
- initialize and configure the camera
- query supported capabilities
- capture a frame
- release a frame safely
- expose only validated behavior

### Lifecycle
1. initialize camera
2. optionally query capabilities
3. capture frame
4. process frame in higher layer
5. release frame
6. deinitialize camera

### Requirements
- camera ownership rules must be explicit
- frame validity must be clearly defined
- release must be the only allowed way to return driver-owned buffers
- invalid state calls must return a status error

### Frame handling
A frame must contain enough metadata for upper layers to understand:

- data pointer
- size in bytes
- width
- height
- pixel format
- timestamp
- ownership/backend context if needed

### Frame immutability
Captured frames should be treated as immutable by upper layers unless a specific API explicitly states otherwise.

---

## 12. Supported Pixel Formats

The HAL should only expose formats validated by Stage 02.

Expected public formats may include:

- `ESPV_PIXFMT_JPEG`
- `ESPV_PIXFMT_RGB565`
- `ESPV_PIXFMT_YUV422`
- `ESPV_PIXFMT_GRAYSCALE`

### Preferred processing format
For upper layers, `GRAYSCALE` is the preferred baseline when supported by the sensor and driver with acceptable stability and cost.

### Important
No conversion pipeline should be introduced inside HAL unless it becomes a justified requirement.

---

## 13. Capability Query

The HAL must expose a structured capability query for hardware and driver constraints.

### Example capabilities
- PSRAM support
- supported frame sizes
- supported pixel formats
- maximum stable capture size
- framebuffer count behavior
- sensor identity
- timestamp support

### Purpose
Capabilities are derived from characterization and driver validation.  
They are not guessed at runtime by upper layers.

---

## 14. Configuration Model

The layer should support both build-time and runtime configuration.

### Build-time configuration
Examples:
- default log level
- memory policy defaults
- feature toggles
- timeout defaults

### Runtime configuration
Examples:
- camera pixel format
- frame size
- framebuffer count
- capture mode
- capture timeout

---

## 15. Implementation Organization

A suggested ESP-IDF backend organization:
```text
firmware/esp-idf/
└── components/
└── esp_vision_backend/
├── include/
├── src/
├── test/
└── CMakeLists.txt

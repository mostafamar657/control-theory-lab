# ADR 001: ESP Vision Concurrency & Dual-Core Policy

## Status
Proposed

## Context
ESP32 boards provide two processing cores (Core 0 and Core 1). To ensure optimal performance for embedded computer vision—where low-latency frame acquisition and intensive image processing must happen simultaneously—a strict concurrency policy is required.

## Policy
The system adopts a "Core-Pinned" architecture to minimize jitter and resource contention.

### 1. Core 0: Protocol & I/O Core
- **Responsibility:**
  - Wi-Fi/Bluetooth stack and network communication.
  - Camera driver (SCCB/I2C communication, DMA transfer, Interrupt Service Routines).
  - Background system monitoring (Watchdog, FreeRTOS system tasks).
- **Goal:** To ensure the camera interface maintains a consistent frame rate by keeping it away from heavy computation tasks.

### 2. Core 1: Vision & Control Core
- **Responsibility:**
  - Core Image Model processing (ImgProc, Computer Vision algorithms).
- **Goal:** To dedicate the maximum possible CPU cycles to image processing and control loop logic without being preempted by network interrupts.

## Implementation Details
- The `Backend/HAL` layer shall provide task abstraction primitives (e.g., `espv_task_create`) that abstract the `xTaskCreatePinnedToCore` call.
- Resource sharing between cores (e.g., Frame Buffers) must be managed strictly using `Semaphore` or `Mutex` mechanisms, provided by the `Backend/HAL`.

## Consequences
- **Positive:** Predictable performance, reduced frame drops, stable control loops.
- **Negative:** Increased complexity in inter-core synchronization; developers must be aware of race conditions when sharing data between Core 0 and Core 1.

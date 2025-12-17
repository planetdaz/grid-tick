# Kinetic-Digital 24 Clock Array

**Version:** 1.0 (Dec 2025)

## Objective

A modular 8×3 matrix of 24 round 1.28" LCDs animating three synchronized "hands" per screen to create kinetic art and large-scale digital digits.

---

## 1. System Architecture

To ensure smooth, high-frame-rate animation (30+ FPS) across all 24 screens, the system utilizes a **Distributed Master-Slave Architecture**.

| Component | Specification |
|-----------|---------------|
| **Primary Master** | 1× ESP32-S3 – Handles NTP time synchronization, global animation logic, and coordinates cluster units |
| **Sub-Masters** | 6× ESP32-S3 units – Each controls a 1×4 modular cluster (4 screens) |
| **Inter-Link** | ESP-NOW (Low-latency 2.4GHz) for wireless synchronization or RS485 for wired backbone |

---

## 2. Hardware Specification

### 2.1 Display Modules

| Property | Specification |
|----------|---------------|
| **Type** | 1.28" Round LCD (IPS) |
| **Controller** | GC9A01 (SPI) |
| **Resolution** | 240×240 pixels |
| **Color Depth** | 8-bit (RGB332) or 16-bit with DMA |

### 2.2 Microcontrollers (Sub-Masters)

| Property | Specification |
|----------|---------------|
| **Unit** | ESP32-S3-WROOM-1 (with 8MB PSRAM) |
| **Key Advantage** | Native hardware support for high-speed SPI (up to 80MHz) and DMA, allowing CPU to process animations while data is pushed to screens |

### 2.3 Power Requirements

| Specification | Value |
|---------------|-------|
| **Peak Current per Screen** | ~150mA (Full White) |
| **Peak Current per ESP32** | ~100mA |
| **Total Load** | ~5.0A @ 5V DC |
| **Power Supply** | 5V 10A DC with distributed 470µF capacitors at each 1×4 cluster |

---

## 3. Wiring Logic (Per 4-Screen Cluster)

Each ESP32-S3 drives 4 screens without multiplexing by utilizing abundant GPIOs and shared SPI lines.

| SPI Signal | GPIO (Shared) | GPIO (Individual) |
|-----------|---------------|-------------------|
| **SCLK** | Pin 12 | — |
| **MOSI** | Pin 11 | — |
| **DC** | Pin 10 | — |
| **RST** | Pin 9 | — |
| **CS 1–4** | — | Pins 1, 2, 4, 5 |

> **Note:** MISO (Input) is omitted as the screens are write-only.

---

## 4. Software Strategy

### 4.1 Graphics Engine

- **Library:** TFT_eSPI or LVGL
- **Rendering:** Use Sprites for the three clock hands. Instead of redrawing 57,600 pixels per screen, only the bounding box of the moving hands is updated
- **Anti-Aliasing:** Hands are rendered as 8-bit alpha-blended sprites to ensure smooth edges during rotation

### 4.2 Sync Protocol

The Primary Master broadcasts a "Sync Frame" every 16ms (60Hz):

```json
{
  "unit": "ALL",
  "time_ms": 1734444000123,
  "state": "TRANSITION",
  "target_angles": [ [H1, M1, S1], [H2, M2, S2], ... ]
}
```

---

## 5. Mechanical Design (3D Printing)

- **Module Size:** 1×4 horizontal strips
- **Mounting:** Screens are friction-fitted into circular recesses with a 2mm bezel
- **Cable Management:** Internal channels for the 5V power bus and a 4-pin interconnect for the "Main Master" (if using wired serial)

---

## 6. Development Milestones

- **Phase 1:** Single ESP32-S3 driving 4 screens via DMA with smooth hand rotation
- **Phase 2:** ESP-NOW communication testing between two sub-masters
- **Phase 3:** 3D print 1×4 housing and stress-test thermal/power delivery
- **Phase 4:** Final assembly of 8×3 matrix and implementation of "ClockClock" digit transitions
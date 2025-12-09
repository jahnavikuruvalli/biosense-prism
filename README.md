# 🫀 BioSense Prism - Project Roadmap

> **An open-source biosensor data visualization and analysis platform**
>
> Built with: React, Category Theory, MQTT, Go, and love for learning 💚

---

## 🎯 Project Vision

Build a comprehensive platform that:

1. **Ingests** data from various biosensors (ECG, PPG, SpO2, Temperature, EEG, EMG, etc.)
2. **Processes** signals in real-time using composable CT-based operations
3. **Visualizes** waveforms and derived metrics with professional UI
4. **Analyzes** data with an integrated LLM agent
5. **Learns** - this is primarily an educational project!

---

## 📋 Table of Contents

- [Phase 1: Foundation](#phase-1-foundation-current)
- [Phase 2: Sensor Integration](#phase-2-sensor-integration)
- [Phase 3: Controller Integration](#phase-3-controller-integration)
- [Phase 4: Advanced Signal Processing](#phase-4-advanced-signal-processing)
- [Phase 5: Data Persistence](#phase-5-data-persistence)
- [Phase 6: LLM Integration](#phase-6-llm-integration)
- [Phase 7: Alerts & Notifications](#phase-7-alerts--notifications)
- [Phase 8: Mobile & Wearables](#phase-8-mobile--wearables)
- [Phase 9: Research Features](#phase-9-research-features)
- [Architecture Overview](#architecture-overview)
- [Learning Resources](#learning-resources)
- [Contributing](#contributing)

---

## Phase 1: Foundation ✅ (Current)

**Status:** Complete  
**Difficulty:** ⭐⭐  
**Skills:** React, JavaScript, CSS, Basic DSP

### Deliverables

- [x] React-based UI with professional styling
- [x] Mock MQTT sensor simulation
- [x] Real-time waveform visualization (ECG, PPG)
- [x] Vitals display (HR, SpO2, Temp, HRV)
- [x] CT-based signal processing pipeline
- [x] Basic LLM agent chat interface (mock)
- [x] Alert system

### Signal Processing Operations Implemented

```
FUNCTOR (map)          COMONAD (extend)       FILTERABLE
─────────────          ─────────────────      ──────────
• scale(k)             • sma(window)          • threshold(min, max)
• offset(d)            • ema(alpha)           • removeOutliers(k)
• normalize            • lowpass(cutoff)      • downsample(factor)
• abs                  • bandpass(low, high)
• square               
• diff (derivative)    FOLDABLE               DETECTION
                       ────────               ─────────
                       • stats()              • detectPeaks(threshold)
                       • calculateHR()        • calculateHRV()
```

### Try It

1. Open `biosense-prism.html` in browser
2. Click **▶ Start** to begin mock data stream
3. Add signal processing operations from left panel
4. Chat with the AI agent about your vitals

---

## Phase 2: Sensor Integration

**Status:** Not Started  
**Difficulty:** ⭐⭐⭐  
**Skills:** Hardware, Electronics, Sensor protocols

### 2.1 Heart & Pulse Sensors

| Sensor                      | Interface | Approx Cost | Difficulty |
|-----------------------------|-----------|-------------|------------|
| **MAX30102**                | I2C       | $3-5        | ⭐⭐         |
| SpO2 + PPG (pulse oximeter) |           |             |            |
| **AD8232**                  | Analog    | $10-15      | ⭐⭐⭐        |
| Single-lead ECG             |           |             |            |
| **Pulse Sensor**            | Analog    | $5          | ⭐          |
| Simple PPG for beginners    |           |             |            |
| **MAX86150**                | I2C       | $15-20      | ⭐⭐⭐        |
| ECG + PPG combo chip        |           |             |            |

#### Task: MAX30102 Integration

```
📁 sensors/max30102/
├── README.md           # Wiring guide, datasheet link
├── arduino/
│   └── max30102.ino    # Arduino sketch
├── esp32/
│   └── max30102.cpp    # ESP-IDF or Arduino
├── raspberry-pi/
│   └── max30102.py     # Python with smbus
└── docs/
    └── wiring.png      # Connection diagram
```

**Learning Goals:**

- I2C communication protocol
- Sensor initialization and configuration
- Reading raw sensor data
- Converting raw values to meaningful units

### 2.2 Temperature Sensors

| Sensor       | Interface | Use Case                  |
|--------------|-----------|---------------------------|
| **MLX90614** | I2C       | Non-contact IR (forehead) |
| **DS18B20**  | 1-Wire    | Contact (skin probe)      |
| **TMP117**   | I2C       | High precision (±0.1°C)   |

### 2.3 Movement & Position

| Sensor       | Interface | Use Case                  |
|--------------|-----------|---------------------------|
| **MPU6050**  | I2C       | Accelerometer + Gyroscope |
| **BNO055**   | I2C       | 9-DOF IMU (orientation)   |
| **MAX30205** | I2C       | Body temp + movement      |

### 2.4 Advanced Biosensors (Future)

| Sensor               | Measures                  | Difficulty |
|----------------------|---------------------------|------------|
| **EEG (OpenBCI)**    | Brainwaves                | ⭐⭐⭐⭐⭐      |
| **EMG**              | Muscle activity           | ⭐⭐⭐⭐       |
| **GSR/EDA**          | Skin conductance (stress) | ⭐⭐         |
| **Respiration Belt** | Breathing rate            | ⭐⭐         |

### Deliverables

- [ ] Sensor library for each supported sensor
- [ ] Wiring diagrams and documentation
- [ ] Calibration procedures
- [ ] Example sketches for each controller platform

---

## Phase 3: Controller Integration

**Status:** Not Started  
**Difficulty:** ⭐⭐⭐  
**Skills:** Embedded systems, MQTT, Networking

### 3.1 ESP32 (Recommended for Beginners)

**Why ESP32?**

- Built-in WiFi + Bluetooth
- Low cost (~$5-10)
- Arduino compatible
- Enough power for signal processing

```cpp
// Example: ESP32 → MQTT → BioSense Prism

#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
#include "MAX30105.h"

MAX30105 sensor;
WiFiClient wifi;
PubSubClient mqtt(wifi);

void setup() {
  WiFi.begin(SSID, PASSWORD);
  mqtt.setServer(MQTT_BROKER, 1883);
  sensor.begin();
}

void loop() {
  // Read sensor
  uint32_t ir = sensor.getIR();
  uint32_t red = sensor.getRed();
  
  // Calculate SpO2 and HR
  float spo2 = calculateSpO2(ir, red);
  float hr = calculateHR(ir);
  
  // Send to MQTT
  StaticJsonDocument<256> doc;
  doc["deviceId"] = "esp32-001";
  doc["timestamp"] = millis();
  doc["sensors"]["ppg"]["value"] = ir;
  doc["sensors"]["spo2"]["value"] = spo2;
  doc["sensors"]["hr"]["value"] = hr;
  
  char buffer[256];
  serializeJson(doc, buffer);
  mqtt.publish("biosense/device/esp32-001", buffer);
  
  delay(4); // ~250 Hz
}
```

#### Task: ESP32 Sensor Hub

```
📁 controllers/esp32/
├── README.md
├── platformio.ini
├── src/
│   ├── main.cpp
│   ├── sensors/
│   │   ├── max30102.h
│   │   ├── ad8232.h
│   │   └── mpu6050.h
│   ├── network/
│   │   ├── mqtt.h
│   │   └── wifi_manager.h
│   └── config.h
└── docs/
    ├── setup.md
    └── pinout.png
```

### 3.2 Raspberry Pi

**Why Raspberry Pi?**

- Full Linux OS
- Run Python scripts
- Can run MQTT broker locally
- USB device support
- Camera for future video vitals

```python
# Example: Raspberry Pi sensor hub

import paho.mqtt.client as mqtt
import json
import time
from max30102 import MAX30102

sensor = MAX30102()
client = mqtt.Client()
client.connect("localhost", 1883)

while True:
    red, ir = sensor.read_fifo()
    
    message = {
        "deviceId": "rpi-001",
        "timestamp": int(time.time() * 1000),
        "sensors": {
            "ppg": {"value": ir, "sampleRate": 100},
            "spo2": {"value": calculate_spo2(red, ir)}
        }
    }
    
    client.publish("biosense/device/rpi-001", json.dumps(message))
    time.sleep(0.01)  # 100 Hz
```

#### Task: Raspberry Pi Gateway

```
📁 controllers/raspberry-pi/
├── README.md
├── requirements.txt
├── src/
│   ├── main.py
│   ├── sensors/
│   │   ├── __init__.py
│   │   ├── max30102.py
│   │   └── mlx90614.py
│   ├── mqtt_client.py
│   └── config.py
├── services/
│   └── biosense.service    # systemd service file
└── setup.sh
```

### 3.3 Arduino (Educational)

**Why Arduino?**

- Simplest to start
- Great documentation
- Huge community
- No WiFi (needs serial bridge)

```
Arduino → USB Serial → Python Bridge → MQTT → BioSense Prism
```

### 3.4 MQTT Message Format (Standard)

```json
{
  "deviceId": "esp32-living-room",
  "timestamp": 1702055432123,
  "sensors": {
    "ecg": {
      "value": 0.42,
      "unit": "mV",
      "sampleRate": 250
    },
    "ppg": {
      "value": 45230,
      "unit": "raw",
      "sampleRate": 100
    },
    "spo2": {
      "value": 98.2,
      "unit": "%"
    },
    "temp": {
      "value": 36.6,
      "unit": "C"
    },
    "accel": {
      "x": 0.02,
      "y": -0.98,
      "z": 0.01,
      "unit": "g"
    }
  },
  "battery": 85,
  "rssi": -45
}
```

### Deliverables

- [ ] ESP32 sensor hub firmware
- [ ] Raspberry Pi gateway software
- [ ] Arduino + Python bridge
- [ ] MQTT topic structure documentation
- [ ] Device provisioning guide

---

## Phase 4: Advanced Signal Processing

**Status:** Not Started  
**Difficulty:** ⭐⭐⭐⭐  
**Skills:** DSP, Math, Go/Rust, WASM (optional)

### 4.1 ECG Analysis

| Feature                  | Description                | Difficulty |
|--------------------------|----------------------------|------------|
| **R-peak detection**     | Pan-Tompkins algorithm     | ⭐⭐⭐        |
| **QRS complex**          | Detect Q, R, S waves       | ⭐⭐⭐⭐       |
| **P & T waves**          | Full waveform segmentation | ⭐⭐⭐⭐       |
| **Arrhythmia detection** | Basic irregular rhythm     | ⭐⭐⭐⭐⭐      |
| **12-lead ECG**          | Multi-lead analysis        | ⭐⭐⭐⭐⭐      |

```go
// Example: Pan-Tompkins R-peak detection in Go

func PanTompkins(ecg []float64, sampleRate float64) []int {
// 1. Bandpass filter (5-15 Hz)
filtered := Bandpass(ecg, 5, 15, sampleRate)

// 2. Derivative
diff := Differentiate(filtered)

// 3. Square
squared := Square(diff)

// 4. Moving window integration
integrated := MovingAverage(squared, int(0.15 * sampleRate))

// 5. Adaptive thresholding
peaks := AdaptiveThreshold(integrated)

return peaks
}
```

### 4.2 HRV Analysis

| Metric            | Description                     | Domain    |
|-------------------|---------------------------------|-----------|
| **SDNN**          | Std dev of NN intervals         | Time      |
| **RMSSD**         | Root mean square of differences | Time      |
| **pNN50**         | % intervals > 50ms different    | Time      |
| **LF/HF ratio**   | Low freq / High freq power      | Frequency |
| **Poincaré plot** | SD1, SD2 scatter analysis       | Nonlinear |

### 4.3 SpO2 Calculation

```javascript
// Beer-Lambert Law for SpO2

function calculateSpO2(redAC, redDC, irAC, irDC) {
    // R ratio
    const R = (redAC / redDC) / (irAC / irDC);

    // Empirical formula (calibration required!)
    const SpO2 = 110 - 25 * R;

    return Math.max(0, Math.min(100, SpO2));
}
```

### 4.4 FFT & Frequency Analysis

| Application      | Frequency Range |
|------------------|-----------------|
| Heart rate       | 0.5 - 4 Hz      |
| Respiratory rate | 0.1 - 0.5 Hz    |
| HRV LF band      | 0.04 - 0.15 Hz  |
| HRV HF band      | 0.15 - 0.4 Hz   |
| EEG Alpha        | 8 - 13 Hz       |
| EEG Beta         | 13 - 30 Hz      |

### 4.5 Go WASM Module (Optional)

For heavy computation, we can compile Go to WASM:

```go
// wasm/signal/fft.go

//go:build js && wasm

package main

import (
	"syscall/js"
	"github.com/mjibson/go-dsp/fft"
)

func fftWrapper(this js.Value, args []js.Value) interface{} {
	data := jsArrayToFloat64(args[0])
	result := fft.FFTReal(data)
	return complexArrayToJS(result)
}

func main() {
	js.Global().Set("bioSignal", js.ValueOf(map[string]interface{}{
		"fft":         js.FuncOf(fftWrapper),
		"panTompkins": js.FuncOf(panTompkinsWrapper),
		"hrv":         js.FuncOf(hrvWrapper),
	}))
	select {}
}
```

Build:

```bash
GOOS=js GOARCH=wasm go build -o signal.wasm ./wasm
```

### Deliverables

- [ ] Pan-Tompkins R-peak detector
- [ ] HRV time-domain metrics
- [ ] HRV frequency-domain analysis (FFT)
- [ ] SpO2 calculation with calibration
- [ ] Respiratory rate from PPG
- [ ] Optional: Go WASM signal module

---

## Phase 5: Data Persistence

**Status:** Not Started  
**Difficulty:** ⭐⭐⭐  
**Skills:** Go, Databases, Time-series data

### 5.1 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Sensors → MQTT Broker → Go Backend → TimescaleDB/InfluxDB    │
│                              │                                  │
│                              ├── WebSocket → Browser UI         │
│                              └── REST API → Historical queries  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Database Options

| Database        | Best For                       | Difficulty |
|-----------------|--------------------------------|------------|
| **SQLite**      | Getting started, single user   | ⭐          |
| **TimescaleDB** | Time-series, SQL compatible    | ⭐⭐⭐        |
| **InfluxDB**    | Time-series, high write volume | ⭐⭐⭐        |
| **QuestDB**     | Fastest time-series queries    | ⭐⭐⭐        |

### 5.3 Go Backend Structure

```
📁 backend/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── mqtt/
│   │   └── subscriber.go
│   ├── websocket/
│   │   └── hub.go
│   ├── storage/
│   │   ├── timescale.go
│   │   └── models.go
│   ├── api/
│   │   ├── routes.go
│   │   └── handlers.go
│   └── signal/
│       ├── filter.go
│       └── analysis.go
├── go.mod
└── Makefile
```

### 5.4 API Endpoints

```
GET  /api/devices              # List all devices
GET  /api/devices/:id          # Get device info
GET  /api/devices/:id/data     # Get historical data
     ?sensor=ecg
     &from=2024-01-01T00:00:00Z
     &to=2024-01-02T00:00:00Z
     &resolution=1s

GET  /api/devices/:id/stats    # Get computed statistics
POST /api/devices/:id/export   # Export data (CSV, JSON)

WS   /ws/stream/:deviceId      # Real-time WebSocket stream
```

### Deliverables

- [ ] Go backend with MQTT subscriber
- [ ] TimescaleDB schema for biosensor data
- [ ] WebSocket server for real-time streaming
- [ ] REST API for historical queries
- [ ] Data export functionality

---

## Phase 6: LLM Integration

**Status:** Mock Only  
**Difficulty:** ⭐⭐⭐⭐  
**Skills:** LLM APIs, Prompt engineering, RAG

### 6.1 Integration Options

| Provider             | Pros             | Cons                  |
|----------------------|------------------|-----------------------|
| **OpenAI API**       | Best quality     | Cost, privacy         |
| **Anthropic Claude** | Great reasoning  | Cost                  |
| **Local LLaMA**      | Private, free    | Needs GPU             |
| **Ollama**           | Easy local setup | Needs decent hardware |

### 6.2 Agent Capabilities

```
┌─────────────────────────────────────────────────────────────────┐
│  LLM Agent can:                                                 │
│                                                                 │
│  📊 Analyze current vitals                                      │
│     "Is my heart rate normal for my age?"                       │
│                                                                 │
│  📈 Interpret trends                                            │
│     "How has my HRV changed this week?"                         │
│                                                                 │
│  🔍 Explain waveforms                                           │
│     "What does this ECG pattern mean?"                          │
│                                                                 │
│  ⚠️ Provide health context                                      │
│     "Why might my SpO2 be lower today?"                         │
│                                                                 │
│  📚 Educational responses                                       │
│     "Explain how PPG measures heart rate"                       │
│                                                                 │
│  ⚠️ DISCLAIMER: Not medical advice!                             │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 Prompt Engineering

```javascript
const systemPrompt = `You are a helpful health data assistant for the BioSense Prism application.
You have access to real-time biosensor data including:
- ECG waveforms and heart rate
- PPG (photoplethysmography) signals  
- SpO2 (blood oxygen saturation)
- Body temperature
- HRV (heart rate variability)

Your role is to:
1. Help users understand their biosensor readings
2. Explain what different metrics mean
3. Identify when readings are outside normal ranges
4. Provide educational information about cardiovascular health

IMPORTANT DISCLAIMERS:
- You are NOT a medical professional
- Always recommend consulting a doctor for health concerns
- Never diagnose conditions
- Encourage users to seek professional medical advice

Current user data:
${JSON.stringify(currentVitals)}

Recent statistics:
${JSON.stringify(recentStats)}
`;
```

### 6.4 Context Injection

```javascript
// Inject real-time data into LLM context

function buildContext(vitals, ecgBuffer, pipeline) {
    const ecgStats = Signal.stats(ecgBuffer);
    const peaks = Signal.detectPeaks(Signal.normalize(ecgBuffer), 0.6);

    return {
        current: {
            heartRate: vitals.hr,
            spo2: vitals.spo2,
            temperature: vitals.temp,
            hrv: vitals.hrv
        },
        ecgAnalysis: {
            mean: ecgStats.mean,
            stdDev: ecgStats.std,
            peakCount: peaks.length,
            rhythm: analyzeRhythm(peaks)
        },
        appliedFilters: pipeline.map(p => p.name),
        timestamp: new Date().toISOString()
    };
}
```

### Deliverables

- [ ] LLM API integration (OpenAI/Anthropic/Local)
- [ ] Context-aware prompts with vital data
- [ ] Chat history persistence
- [ ] Health disclaimer system
- [ ] Rate limiting and error handling

---

## Phase 7: Alerts & Notifications

**Status:** Basic Only  
**Difficulty:** ⭐⭐⭐  
**Skills:** Backend, Push notifications, Email

### 7.1 Alert Types

| Alert               | Trigger             | Severity    |
|---------------------|---------------------|-------------|
| Low SpO2            | < 94%               | 🔴 Critical |
| High HR             | > 120 BPM sustained | 🟡 Warning  |
| Low HR              | < 50 BPM            | 🟡 Warning  |
| Fever               | > 38°C              | 🟡 Warning  |
| Irregular rhythm    | Arrhythmia detected | 🔴 Critical |
| Sensor disconnected | No data 30s         | 🔵 Info     |

### 7.2 Notification Channels

- [ ] In-app notifications (current)
- [ ] Browser push notifications
- [ ] Email alerts
- [ ] SMS (Twilio)
- [ ] Telegram bot
- [ ] Webhook (for integrations)

### Deliverables

- [ ] Configurable alert thresholds
- [ ] Alert escalation rules
- [ ] Push notification service
- [ ] Email integration
- [ ] Alert history and acknowledgment

---

## Phase 8: Mobile & Wearables

**Status:** Future  
**Difficulty:** ⭐⭐⭐⭐⭐  
**Skills:** React Native/Flutter, BLE, Mobile development

### 8.1 Mobile App

```
📁 mobile/
├── App.tsx
├── screens/
│   ├── Dashboard.tsx
│   ├── DeviceConnect.tsx
│   ├── History.tsx
│   └── Settings.tsx
├── components/
│   ├── VitalCard.tsx
│   ├── WaveformChart.tsx
│   └── AlertBanner.tsx
└── services/
    ├── ble.ts
    ├── mqtt.ts
    └── notifications.ts
```

### 8.2 BLE Direct Connection

Skip MQTT and connect phone directly to sensors via Bluetooth:

```
Sensor (ESP32 BLE) ←→ Phone App ←→ Cloud Sync
```

### 8.3 Apple Watch / WearOS Integration

- Read HealthKit data (iOS)
- Read Health Connect data (Android)
- Display BioSense alerts on watch

### Deliverables

- [ ] React Native or Flutter app
- [ ] BLE sensor connection
- [ ] Offline data collection
- [ ] Cloud sync
- [ ] Watch complications/tiles

---

## Phase 9: Research Features

**Status:** Future  
**Difficulty:** ⭐⭐⭐⭐⭐  
**Skills:** ML, Statistics, Data science

### 9.1 Machine Learning

| Feature                       | Application                       |
|-------------------------------|-----------------------------------|
| **Anomaly detection**         | Identify unusual patterns         |
| **Sleep staging**             | Classify sleep phases from HR/HRV |
| **Stress detection**          | Predict stress from HRV + GSR     |
| **Activity classification**   | Detect walking, running, resting  |
| **Arrhythmia classification** | Detect AFib, PVCs                 |

### 9.2 Research Tools

- [ ] Data annotation interface
- [ ] Experiment session recorder
- [ ] Multi-subject comparison
- [ ] Statistical analysis dashboard
- [ ] Data export for research (EDF, CSV)

### 9.3 Integration with Research Platforms

- [ ] OpenBCI integration
- [ ] BIOPAC compatibility
- [ ] HL7 FHIR export
- [ ] Apple HealthKit sync
- [ ] Google Fit sync

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           BioSense Prism Architecture                        │
│                                                                             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────────────────────┐   │
│  │   Sensors   │     │ Controllers │     │         MQTT Broker         │   │
│  │             │     │             │     │                             │   │
│  │ • MAX30102  │────▶│ • ESP32     │────▶│  • Mosquitto               │   │
│  │ • AD8232    │     │ • RPi       │     │  • HiveMQ                  │   │
│  │ • MLX90614  │     │ • Arduino   │     │  • EMQX                    │   │
│  │ • MPU6050   │     │             │     │                             │   │
│  └─────────────┘     └─────────────┘     └──────────────┬──────────────┘   │
│                                                          │                  │
│                                          ┌───────────────┼───────────────┐  │
│                                          │               │               │  │
│                                          ▼               ▼               │  │
│                               ┌─────────────────┐ ┌─────────────────┐    │  │
│                               │   Go Backend    │ │  Browser UI     │    │  │
│                               │                 │ │  (Direct MQTT)  │    │  │
│                               │ • MQTT Sub      │ │                 │    │  │
│                               │ • Processing    │ │ • React         │    │  │
│                               │ • Storage       │ │ • CT Pipeline   │    │  │
│                               │ • REST API      │ │ • Charts        │    │  │
│                               │ • WebSocket     │ │ • LLM Agent     │    │  │
│                               └────────┬────────┘ └─────────────────┘    │  │
│                                        │                                 │  │
│                                        ▼                                 │  │
│                               ┌─────────────────┐                        │  │
│                               │   TimescaleDB   │                        │  │
│                               │                 │                        │  │
│                               │ • Time-series   │                        │  │
│                               │ • Historical    │                        │  │
│                               │ • Analytics     │                        │  │
│                               └─────────────────┘                        │  │
│                                                                          │  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Learning Resources

### Signal Processing

- 📚 [Think DSP](https://greenteapress.com/wp/think-dsp/) - Free book
- 📺 [3Blue1Brown - Fourier Transform](https://www.youtube.com/watch?v=spUNpyF58BY)
- 📖 [ECG Basics](https://litfl.com/ecg-library/)

### Hardware

- 📺 [Andreas Spiess YouTube](https://www.youtube.com/c/AndreasSpiess) - ESP32 tutorials
- 📚 [ESP32 Arduino Core](https://docs.espressif.com/projects/arduino-esp32/)
- 📖 [MAX30102 Datasheet](https://www.maximintegrated.com/en/products/sensors/MAX30102.html)

### MQTT

- 📚 [MQTT Essentials](https://www.hivemq.com/mqtt-essentials/)
- 🔧 [Mosquitto Broker](https://mosquitto.org/)
- 📖 [MQTT.js Documentation](https://github.com/mqttjs/MQTT.js)

### Go Backend

- 📚 [Go by Example](https://gobyexample.com/)
- 📖 [Paho MQTT Go](https://github.com/eclipse/paho.mqtt.golang)
- 🔧 [Gin Web Framework](https://gin-gonic.com/)

### Category Theory (for the curious!)

- 📺 [Category Theory for Programmers](https://www.youtube.com/playlist?list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_)
- 📚 [Mostly Adequate Guide to FP](https://mostly-adequate.gitbook.io/mostly-adequate-guide/)

---

## Contributing

### Getting Started

1. **Fork the repository**
2. **Pick a task** from any phase
3. **Create a branch** `feature/phase-X-task-name`
4. **Submit a PR** with:
    - Clear description
    - Screenshots/videos if UI changes
    - Tests if applicable

### Code Style

- **JavaScript/React**: Use functional components, hooks
- **Go**: Follow standard Go conventions
- **Python**: PEP 8, type hints
- **Commits**: Conventional commits (`feat:`, `fix:`, `docs:`)

### Communication

- 💬 Create GitHub Issues for questions
- 📝 Document everything you learn
- 🤝 Help other interns!

---

## Quick Start for Interns

### Week 1: Setup & Exploration

1. Run `biosense-prism.html` and understand the UI
2. Read the CT signal processing code
3. Set up ESP32 development environment
4. Order sensors (MAX30102 is a great start!)

### Week 2-3: First Sensor

1. Wire MAX30102 to ESP32
2. Read raw sensor data
3. Display in Serial Monitor
4. Send to MQTT broker

### Week 4+: Integration

1. Connect ESP32 to BioSense Prism
2. Replace mock data with real sensors
3. Calibrate and validate readings
4. Document your setup!

---

## License

MIT License - Build cool stuff, learn lots, share with others! 💚

---

**Remember:** The goal is to **learn**. Don't worry about perfection. Ask questions. Break things. Have fun! 🚀

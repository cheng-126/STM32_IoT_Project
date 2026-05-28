# STM32 IoT Project

> STM32F103C8T6 + ESP8266 + OneNet MQTT + WeChat Mini Program  
> Temperature & Humidity Sensing · OLED Real-time Display · Remote LED Control

---

## Features

- **Sensing**: DHT11 sensor collects temperature and humidity data
- **Display**: 0.96" OLED shows real-time temperature, humidity, and LED status
- **Cloud**: ESP8266 reports data to OneNet platform via MQTT protocol
- **Remote Control**: WeChat Mini Program monitors data in real time and remotely controls the onboard LED

---

## Hardware

| Component | Description / Wiring |
|-----------|----------------------|
| STM32F103C8T6 | Main MCU (Blue Pill), 72MHz |
| DHT11 | Temperature & humidity sensor, OUT → PA0 |
| ESP8266-01S | WiFi module, RX → PA2, TX → PA3 |
| 0.96" OLED | I²C display, SCL → PB1, SDA → PB0 |
| Onboard LED | PC13, active low |
| USB-TTL / STLink | Flashing & debugging |

---

## System Architecture

```
[Uplink]   DHT11 → STM32 → OLED → ESP8266 → OneNet → WeChat Mini Program
[Downlink] WeChat Mini Program → OneNet REST API → MQTT → ESP8266 → STM32 → LED
```

---

## Software Toolchain

| Tool | Purpose |
|------|---------|
| STM32CubeMX | GUI peripheral configuration, HAL code generation |
| VSCode + EIDE | Code editing, ARM GCC compilation, flashing |
| WeChat DevTools | Mini Program development, preview, upload |
| OneNet Console | Device management, data model config, API testing |

---

## Project Structure

```
.
├── Core/                   # Application code (main.c, DHT11, OLED, ESP8266 drivers)
├── Drivers/                # STM32 HAL library
├── u8g2/                   # OLED graphics library
├── cmake/                  # CMake toolchain configuration
├── CMakeLists.txt          # Build entry point
├── CMakePresets.json       # Build presets
├── STM32F103XX_FLASH.ld    # Linker script
├── startup_stm32f103xb.s  # Startup file
├── 1_led.ioc               # STM32CubeMX project file
├── .clang-format           # Code formatting rules
└── README.md
```

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

### 2. Configure credentials

Fill in your personal information in `esp8266.h`:

```c
#define WIFI_SSID         "Your_WiFi_SSID"
#define WIFI_PASSWORD     "Your_WiFi_Password"
#define ONENET_HOST       "mqtts.heclouds.com"
#define ONENET_PORT       1883
#define ONENET_PRODUCT_ID "Your_Product_ID"
#define ONENET_DEVICE_ID  "Your_Device_Name"
#define ONENET_TOKEN      "Your_MQTT_Token"
```

### 3. Build & Flash

Open the project in VSCode with the EIDE plugin, select the appropriate build preset, then compile and flash to the board.

---

## OneNet Platform Setup

1. Log in to [OneNet Console](https://open.iot.10086.cn) and create a new product (protocol: MQTT)
2. Add the following properties to the data model:

| Identifier | Type | Permission |
|------------|------|------------|
| `temperature` | int32 | Read-only |
| `humidity` | int32 | Read-only |
| `led` | bool | Read/Write |

3. Create a device, generate an MQTT access Token using the Token tool, and fill it into `esp8266.h`

---

## WeChat Mini Program

The Mini Program communicates with the device via OneNet REST API:

- **Fetch data**: `GET https://iot-api.heclouds.com/thingmodel/query-device-property`
- **Control LED**: `POST https://iot-api.heclouds.com/thingmodel/set-device-property`

Before releasing, add the following to the server domain whitelist at [WeChat MP Console](https://mp.weixin.qq.com) → Development → Server Domain:

```
https://iot-api.heclouds.com
```

---

## Key Design Decisions

| Design Point | Approach | Reason |
|--------------|----------|--------|
| MQTT connection | Manual TCP + raw MQTT packets | Compatible with all ESP8266 firmware versions |
| UART receive | Interrupt + static buffer | Non-blocking, doesn't stall the main loop |
| DHT11 timing protection | Disable global interrupts | Prevents ISR from disrupting µs-level timing |
| Main loop architecture | Super loop + non-blocking timer | Simple and reliable, near-zero LED response latency |
| Publish QoS | QoS 0 | Occasional data loss is acceptable for sensor readings |
| Subscribe QoS | QoS 1 | LED control commands must not be lost |
| OLED driver | Software I²C (PB0/PB1) | Avoids known hardware I²C bug on STM32F103 |
| Mini Program UX | Optimistic update | Instant UI response without waiting for 3s timeout |

---

## Troubleshooting

| Symptom | What to Check |
|---------|---------------|
| OLED blank | Verify I²C address (0x78), confirm PB0/PB1 initialized as GPIO_Output |
| DHT11 returns 1 | Check PA0 wiring, 3.3V power, internal pull-up config |
| ESP8266 no response | Confirm TX/RX are crossed (MCU PA2 → ESP RX, PA3 → ESP TX) |
| WiFi connection fails | Check SSID and password, avoid Chinese SSID |
| MQTT connection fails | Check Token expiry (et timestamp), verify Host/Port |
| Mini Program auth error | REST API Token version must be `2022-05-01` |
| No data on mobile | Check server domain config; re-upload Mini Program after any changes |

---

## Future Extensions

- **Hardware**: Add light, CO₂, barometric sensors; relay control; buzzer alerts
- **Software**: Auto reconnect on disconnect; MQTT keepalive heartbeat; historical data charts; OTA firmware update
- **Architecture**: FreeRTOS multi-tasking; DMA + idle interrupt to replace RXNE interrupt; WebSocket real-time push

---

## License

MIT

# ESP32 Tank Level Monitoring with XKC-Y25-V & Home Assistant

[![ESPHome Compatible](https://img.shields.io/badge/ESPHome-Compatible-brightgreen)](https://esphome.io/)
[![Home Assistant Integration](https://img.shields.io/badge/Home%20Assistant-Integration-blue)](https://www.home-assistant.io/)
[![DIY Project](https://img.shields.io/badge/DIY-Friendly-orange)](.)

**This project provides a robust, DIY solution for monitoring liquid levels in a tank using non-contact sensors and seamless integration with Home Assistant.**

![Final Installation](images/result.jpg)
_The finished system mounted on a tank, with sensors attached.

## Overview

This system continuously monitors liquid presence at three key points in a tank (e.g., low, high, and an overall check point) using XKC-Y25-V non-contact capacitive sensors. An ESP32 (specifically, a Seeed Studio XIAO ESP32C3) running ESPHome reads the sensor states and reports them wirelessly to Home Assistant for real-time monitoring, logging, and automation triggers. The entire setup is housed in a waterproof enclosure for durability.

## Key Features

*   **Real-time Monitoring:** Instantly know the state of three crucial tank levels via Home Assistant.
*   **Non-Contact Sensing:** Utilizes XKC-Y25-V sensors that detect liquid through non-metallic tank walls, avoiding direct contact with the contents – ideal for various liquids, including potentially corrosive ones.
*   **Triple-Level Detection:** Configure sensors for minimum, maximum, and intermediate levels, or any three points relevant to your needs.
*   **Reliable Integration:** Uses ESPHome for easy configuration and robust WiFi communication with Home Assistant via its native API.
*   **DIY Friendly:** Built with readily available components.
*   **Durable & Safe:** Housed in an IP67 enclosure with waterproof connectors (IP68) and includes necessary logic level shifting for safe operation between the 5V sensors and the 3.3V ESP32.
*   **AC Powered:** Uses a compact AC-DC converter for a permanent installation.

## How It Works Visually

```mermaid
graph LR
    subgraph Enclosure (IP67)
        AC[Mains AC Power] --> HLK[HLK-5M05 Power Supply (5V DC)]
        HLK --> ESP32[XIAO ESP32C3]
        HLK --> LLS[Logic Level Shifter HV (5V)]
        ESP32 -- 3.3V --> LLS_LV[LLS LV (3.3V)]
        ESP32 -- GND --> LLS_GND[LLS GND]
        HLK -- GND --> ESP32_GND[ESP32 GND]

        S1_SIG_IN[Sensor 1 Signal (5V)] --> LLS_HV1[LLS HV1]
        LLS_LV1[LLS LV1] -- 3.3V Signal --> ESP32_GPIO9[ESP32 GPIO9]

        S2_SIG_IN[Sensor 2 Signal (5V)] --> LLS_HV2[LLS HV2]
        LLS_LV2[LLS LV2] -- 3.3V Signal --> ESP32_GPIO10[ESP32 GPIO10]

        S3_SIG_IN[Sensor 3 Signal (5V)] --> LLS_HV3[LLS HV3]
        LLS_LV3[LLS LV3] -- 3.3V Signal --> ESP32_GPIO8[ESP32 GPIO8]

        ESP32 -- WiFi --> Router[WiFi Router]
    end

    subgraph Outside Enclosure
        Sensor1[XKC-Y25-V Sensor 1] -- Brown(+5V), Blue(GND), Yellow(Signal) --> S1_CONN[SP13 Connector 1]
        Sensor2[XKC-Y25-V Sensor 2] -- Brown(+5V), Blue(GND), Yellow(Signal) --> S2_CONN[SP13 Connector 2]
        Sensor3[XKC-Y25-V Sensor 3] -- Brown(+5V), Blue(GND), Yellow(Signal) --> S3_CONN[SP13 Connector 3]

        S1_CONN -- 5V --> HLK
        S1_CONN -- GND --> HLK
        S1_CONN -- Signal (5V) --> S1_SIG_IN

        S2_CONN -- 5V --> HLK
        S2_CONN -- GND --> HLK
        S2_CONN -- Signal (5V) --> S2_SIG_IN

        S3_CONN -- 5V --> HLK
        S3_CONN -- GND --> HLK
        S3_CONN -- Signal (5V) --> S3_SIG_IN

        AC_Source[AC Power Source] --> AC_CONN[SP16 Power Connector]
        AC_CONN --> AC
    end

    Router -- Network --> HA[Home Assistant]

    style Enclosure fill:#f9f,stroke:#333,stroke-width:2px
```
```

## Components & Hardware Used

### XKC-Y25-V Sensor

- **Type**: Non-contact capacitive liquid level sensor.
- **Function**: It detects the liquid level by measuring changes in capacitance through a non-metallic container wall. This sensor is robust against corrosive liquids and scale.
- **Electrical Characteristics**: Operates on a supply voltage of 5V – 24V, with a digital high/low output. For this project, the sensor is supplied with 5V.
- **Terminal Colors**:
  - Brown Wire: Power Supply Positive (+5V)
  - Yellow Wire: Signal Output (5V digital signal when liquid is detected according to the chosen output mode)
  - Blue Wire: Power Supply Negative (GND)
  - Black Wire: Common terminal (not used for the positive output configuration)
- **Manual**: [XKC-Y25-V-manual.pdf](./XKC-Y25-V-manual.pdf)

### Seeed Studio XIAO ESP32C3

- **Model**: Compact ESP32-C3 based development board.
- **Role**: Serves as the central controller that reads sensor states, processes data, and communicates with Home Assistant over WiFi via the ESPHome API.
- **Operating Voltage**: 3.3V logic on GPIO pins.
- **GPIO Pins Used**: GPIO9, GPIO10, GPIO8 (as defined in `esphome.yaml`).

### 4-Channel IIC I2C Logic Level Shifter Bi-Directional Module

- **Type**: Bi-directional logic level converter module.
- **Purpose**: Safely converts the sensor's 5V digital output (from the yellow wire) down to 3.3V, which is compatible with the ESP32's GPIO input pins. It also allows for bi-directional communication if needed, although only high-to-low conversion is used in this project.
- **Channels**: Provides 4 independent channels for level shifting.
- **Connection**:
  - `HV` pin connects to the 5V power supply.
  - `LV` pin connects to the 3.3V power supply (from the ESP32).
  - `GND` pin connects to the common ground.
  - Each sensor's 5V signal output (Yellow wire) connects to an `HVx` pin (e.g., `HV1`, `HV2`, `HV3`).
  - The corresponding 3.3V signal output is taken from the respective `LVx` pin (e.g., `LV1`, `LV2`, `LV3`) and connected to the ESP32 GPIO pin.

### Power Supply

- **Module**: Hi-Link HLK-5M05 AC-DC Step-Down PCB Mount Power Supply Module (5W, 5V, 1A).
- **Function**: Converts mains AC voltage to the stable 5V DC required by the ESP32 board (via its 5V input pin) and the XKC-Y25-V sensors.
- **Distribution**: The 5V output powers the ESP32 (which internally regulates to 3.3V for its logic) and the `HV` side of the logic level shifter. The ESP32's 3.3V output powers the `LV` side of the shifter.
- All components share a common ground (`GND`).

### Waterproof Connectors

- **Sensor Connectors**: SP13 IP68 Waterproof Connector (4 pins). Used to create detachable connections for the three XKC-Y25-V sensors, allowing for easier installation and maintenance. Each sensor uses one 4-pin connector (Brown, Blue, Yellow wires connected; Black wire pin unused).
- **Power Connector**: SP16 IP68 Waterproof Connector (3 pins). Used for the main AC power input connection to the enclosure, ensuring a watertight seal.

### Enclosure

- **Type**: Plastic IP67 rated enclosure.
- **Purpose**: Houses the ESP32, power supply, logic level shifter, and internal wiring, protecting them from environmental factors like water and dust.

### Waterproof Vent Plug

- **Type**: M12 Nylon Breathable Valve (IP68).
- **Purpose**: Installed in the enclosure to allow pressure equalization while preventing water ingress, crucial for outdoor or varying temperature environments to avoid condensation buildup or seal stress.

## Schematic Diagram (Textual Representation)

Below is a textual schematic showing the interconnections for one sensor. Repeat the Sensor -> Level Shifter -> ESP32 connection for the other two sensors using different shifter channels and GPIO pins.

```text
+5V Supply ───────────────────┬──────────────────────┬──────────→ ESP32 VIN
                              │                      │
                              │ Brown (+5V)          │
                        ┌─────┴────────┐       ┌─────┴─────┐
                        │ XKC-Y25-V    │       │ Level     │
                        │ Sensor       │       │ Shifter   │
                        │              │       │ Module    │
                        │       Yellow ├───────┤ HV (5V)   │
                        │ (Signal Out) │       │ HV1 Input │
                        └─────┬────────┘       ├───────────┤
                              │ Blue (GND)     │ LV1 Output├─→ ESP32 GPIO (e.g., GPIO9)
                              │                │ LV (3.3V) │ ←── ESP32 3.3V Pin
                              │                │ GND       │
Ground ───────────────────────┴────────────────┴───────────┴──→ ESP32 GND Pin
(Common GND)
```

### Connections Summary:

- **Power**:
  - All components share a common GND.
  - XKC-Y25-V Sensors (Brown wire) connect to +5V.
  - Level Shifter HV pin connects to +5V.
  - Level Shifter LV pin connects to ESP32 3.3V output.
  - Level Shifter GND pin connects to common GND.
- **Signals** (Repeated for each sensor):
  - Sensor 1 (Yellow wire) → Level Shifter HV1 → Level Shifter LV1 → ESP32 GPIO9
  - Sensor 2 (Yellow wire) → Level Shifter HV2 → Level Shifter LV2 → ESP32 GPIO10
  - Sensor 3 (Yellow wire) → Level Shifter HV3 → Level Shifter LV3 → ESP32 GPIO8
- **Unused**:
  - XKC-Y25-V Sensor Black wire is unused.
  - Level Shifter Channel 4 (HV4/LV4) is unused.

> Note: This diagram shows the signal flow for one sensor. The Tank High Level and Tank Low Level sensors are connected identically using separate channels on the logic level shifter (e.g., HV2/LV2 and HV3/LV3) and connected to their respective ESP32 GPIO pins (GPIO10 and GPIO8).

## How It Works

### Sensor Operation

XKC-Y25-V Sensor:
The sensor detects the liquid level using capacitive sensing through the tank wall. When liquid is present, its internal circuitry causes the output (yellow wire) to change state (from LOW to HIGH or vice versa based on configuration).

### Logic Level Shifting

- The 5V signal from each sensor's yellow wire is connected to a high-voltage input pin (`HV1`, `HV2`, or `HV3`) on the 4-channel logic level shifter.
- The shifter module, powered by both 5V (`HV` pin) and 3.3V (`LV` pin), converts the incoming 5V HIGH signal to a 3.3V HIGH signal on the corresponding low-voltage output pin (`LV1`, `LV2`, or `LV3`).
- This 3.3V signal is then safely fed into the designated ESP32 GPIO input pin (`GPIO9`, `GPIO10`, or `GPIO8`).

### ESP32 Data Acquisition

- The ESP32, running ESPHome firmware, continuously reads the state of the GPIO pins connected to the level shifter outputs (GPIO9, GPIO10, GPIO8).
- Each sensor's state (HIGH/ON or LOW/OFF) corresponds to the detection of liquid at its specific level (Sensor 1, Sensor 2, Sensor 3).

### Integration with Home Assistant

- Through the API, ESPHome sends sensor state updates to Home Assistant. This enables you to view real-time measurements and configure automation based on sensor data.
- The binary sensors are logged and managed via Home Assistant dashboards.

## ESPHome Configuration

The firmware running on the Seeed Studio XIAO ESP32C3 is managed by ESPHome. The configuration defines the board type, WiFi credentials (via secrets), enables the API for Home Assistant communication, and sets up the binary sensors connected to the GPIO pins.

Key aspects of the configuration (`esphome.yaml`):

- **Board**: `seeed_xiao_esp32c3`
- **Framework**: `esp-idf`
- **Binary Sensors**:
  - `Tank Level Sensor 1`: GPIO9 (Input with Pull-up)
  - `Tank Level Sensor 2`: GPIO10 (Input with Pull-up)
  - `Tank Level Sensor 3`: GPIO8 (Input with Pull-up)
- **Device Class**: Set to `moisture` for appropriate representation in Home Assistant.

You can view the complete configuration file here: [esphome.yaml](./esphome.yaml)

## Project Images

Below are images of the assembled PCB and the final installation:

**Assembled PCB inside the enclosure:**
![Assembled PCB](images/pcb.jpg)

**Final Installation on the Tank:**
![Final Installation](images/result.jpg)

*Shows the enclosure mounted and the three XKC-Y25-V sensor attached to the tank surface.*


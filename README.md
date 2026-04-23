# esphome-yardgate

ESPHome-based sliding gate controller running on a [PRODINo ESP32EX](https://kmpelectronics.eu/shop/prodino-esp32ex/) board with Home Assistant integration.

## Features

- Open, close, and stop the gate from Home Assistant or the built-in web UI
- IR motion curtain sensor pauses/stops the gate when an obstruction is detected
- Magnetic endstop switches for precise open/closed position detection
- Detects external gate activations (wall button / physical remote clicks)
- OTA firmware updates
- Fallback Wi-Fi hotspot when the main network is unreachable
- Optional MQTT support (disabled by default; Home Assistant native API is used instead)
- Optional Ethernet (commented out; requires an Olimex ESP32-POE board instead)

## Hardware

| Component | Details |
|---|---|
| Controller board | [PRODINo ESP32EX](https://kmpelectronics.eu/shop/prodino-esp32ex/) |
| Board schematic | [ProDINoESP32-ETHv02-Schematic.pdf](https://kmpelectronics.eu/wp-content/uploads/2019/12/ProDINoESP32-ETHv02-Schematic.pdf) |
| I/O expander | MCP23S08 (onboard, SPI-connected via GPIO18/19/23, CS on GPIO32) |
| Flashing guide | [PRODINo ESP32 Tutorial](https://kmpelectronics.eu/tutorials-examples/prodino-esp32-versions-tutorial/) |

### Pin mapping (MCP23S08 expander)

| MCP23S08 pin | Board bus | Function |
|---|---|---|
| REL1 (pin 7) | Bus 21 | Gate relay — triggers the gate motor/remote |
| IN1 (pin 3) | Bus 24 | IR safety curtain (active = obstruction detected) |
| IN2 (pin 2) | Bus 21 | External start button / remote click detection |
| IN3 (pin 1) | Bus 25 | Endstop — gate fully open |
| IN4 (pin 0) | Bus 26 | Endstop — gate fully closed |

### Gate timing

| Parameter | Value |
|---|---|
| Open travel time | 45 s |
| Close travel time | 45 s |
| Maximum run time | 2 min |
| Relay pulse width | 500 ms |

## Project structure

```
yardgate.yml          # Main ESPHome configuration
common/
  api.yml             # Home Assistant native API (encrypted)
  wifi.yml            # Wi-Fi + fallback hotspot
  ethernet.yml        # Ethernet (optional, disabled)
  mqtt.yml            # MQTT broker (optional, disabled)
  ota.yml             # Over-the-air updates
  logger.yml          # Serial logging (DEBUG level by default)
  time.yml            # SNTP time sync
  webserver.yml       # Built-in web server (v3, port 80)
  secrets.yaml        # All secrets (not committed)
```

## Setup

### Prerequisites

- [ESPHome](https://esphome.io) installed (CLI or Docker/Podman)
- USB-to-serial adapter for the first flash (subsequent flashes can use OTA)

### Secrets

Create `common/secrets.yaml` with the following keys:

```yaml
apiKey: "<32-byte base64 key for HA native API encryption>"
otaPwd: "<OTA password>"
web_server_username: "<web UI username>"
web_server_password: "<web UI password>"
vnAutomationWifiSsid: "<Wi-Fi SSID>"
vnAutomationWifiPwd: "<Wi-Fi password>"
fallbackWifiPwd: "<fallback hotspot password>"
timezone: "UTC"           # your local timezone (POSIX or TZ name)
timeserver: "pool.ntp.org"         # NTP server(s)

# Only needed if MQTT is enabled in yardgate.yml
mqttBrokerAddress: "<broker host>"
mqttBrokerProt: 1883
mqttUser: "<mqtt user>"
mqttPwd: "<mqtt password>"
```

### Compile and flash (Podman)

First flash over USB:

```bash
podman run --security-opt label=disable --rm \
  -v "${PWD}":/config \
  --device=/dev/ttyUSB0 \
  -it esphome/esphome run ./yardgate.yml
```

Subsequent OTA updates (no USB needed):

```bash
podman run --security-opt label=disable --rm \
  -v "${PWD}":/config \
  -it esphome/esphome run ./yardgate.yml
```

### Compile and flash (ESPHome CLI)

```bash
esphome run yardgate.yml
```

## Home Assistant integration

The device uses the **ESPHome native API** with encryption. Add it to Home Assistant via **Settings → Devices & Services → Add Integration → ESPHome**.

Exposed entities:

| Entity | Type | Description |
|---|---|---|
| YardGate | Cover (gate) | Open / close / stop the gate |
| YardGate Remote | Switch | Momentary relay pulse (500 ms) |
| Yard gate Controller Restart | Button | Restart the ESP32 |
| yardGateMotion | Binary sensor | IR safety curtain state |
| YardGate click | Binary sensor | External remote/button click |
| yardGateEndSwitchOpen | Binary sensor | Gate fully open endstop |
| yardGateEndSwitchClosed | Binary sensor | Gate fully closed endstop |
| Yard gate Controller wifi signal | Sensor | Wi-Fi RSSI (updated every 60 s) |

## License

See [LICENSE](LICENSE).

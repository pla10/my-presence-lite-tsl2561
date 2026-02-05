# Everything Presence Lite - ESP32-H2 OpenThread Variant

This is an **ESP32-H2** variant of the Everything Presence Lite that uses **OpenThread** (Thread/Matter networking) instead of WiFi.

## Hardware Requirements

- **ESP32-H2 board** (e.g., Waveshare ESP32-H2-Zero or ESP32-H2-DevKitM-1)
- **Thread Border Router** (Apple HomePod Mini, Google Nest Hub, or Home Assistant with a Thread dongle)
- **LD2450 mmWave sensor** (connected via UART)
- **TSL2561 light sensor** (connected via I2C)
- Optional: **SCD40 CO2 sensor** (for the CO2 variant)

## Key Differences from WiFi Version

| Feature | WiFi (ESP32) | OpenThread (ESP32-H2) |
|---------|--------------|----------------------|
| Networking | WiFi 2.4GHz | Thread (802.15.4) |
| Bluetooth | Supported | Not supported |
| IPv6 | Optional | Required |
| Range | Depends on WiFi | Mesh network |
| Power | Higher | Lower (mesh advantage) |

## Pin Mappings for ESP32-H2

| Function | GPIO Pin | Notes |
|----------|----------|-------|
| I2C SDA | GPIO12 | TSL2561 light sensor (safe non-strapping) |
| I2C SCL | GPIO11 | TSL2561 light sensor (safe non-strapping) |
| UART TX | GPIO4 | LD2450 mmWave sensor |
| UART RX | GPIO5 | LD2450 mmWave sensor |
| LED | GPIO13 | Status LED |
| Boot Button | GPIO9 | Factory reset function |

## Setup Instructions

### 1. Get Your Thread Network Credentials (TLV)

You need to obtain the Thread network credentials from your Thread Border Router:

#### From Home Assistant:
1. Go to **Settings** → **Thread**
2. Select your border router
3. Click **"Copy credentials"**
4. This copies the TLV (Thread Network Dataset) to your clipboard

#### Example TLV format:
```
0e080000000000010000000300001035060004001fffe00208dead00beef00cafe...
```

### 2. Update the YAML File

Edit `everything-presence-lite-ha-no-ble-h2.yaml` and replace the `openthread_tlv` value with your actual TLV:

```yaml
substitutions:
  openthread_tlv: "YOUR_TLV_HERE"
```

### 3. Flash the ESP32-H2

```bash
esphome run everything-presence-lite-ha-no-ble-h2.yaml
```

### 4. Add to Home Assistant

The device should automatically appear in Home Assistant via the Thread network:
- Go to **Settings** → **Devices & Services**
- Look for "Everything Presence Lite H2"
- Click **Configure** to add it

## Available Variants

- **everything-presence-lite-ha-no-ble-h2.yaml** - Base version with LD2450 sensor
- **everything-presence-lite-ha-no-ble-h2-co2.yaml** - With SCD40 CO2 sensor (to be created)

## Troubleshooting

### Device not appearing in Home Assistant
- Verify your Thread Border Router is online
- Check that the TLV is correct
- Ensure IPv6 is enabled in your network
- Check ESPHome logs: `esphome logs everything-presence-lite-ha-no-ble-h2.yaml`

### Compilation errors
- Ensure you're using ESPHome 2025.6.0 or later (OpenThread support)
- Verify the esp32-h2-devkitm-1 board is supported in your ESPHome version

### Sensor not detecting
- Check UART connections to LD2450 (GPIO4 TX, GPIO5 RX)
- Check I2C connections to TSL2561 (GPIO12 SDA, GPIO11 SCL)
- Run I2C scan to verify sensor addresses

## Technical Details

### OpenThread Configuration
- Uses ESP-IDF framework (required for OpenThread)
- IPv6-only networking
- Mesh networking capability
- Lower power consumption than WiFi

### Board Configuration
```yaml
esp32:
  board: esp32-h2-devkitm-1
  variant: ESP32H2
  framework:
    type: esp-idf
```

## References

- [ESPHome OpenThread Documentation](https://esphome.io/components/openthread/)
- [ESPHome 2025.6.0 Changelog](https://esphome.io/changelog/2025.6.0/)
- [Thread Group Official Site](https://www.threadgroup.org/)

## Notes

- **Bluetooth is NOT available** on ESP32-H2 variants (hardware limitation)
- **WiFi is NOT available** - ESP32-H2 only supports Thread/802.15.4
- Thread provides mesh networking, which can extend range through other Thread devices
- Requires a Thread Border Router to communicate with IPv4 networks

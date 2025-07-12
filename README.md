# ESP32 Optoma UHZ50 Connector for Home Assistant

Control your Optoma UHZ50 projector through ESP32 using ESPHome and integrate it seamlessly with Home Assistant.

## ⚠️ Security Notice

**IMPORTANT**: Before using this project, generate new API encryption key and passwords! The example keys in `secrets.yaml.example` are public and should never be used in production.

## Features

- Power ON/OFF control
- Input source switching (HDMI 1, 2, 3)
- Automatic projector state detection
- Full Home Assistant integration
- Real-time status monitoring

## Hardware Requirements

### ESP32 Board
- ESP32-S3-DevKitC-1 (or compatible)
- Alternative: M5Stack AtomS3

### Projector Connection
- **RX Pin**: GPIO5
- **TX Pin**: GPIO6
- **Baud Rate**: 9600
- **Protocol**: UART

### Wiring
Connect your ESP32 to the Optoma UHZ50's RS-232 port using a suitable level converter if needed.

## Project Structure

This project uses pure ESPHome configuration without custom components:

```
esp32-optoma-uhz50-connector/
├── optoma-uhz50.yaml                       # Main ESPHome configuration
├── secrets.yaml.example                   # Project secrets template
├── .gitignore                             # Git ignore file
└── README.md                              # This file
```

## How It Works

This project **doesn't require any custom components** - it uses built-in ESPHome features:

### UART Communication
- Uses ESPHome's built-in `uart.debug` feature with `dummy_receiver: true`
- All UART data processing happens in lambda functions directly in the YAML
- No C++ files needed - everything is pure YAML configuration!

## Quick Start Guide

### Step 1: Prerequisites

1. **Install ESPHome** (choose one method):
   ```bash
   # Using pip
   pip install esphome
   
   # Using Home Assistant Add-on (recommended)
   # Install "ESPHome" add-on from Home Assistant Supervisor
   ```

2. **Clone or download this project**:
   ```bash
   git clone <your-repo-url>
   cd esp32-optoma-uhz50-connector
   ```

### Step 2: Configuration

1. **Create secrets file**:
   ```bash
   cp secrets.yaml.example secrets.yaml
   ```

2. **Edit secrets.yaml** with project-specific credentials:
   ```yaml
   # Note: WiFi credentials should be in your global ESPHome secrets.yaml
   # This local secrets.yaml only contains project-specific keys:
   # - api_encryption_key
   # - ota_password  
   # - ap_ssid/ap_password
   ```

3. **Update security keys** in `secrets.yaml` (STRONGLY recommended):
   ```bash
   # Generate new API encryption key
   esphome wizard  # Follow prompts to get a new key
   
   # Or generate online at: https://esphome.io/components/api.html#configuration-variables
   # Update these values in secrets.yaml:
   # - api_encryption_key
   # - ota_password  
   # - ap_password
   ```

### Step 3: Compile and Flash

#### Option A: Using ESPHome CLI
```bash
# Compile and upload via USB
esphome run optoma-uhz50.yaml

# Or compile and upload Over-The-Air (after first flash)
esphome run optoma-uhz50.yaml --device <device-ip>
```

#### Option B: Using Home Assistant ESPHome Add-on
1. Copy this entire project folder to `/config/esphome/` in Home Assistant
2. Open ESPHome dashboard in Home Assistant
3. Click "New Device" → "Continue" → "Skip"
4. Delete the generated YAML and upload `optoma-uhz50.yaml`
5. Click "Install" → "Plug into this computer"

### Step 4: Home Assistant Integration

1. **Automatic Discovery**: 
   - After successful flash, the device should appear in Home Assistant automatically
   - Go to Settings → Devices & Services → Integrations
   - Look for "ESPHome" integration with "projector-connector"

2. **Manual Addition** (if auto-discovery fails):
   - Go to Settings → Devices & Services → Add Integration
   - Search for "ESPHome"
   - Enter your ESP32's IP address

### Step 5: Available Entities

After integration, you'll have these entities in Home Assistant:

| Entity | Type | Description |
|--------|------|-------------|
| `switch.projector_power` | Switch | Turn projector ON/OFF |
| `select.projector_source` | Select | Choose input source (HDMI 1/2/3) |

## Usage Examples

### Automation Example
```yaml
# automation.yaml
- alias: "Turn on projector when media starts"
  trigger:
    - platform: state
      entity_id: media_player.living_room_tv
      to: "playing"
  action:
    - service: switch.turn_on
      entity_id: switch.projector_power
    - delay: "00:00:05"
    - service: select.select_option
      entity_id: select.projector_source
      data:
        option: "HDMI 1"
```

### Lovelace Card Example
```yaml
# Add to your dashboard
type: entities
title: Projector Control
entities:
  - switch.projector_power
  - select.projector_source
```

## Protocol Details

The projector uses these UART commands:

| Command | Hex Bytes | Description |
|---------|-----------|-------------|
| Power ON | `7E 30 30 30 30 20 31 0D` | Turn projector on |
| Power OFF | `7E 30 30 30 30 20 30 0D` | Turn projector off |
| Status Query | `7E 30 30 31 32 34 20 31 0D` | Get power status |
| HDMI 1 | `7E 30 30 31 32 20 31 0D` | Switch to HDMI 1 |
| HDMI 2 | `7E 30 30 31 32 20 31 35 0D` | Switch to HDMI 2 |
| HDMI 3 | `7E 30 30 31 32 20 31 36 0D` | Switch to HDMI 3 |

### Debug Mode

Enable verbose logging by changing in `optoma-uhz50.yaml`:
```yaml
logger:
  level: VERBOSE
```

### Polling Interval
Adjust status checking frequency:
```yaml
interval:
  - interval: 30s  # Change from 20s to 30s
    then:
      - uart.write: [0x7E, 0x30, 0x30, 0x31, 0x32, 0x34, 0x20, 0x31, 0x0D]
```

## Contributing

Issues and pull requests welcome! Please follow ESPHome coding standards.

## License

This project follows ESPHome's open-source philosophy. Use freely for personal and commercial projects. 
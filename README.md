# ESPHome WLED Controller

An ESPHome-based controller for WLED-powered LED strips, integrated with Home Assistant. Features a 16x2 LCD display, rotary encoder for brightness, and a joystick-driven preset menu.

## Features

- **Brightness control** — rotary encoder adjusts WLED brightness (0-100%)
- **Power toggle** — rotary encoder push-button toggles WLED on/off
- **Preset menu** — joystick opens an on-screen menu to select WLED presets
- **LCD auto-blank** — display turns off after 10 seconds of inactivity
- **HA connection status** — LCD shows "No HA connection" when offline

## Hardware

| Component         | Details                                                    |
| ----------------- | ---------------------------------------------------------- |
| Microcontroller   | Freenove ESP32 Dev Board (esp32dev, dual-core 240MHz)      |
| Display           | 1602A QAPAS — 16x2 character LCD (HD44780, 4-bit parallel) |
| Encoder           | GIAK KY-040 rotary encoder with push-button                |
| Joystick          | KY-023 analog joystick with push-button (spring-return)    |
| Backlight control | 2N2222 NPN transistor + 1kΩ resistor                       |
| Contrast          | 10kΩ multiturn trimmer potentiometer                       |

## Wiring

### LCD (16x2, HD44780)

| LCD Pin           | Connection           |
| ----------------- | -------------------- |
| 1 VSS             | GND                  |
| 2 VDD             | 5V                   |
| 3 V0              | Trimpot wiper        |
| 4 RS              | GPIO23               |
| 5 R/W             | GND                  |
| 6 E               | GPIO22               |
| 7-10              | Not connected        |
| 11 DB4            | GPIO21               |
| 12 DB5            | GPIO19               |
| 13 DB6            | GPIO18               |
| 14 DB7            | GPIO4                |
| 15 A (backlight+) | 5V                   |
| 16 K (backlight-) | Transistor collector |

### Contrast trimpot

| Leg       | Connection     |
| --------- | -------------- |
| 1         | 5V             |
| 2 (wiper) | LCD pin 3 (V0) |
| 3         | GND            |

### Backlight transistor (2N2222 NPN)

| Pin       | Connection            |
| --------- | --------------------- |
| Base      | 1kΩ resistor → GPIO17 |
| Collector | LCD pin 16 (K)        |
| Emitter   | GND                   |

### Rotary encoder

| Pin              | Connection |
| ---------------- | ---------- |
| A                | GPIO34     |
| B                | GPIO35     |
| SW (push-button) | GPIO32     |

### Joystick

| Pin              | Connection |
| ---------------- | ---------- |
| Y axis           | GPIO36     |
| X axis           | GPIO39     |
| SW (push-button) | GPIO25     |

## Setup

1. Install [ESPHome](https://esphome.io/)
2. Copy `secrets.yaml.example` to your `secrets.yaml` and fill in the values
3. Update the `substitutions` block at the top of `esp32-led-controller.yaml` to match your Home Assistant entity IDs:
   ```yaml
   substitutions:
     light_entity: light.your_wled_light
     preset_entity: select.your_wled_preset_selector
   ```
4. Flash the device
5. Add the device to Home Assistant via the ESPHome integration
6. Adjust the contrast trimpot until characters are visible on the LCD

## WLED Presets

### How presets work

The preset names in the config must **exactly match** the preset names you create in WLED.

### Create presets in WLED

1. Open the WLED web interface
2. Set up the effect, colours, and speed you want
3. Go to **Presets** → save the preset with the exact name listed below
4. Repeat for each preset you want

### Default presets

These presets are already in the must exist in WLED with these exact names:

- Candle Multi
- Colorwaves
- Fireworks
- Gradient
- Lake
- Noise 3
- Random Colors
- Solid

### Adding custom presets

To add a preset, create it in WLED first, then add a new entry to the `lcd_menu` section in `esp32-led-controller.yaml`:

```yaml
- type: command
  text: "Your Preset Name"
  on_value:
    - homeassistant.action:
        action: select.select_option
        data:
          entity_id: ${preset_entity}
          option: "Your Preset Name"
    - display_menu.hide: preset_menu
    - script.execute: lcd_wake
```

Also update the `preset_count` global at the top of the config to match the new total number of presets:

```yaml
- id: preset_count
  type: int
  initial_value: "9" # change this to your new total
```

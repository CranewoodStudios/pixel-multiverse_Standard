# Button LED Configuration

## Overview

The `pm_daemon.py` now supports JSON-based button LED configuration with customizable attract mode patterns. This allows you to:

- Define LED coordinate mappings
- Configure attract mode patterns (linear, radial, circular)
- Customize colors, directions, and timing for each pattern

## Configuration File

Create a `buttons.json` file at `/recalbox/share/pixel-multiverse/buttons.json` (or configure `BUTTONS_JSON` in `pm_daemon.py`).

### Basic Structure

```json
{
  "buttons": {
    "enabled": true,
    "num_leds": 30,
    "refresh_rate": 60,
    "led_map": [
      {
        "coord": [0, 0],
        "value": 0
      }
    ],
    "attract_program": [
      {
        "pattern": "linear",
        "params": {
          "direction": "left_to_right",
          "color_on": [0, 0, 255, 40],
          "color_off": [0, 0, 0, 0],
          "delay": 0.05
        }
      }
    ]
  }
}
```

## LED Mapping (`led_map`)

Maps physical LED positions (x, y coordinates) to LED indices:

```json
"led_map": [
  {
    "coord": [0, 0],
    "value": 0
  },
  {
    "coord": [1, 0],
    "value": 1
  }
]
```

**Note:** Coordinates should reflect your physical button layout. For example, if you have buttons arranged in a 5x6 grid, use coordinates from [0,0] to [4,5].

## Attract Patterns

### Pattern Types

1. **linear** - Sweep patterns in straight lines
   - Directions: `left_to_right`, `right_to_left`, `top_to_bottom`, `bottom_to_top`

2. **radial** - Rotational patterns around center
   - Directions: `clockwise`, `anticlockwise`

3. **circular** - Expanding/contracting patterns from center
   - Directions: `outward`, `inward`

4. **sequential_colors** - Sequential color cycling pattern (Picade Max startup sequence)
   - Each LED flashes through: Red → Green → Blue → White
   - After all LEDs complete, fades all to off
   - Pattern then loops continuously

### Pattern Parameters

#### Common Parameters (linear, radial, circular)
- `direction`: Pattern direction (see above for each type)
- `color_on`: BGRBR tuple `[blue, green, red, brightness]` (0-255 each)
- `color_off`: BGRBR tuple for "off" state (optional, defaults to black)
- `delay`: Time in seconds between animation steps

#### Sequential Colors Parameters
- `num_leds`: Number of LEDs to animate (default: 7)
- `dwell_ms`: Milliseconds each color stays on per LED (default: 500)
- `fade_steps`: Number of fade steps for fade-out (default: 60)
- `fade_ms`: Milliseconds per fade step (default: 20)
- `brightness`: Maximum brightness level 0-255 (default: 255)

### Example Patterns

```json
"attract_program": [
  {
    "pattern": "linear",
    "params": {
      "direction": "left_to_right",
      "color_on": [255, 0, 0, 40],
      "color_off": [0, 0, 0, 0],
      "delay": 0.05
    }
  },
  {
    "pattern": "radial",
    "params": {
      "direction": "clockwise",
      "color_on": [0, 255, 255, 40],
      "color_off": [0, 0, 0, 0],
      "delay": 0.05
    }
  },
  {
    "pattern": "circular",
    "params": {
      "direction": "outward",
      "color_on": [255, 0, 255, 40],
      "color_off": [0, 0, 0, 0],
      "delay": 0.05
    }
  },
  {
    "pattern": "sequential_colors",
    "params": {
      "num_leds": 7,
      "dwell_ms": 500,
      "fade_steps": 60,
      "fade_ms": 20,
      "brightness": 255
    }
  }
]
```

## Color Format

Colors use BGRBR format (Blue, Green, Red, Brightness):
- Values range from 0-255
- Brightness is typically 0-40 for LEDs (hardware dependent)
- Examples:
  - Red: `[0, 0, 255, 40]`
  - Green: `[0, 255, 0, 40]`
  - Blue: `[255, 0, 0, 40]`
  - Yellow: `[0, 255, 255, 40]`
  - Purple: `[255, 0, 255, 40]`
  - Cyan: `[255, 255, 0, 40]`
  - White: `[255, 255, 255, 40]`

## Backward Compatibility

If `buttons.json` is not found:
- Falls back to hardcoded "breath" or "rainbow" modes
- Uses existing `NUM_LEDS` configuration from `pm_daemon.py`
- All event handling continues to work normally

## Reloading Configuration

Send a `reload-config` event to reload both `systems.json` and `buttons.json`:

```bash
echo '{"event":"reload-config"}' > /tmp/pm.fifo
```

## Troubleshooting

Check daemon logs for configuration messages:
- "loaded buttons.json: X LEDs mapped, Y patterns" - Success
- "buttons.json not found at..." - Configuration file missing
- "buttons.json not loaded: ..." - Parse error in JSON file

### JSON Format Validation

If you encounter parse errors, validate your JSON syntax:
```bash
python3 -m json.tool /recalbox/share/pixel-multiverse/buttons.json
```

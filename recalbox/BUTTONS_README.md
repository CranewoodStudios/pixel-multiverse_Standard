# LED Daemon Configuration

## Overview

The `pm_daemon.py` LED daemon uses a unified JSON configuration file that consolidates all hardware, button, and system settings. This allows you to:

- Configure hardware settings (LEDs, brightness, frame rate)
- Define LED coordinate mappings for button layouts
- Configure attract mode patterns (linear, radial, circular, sequential colors)
- Set system-specific colors and start animations
- Customize colors, directions, and timing for each pattern

## Configuration File

The unified configuration file is located at `/recalbox/share/pixel-multiverse/config.json`.

### Structure

```json
{
  "hardware": {
    "num_leds": 7,
    "led_order": [0, 1, 2, 3, 4, 5, 6],
    "brightness_limit": 170,
    "fps": 60
  },
  "paths": {
    "fifo": "/tmp/pm.fifo",
    "es_state": "/tmp/es_state.inf"
  },
  "buttons": {
    "enabled": true,
    "led_map": [...],
    "attract_program": [...]
  },
  "defaults": {
    "menu_color": {"b": 0, "g": 32, "r": 64, "br": 28},
    "attract_mode": "breath"
  },
  "systems": {
    "snes": {
      "accent": {"b": 0, "g": 0, "r": 255, "br": 36},
      "start_layout": [...],
      "rom_overrides": {...}
    }
  }
}
```

## Hardware Settings

The `hardware` section configures LED hardware parameters:

```json
"hardware": {
  "num_leds": 7,
  "led_order": [0, 1, 2, 3, 4, 5, 6],
  "brightness_limit": 170,
  "fps": 60
}
```

- `num_leds`: Total number of LEDs
- `led_order`: Physical LED order mapping (change if your wiring differs)
- `brightness_limit`: Maximum brightness cap (0-255)
- `fps`: Frame rate for animations

## Button LED Mapping

Maps physical LED positions (x, y coordinates) to LED indices:

```json
"buttons": {
  "enabled": true,
  "led_map": [
    {"coord": [0, 0], "value": 0},
    {"coord": [1, 0], "value": 1},
    {"coord": [2, 0], "value": 2}
  ]
}
```

**Note:** Coordinates should reflect your physical button layout. For example, if you have buttons arranged in a 5x6 grid, use coordinates from [0,0] to [4,5].

## Attract Patterns

The `attract_program` defines a sequence of patterns that cycle during attract mode:

### Pattern Types

1. **linear** - Sweep patterns in straight lines
   - Directions: `left_to_right`, `right_to_left`, `top_to_bottom`, `bottom_to_top`

2. **radial** - Rotational patterns around center
   - Directions: `clockwise`, `anticlockwise`

3. **circular** - Expanding/contracting patterns from center
   - Directions: `outward`, `inward`

4. **sequential_colors** - Sequential color cycling pattern
   - Each LED flashes through: Red → Green → Blue → White
   - After all LEDs complete, fades all to off
   - Pattern then loops continuously

### Pattern Parameters

#### Common Parameters (linear, radial, circular)
- `direction`: Pattern direction (see above for each type)
- `color_on`: Color tuple `[blue, green, red, brightness]` (0-255 each)
- `color_off`: Color tuple for "off" state (optional, defaults to black)
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

## System Configuration

Configure system-specific colors and game start animations:

```json
"systems": {
  "snes": {
    "accent": {"b": 0, "g": 0, "r": 255, "br": 36},
    "start_layout": [
      "#ff0000:100",
      "#ffffff:100",
      "#ffffff:30"
    ],
    "rom_overrides": {
      "Super Mario World": {
        "start_layout": ["#00ff00:100", "#ffffff:100"]
      }
    }
  }
}
```

- `accent`: System accent color used for menu animations
- `start_layout`: LED layout shown when game starts
- `rom_overrides`: Per-ROM custom layouts

## Color Formats

Colors can be specified in three formats:

### 1. Dictionary Format
```json
{"b": 0, "g": 32, "r": 64, "br": 28}
```

### 2. String Format (Hex)
```json
"#ff0000:100"
```
Format: `#RRGGBB:brightness`

### 3. Array Format
```json
[0, 0, 255, 40]
```
Format: `[blue, green, red, brightness]`

**Color Examples:**
- Red: `{"r": 255, "g": 0, "b": 0, "br": 40}` or `[0, 0, 255, 40]`
- Green: `{"r": 0, "g": 255, "b": 0, "br": 40}` or `[0, 255, 0, 40]`
- Blue: `{"r": 0, "g": 0, "b": 255, "br": 40}` or `[255, 0, 0, 40]`
- Yellow: `[0, 255, 255, 40]`
- Purple: `[255, 0, 255, 40]`
- Cyan: `[255, 255, 0, 40]`
- White: `[255, 255, 255, 40]`

## Defaults

The `defaults` section specifies fallback settings:

```json
"defaults": {
  "menu_color": {"b": 0, "g": 32, "r": 64, "br": 28},
  "attract_mode": "breath"
}
```

- `menu_color`: Default menu breathing animation color
- `attract_mode`: Default attract mode (currently must match configured patterns)

## Reloading Configuration

Send a `reload-config` event to reload the configuration:

```bash
echo '{"event":"reload-config"}' > /tmp/pm.fifo
```

## Troubleshooting

Check daemon logs for configuration messages:
- "loaded config.json: X systems, Y LEDs mapped, Z patterns" - Success
- "Configuration file not found: ..." - File missing
- "Failed to load configuration: ..." - Parse error or invalid structure

### JSON Format Validation

Validate your JSON syntax:
```bash
python3 -m json.tool /recalbox/share/pixel-multiverse/config.json
```

### Common Issues

1. **Missing required configuration**: The daemon now fails explicitly if required configuration is missing (no silent fallbacks)
2. **Invalid color format**: Ensure colors follow one of the three supported formats
3. **Missing system accent**: Game start animations require system accent colors to be configured

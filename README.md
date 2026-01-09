# M5 Atom Echo - Custom microWakeWord Configuration

This repository contains ESPHome configuration for M5 Atom Echo with a custom Ukrainian wake word "Havrylo".

## Repository Structure

```
.
├── atom-echo-1.yaml      # ESPHome device configuration
├── models/
│   ├── havrylo.json      # Wake word manifest file
│   └── havrylo.tflite    # TensorFlow Lite wake word model
└── README.md
```

## Custom Wake Word Model Requirements

When creating a custom microWakeWord model for ESPHome, the manifest JSON file must follow this exact format:

### Manifest File Format (`models/your_model.json`)

```json
{
  "type": "micro",
  "wake_word": "Your Wake Word",
  "author": "your_name",
  "model": "your_model.tflite",
  "trained_languages": ["en"],
  "version": 2,
  "micro": {
    "probability_cutoff": 0.7,
    "feature_step_size": 10,
    "sliding_window_size": 5,
    "tensor_arena_size": 22860,
    "minimum_esphome_version": "2024.7.0"
  }
}
```

### Critical Configuration Rules

1. **Model Path Format**: The `model` field MUST use a **relative path** (e.g., `"model": "your_model.tflite"`), NOT an absolute URL or `file://` scheme. The `.tflite` file must be in the same directory as the manifest JSON.

2. **Version**: Must be `2` for ESPHome 2024.7.0+

3. **Required Fields in `micro` section**:
   - `probability_cutoff`: Detection threshold (0.0-1.0)
   - `feature_step_size`: Audio feature extraction step (typically 10)
   - `sliding_window_size`: Averaging window size (typically 5)
   - `tensor_arena_size`: Memory allocation for TFLite (depends on model size)
   - `minimum_esphome_version`: Minimum compatible ESPHome version

4. **Optional Fields**:
   - `website`: Can be omitted entirely (do NOT use empty string `""`)

## ESPHome YAML Configuration

Reference the model using GitHub shorthand:

```yaml
micro_wake_word:
  models:
    - model: github://YOUR_USERNAME/YOUR_REPO/models/your_model.json@main
```

Or use direct HTTPS URL (manifest must then contain absolute URL for model):

```yaml
micro_wake_word:
  models:
    - model: https://raw.githubusercontent.com/YOUR_USERNAME/YOUR_REPO/main/models/your_model.json
```

## Building and Flashing

### Prerequisites

- ESPHome 2024.7.0 or later
- Python 3.x with esptool
- USB access to the device (user must be in `uucp` or `dialout` group)

### Commands

```bash
# Compile the firmware
esphome compile atom-echo-1.yaml

# Flash to device
esphome upload atom-echo-1.yaml --device /dev/ttyUSB0

# Monitor device logs
esphome logs atom-echo-1.yaml --device /dev/ttyUSB0
```

### Serial Port Permissions (Linux)

If you get permission errors, add your user to the appropriate group:

```bash
# For Arch Linux
sudo usermod -a -G uucp $USER

# For Debian/Ubuntu
sudo usermod -a -G dialout $USER

# Log out and back in for changes to take effect
```

## Troubleshooting

### "Invalid manifest file: Expected a file scheme or a URL scheme with host"

This error occurs when:
- The `model` field in the JSON uses an incorrect path format
- **Solution**: Use a relative filename (e.g., `"model": "havrylo.tflite"`) when hosting on GitHub

### Model not detected / Low accuracy

Adjust these parameters in the manifest:
- Lower `probability_cutoff` for more sensitive detection
- Increase `sliding_window_size` for more stable detection (may add latency)

### Out of memory errors

Increase `tensor_arena_size` in the manifest if you get tensor allocation failures.

## Hardware

- **Device**: M5Stack Atom Echo
- **Board**: ESP32 (m5stack-atom)
- **Framework**: ESP-IDF
- **Wake Word**: "Havrylo" (Ukrainian)

## License

Custom wake word model trained using [microWakeWord](https://github.com/kahrendt/microWakeWord).

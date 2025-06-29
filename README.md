# Sygic HUD Navigation Display

Complete navigation display system for ESP32-S3 with 480x320 touchscreen that receives navigation data from Sygic via BLE HUB and displays comprehensive navigation information.

## Features

### 🧭 Navigation Display
- **Turn Instructions**: Complete set of navigation arrows including:
  - Go straight, turn left/right
  - Enter/exit highway arrows
  - Roundabout indicators
  - Destination arrival icon

### 📏 Distance & Progress
- **Distance Display**: Shows distance to next turn (e.g., "350m", "1.2km")
- **Progress Bar**: Visual progress indicator for current navigation segment
- **Dynamic Units**: Automatically switches between meters and kilometers

### 🚗 Speed Information
- **Current Speed**: Large circular display with km/h indicator
- **Speed Limit**: Square speed limit sign with color coding
- **Speed Warning**: Red highlighting when exceeding speed limit

### ⚠️ Safety Features
- **Speed Camera Warnings**: Flashing red overlay with "CAMERA!" alert
- **Visual Alerts**: Color-coded warnings and status indicators

### 📊 Navigation Status
- **Connection Status**: Shows Sygic connection state
- **Route Status**: Displays "Navigation Active", "No Route", or "Recalculating"
- **ETA Display**: Shows estimated time of arrival (when available)

### 🎨 Interface Design
- **Optimized Layout**: Designed for 480x320 landscape display
- **High Contrast**: Clear visibility in various lighting conditions
- **Color Coding**: 
  - Green: Active navigation, normal speed
  - Red: Warnings, speed violations
  - Yellow: Cautions, recalculating
  - Gray: Inactive states

## Hardware Requirements

- **ESP32-S3** with 16MB Flash, 8MB PSRAM
- **480x320 TFT Display** with AXS15231B controller
- **Touch Controller** (AXS15231B_Touch)
- **BLE Support** for Sygic HUB communication

## Pin Configuration

Display pins are defined in `src/pincfg.h`:
- TFT_CS: 45
- TFT_SCK: 47  
- TFT_SDA0-3: 21, 48, 40, 39
- TFT_BL: 1

Touch pins:
- Touch_SDA: 4
- Touch_SCL: 8
- Touch_INT: 3

## Installation

1. Clone this repository
2. Open in PlatformIO IDE
3. Build and upload to ESP32-S3 device
4. Connect Sygic app via BLE HUB

## Usage

1. Power on the device
2. The display shows "Waiting for Sygic..."
3. In Sygic app, connect to "ESP32_Sygic_HUD" via BLE
4. Start navigation in Sygic
5. HUD displays comprehensive navigation information

## BLE Protocol

The system uses Sygic's standard BLE HUB protocol:
- Service UUID: `DD3F0AD1-6239-4E1F-81F1-91F6C9F01D86`
- Write Characteristic: `DD3F0AD3-6239-4E1F-81F1-91F6C9F01D86`
- Indicate Characteristic: `DD3F0AD2-6239-4E1F-81F1-91F6C9F01D86`

## Code Structure

- `src/main.cpp`: Main application with BLE handling and display functions
- `src/pincfg.h`: Hardware pin definitions
- `src/dispcfg.h`: Display configuration
- `src/AXS15231B_touch.h/cpp`: Touch controller driver

## Display Functions

Key rendering functions:
- `drawNavigationInterface()`: Main UI coordinator
- `drawTurnInstruction()`: Navigation arrows and icons
- `drawDistanceDisplay()`: Distance and progress information
- `drawSpeedSection()`: Speed and limit displays
- `drawSpeedCameraWarning()`: Safety warnings

## Navigation Data

The system tracks:
```cpp
struct NavigationData {
  uint8_t currentSpeed;      // Current vehicle speed
  uint8_t speedLimit;        // Road speed limit
  uint8_t direction;         // Turn direction code
  uint32_t distance;         // Distance to next turn
  String eta;                // Estimated arrival time
  bool hasRoute;             // Navigation active
  bool speedCameraWarning;   // Speed camera ahead
  bool deviceConnected;      // Sygic connection status
};
```

## Customization

The interface can be customized by modifying:
- Color definitions in `main.cpp`
- Layout coordinates in drawing functions
- Update intervals and timeouts
- Additional navigation features

## Building

Use PlatformIO with the provided configuration:
```bash
pio run
pio upload
pio monitor
```

## Troubleshooting

- **No Display**: Check display connections and power
- **No BLE Connection**: Verify Sygic HUB is enabled
- **Touch Issues**: Calibrate touch coordinates in `dispcfg.h`
- **Performance**: Monitor heap usage and optimize drawing calls 

# DBC to Arduino CAN Library Generator

This Python program parses DBC (Database CAN) files and generates Arduino-compatible libraries with functions to create CAN messages in binary form.

## Features

- **Complete DBC Parsing**: Supports parsing of DBC files with message definitions and signal specifications
- **Arduino Compatibility**: Generates C++ code compatible with Arduino IDE
- **Binary Frame Generation**: Creates functions that pack signals into 8-byte CAN frames
- **Multiple Data Types**: Handles various signal types (bool, uint8_t, uint16_t, uint32_t, float, etc.)
- **Byte Order Support**: Handles both Intel (little-endian) and Motorola (big-endian) byte ordering
- **Scaling and Offset**: Applies signal scaling factors and offsets automatically
- **Example Generation**: Creates Arduino sketch examples showing how to use the library

## Usage

```bash
python createLib.py <dbc_file> [options]
```

### Options

- `-o, --output`: Output directory (default: CANLibrary)
- `-n, --name`: Library name (default: CANMessages)

### Examples

```bash
# Generate library from TEST.dbc
python createLib.py TEST.dbc -o TEST_Library -n TEST_CAN

# Generate library from EXAMPLE.dbc with custom name
python createLib.py EXAMPLE.dbc -o MyCANLib -n MyCANMessages
```

## Generated Library Structure

The generator creates a complete Arduino library with the following structure:

```
OutputDirectory/
├── LibraryName.h          # Header file with message structures and function declarations
├── LibraryName.cpp        # Implementation file with pack/unpack functions
├── library.properties     # Arduino library metadata
└── examples/
    └── PackMessageName/
        └── PackMessageName.ino  # Example Arduino sketch
```

## Generated Functions

For each CAN message in the DBC file, the library generates:

1. **Message Structure**: A typedef struct containing all signals as appropriately typed fields
2. **Pack Function**: `pack_MessageName(const MessageName_t* data, uint8_t* frame)`
3. **Unpack Function**: `unpack_MessageName(const uint8_t* frame, MessageName_t* data)`

## Example Usage in Arduino

```cpp
#include <LIBRARY_CAN.h>

void setup() {
  Serial.begin(115200);
  
  // Create message data structure
  ID208_t absData;
  
  // Set signal values
  absData.Front_Wheel_Speed = 25.5;      // km/h
  absData.Rear_Wheel_Speed = 25.0;       // km/h
  absData.Front_Wheel_Sensor_Status = true;
  absData.Rear_Wheel_Sensor_Status = true;
  absData.ABS_System_Status_1 = 1;
  absData.Current_ABS_Mode = 0;
  absData.ABS_Warning_Light_Status = 0;
  absData.Modulator_ID = 42;
  absData.ABS_Odometer = 1000;
  
  // Pack into CAN frame
  uint8_t canFrame[8];
  pack_ID208(&absData, canFrame);
  
  // Send via CAN (using your preferred CAN library)
  // CAN.sendMsgBuf(CAN_ID208_ID, 0, 8, canFrame);
  
  // Or print the frame for debugging
  Serial.print("CAN Frame: ");
  for (int i = 0; i < 8; i++) {
    Serial.print(canFrame[i], HEX);
    Serial.print(" ");
  }
  Serial.println();
}
```

## Signal Data Types

The generator automatically selects appropriate Arduino data types based on signal properties:

- **Boolean signals** (1 bit): `bool`
- **Unsigned integers**: `uint8_t`, `uint16_t`, `uint32_t`, `uint64_t`
- **Signed integers**: `int8_t`, `int16_t`, `int32_t`, `int64_t`  
- **Scaled values** (factor ≠ 1.0): `float`

## Bit Packing

The library handles bit-level packing according to DBC specifications:

- **Intel Format (Little Endian)**: Bits are packed starting from the start bit, incrementing upward
- **Motorola Format (Big Endian)**: Bits are packed starting from the start bit, decrementing downward
- **Scaling**: Raw values are automatically scaled using the formula: `physical_value = raw_value * factor + offset`

## Requirements

- Python 3.6+
- No external dependencies (uses only standard library)

## Supported DBC Features

- ✅ Message definitions (BO_)
- ✅ Signal definitions (SG_)
- ✅ Node definitions (BU_)
- ✅ Multiple encodings (UTF-8, Latin-1, CP1252, ISO-8859-1)
- ✅ Intel and Motorola byte order
- ✅ Signed and unsigned signals
- ✅ Signal scaling and offset
- ✅ Signal length up to 64 bits

## Notes

- The generated library is self-contained and doesn't require external CAN libraries
- You'll need to integrate with your preferred Arduino CAN library (MCP2515, ESP32 CAN, etc.)
- All CAN frames are assumed to be 8 bytes (standard CAN)
- Message IDs are defined as preprocessor constants for easy reference

## Disclaimer

- This is a vibe coded mess so don't complain too much pls
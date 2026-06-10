# Telemetry Transmission Protocol (TTP)

TTP is a binary transmission protocol for aerospace vehicle communication, based on HTTP. Each 64-byte frame carries information such as sensor readings, flight data and a checksum field to make sure transmitted data is received in its entirety.

Uses a `TelemetryFrame` struct defined as so:

| Byte Offset | Field                | Type        | Description                                                                   |
|-------------|----------------------|-------------|-------------------------------------------------------------------------------|
| 0           | Sync Word            | uint16_t    | Always 0x1ACF (used for validation)                                           |
| 2           | Version              | uint8_t     | Protocol version, currently 0x02                                              |
| 3           | Packet ID            | uint32_t    | Unique packet ID                                                              |
| 7           | Timestamp            | uint32_t    | Milliseconds since startup.                                                   |
| 11          | Status Code          | uint16_t    | Transmission status codes                                                     |
| 13          | Payload              | ...         | Flight data                                                                   |
| 62          | Checksum             | uint16_t    | Packet checksum (used for validation). CRC16-CCITT                            |

## Payload Data

| Byte Offset | Field | Type | Description |
|-------------|-------|------|-------------|
| 13 | Altitude | int32_t | Millimetres |
| 17 | Velocity | int32_t | Millimetres per second |
| 21 | Acceleration | int32_t | Milli-m/s² |
| 25 | Mass | uint32_t | Milligrams |
| 29 | Thrust | uint32_t | Milli-Newtons |
| 33 | Dynamic Pressure | uint32_t | Milli-Pascals |
| 37 | Mach | uint32_t | Mach × 1000 |
| 41 | Battery | uint16_t | Millivolts |
| 43 | Reserved | uint8_t[19] | Padding to byte 62 |

## Status Code Breakdown
The 16-bit status code is divided into functional 4-bit chunks to encode messages.

| Bits  | Name   | Description                                       |
|-------|--------|---------------------------------------------------|
| 15-12 | Origin | 0: Vehicle, 1: Ground Control, 2: Relay         |
| 11-8  | Class  | e.g. 2: Success, 3: Client Error, 4: Server Error | 
| 7-0   | Detail | Encodes any extra data (if needed)                | 

## Class Codes

### 0XX Loopback Codes
Sends a packet with identical packet ID with success code 200 OK.

### 1XX Informational Response Codes
Status responses with no immediate action required.

### 2XX Success Codes
#### 200 OK
Standard response for successful TTP requests.

### 4XX Client (Ground Control) Error Codes
#### 400 BAD REQUEST
Used when ground control sends a bad packet (sync word mismatch, checksum validation failure, etc.)

### 5XX Server (Vehicle) Error Codes

### EXX Log Codes
#### E01 LAUNCH
Launch has been transmitted.
#### E02 APOGEE
Apogee has been reached in flight.
#### E03 CHUTE_DEPLOYED
Vehicle parachute has been successfully deployed.
#### E04 LANDED
Vehicle has landed safely.

### FXX Emergency Codes
#### F01 CRITFAIL
Critical failure has occurred on the vehicle. Immediate mission failure.

## Control Flow
TTP utilises a Request-Acknowledge pattern, similar to HTTP/HTTPS:
1. Ground Control sends a command packet (e.g. `0x1000`)
2. Vehicle validates the checksum.
3. Vehicle responds with the same ID and a `0x0200` (OK) status code to confirm execution.

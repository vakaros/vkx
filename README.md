#### VKX Version: 1.3

### Abstract

VKX is a basic binary serialization format used by Vakaros devices for logging. It is optimized to be compact and efficient to write. This document describes how to parse the various message types that occur in VKX files.

### Primitive Types

This document uses the following type IDs to describe primitive data types. All types are in little endian byte order. 

| ID   | Size in Bytes | Description                                            |
| :--- | ------------- | ------------------------------------------------------ |
| U1   | 1             | Unsigned 8-bit integer                                 |
| U2   | 2             | Unsigned 16-bit integer                                |
| U4   | 4             | Unsigned 32-bit integer                                |
| U8   | 8             | Unsigned 64-bit integer                                |
| I1   | 1             | Signed 8-bit integer                                   |
| I2   | 2             | Signed 16-bit integer                                  |
| I4   | 4             | Signed 32-bit integer                                  |
| F4   | 4             | 32-bit IEEE 754 single precision floating point number |
| X4   | 4             | 32-bit bitfield                                        |
| S4   | 4             | Fixed-size string (4 bytes)                            |
| S32  | 32            | Fixed-size string (32 bytes)                           |

### 

### Row Structure

A VKX file is separated into "rows" that always have an unique `U8` key and a fixed-size payload defined for that key. 



## Row Definitions

### `0xFF` Page Header

The page header occurs approximately every 2 kB and also designates which log format version is being used. 

| Index | Type | Length | Description                        |
| ----- | ---- | ------ | ---------------------------------- |
| 0     | U1   | 1      | VKX format version number. See [revision history](#revision-history) |
| 1     | -    | 6      | Internal log state                 |

Payload Total Size: 7 bytes



### `0xFE` Page Terminator

The page terminator is used to optimize iterating backwards through a log. 

| Index | Type | Length | Description                                                  |
| ----- | ---- | ------ | ------------------------------------------------------------ |
| 0     | U2   | 2      | Length of previous page including header and terminator in bytes |

Payload Total Size: 2 bytes



### `0x02` Position, Velocity, and Orientation

This is the primary message for telemetry data. 

| Index | Type | Length | Description                                                  |
| ----- | ---- | ------ | ------------------------------------------------------------ |
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC                           |
| 1     | I4   | 4      | Latitude as a 32-bit integer in 10<sup>-7</sup> degrees      |
| 2     | I4   | 4      | Longitude as a 32-bit integer in 10<sup>-7</sup> degrees     |
| 3     | F4   | 4      | Speed Over Ground (SOG) in meters/second                     |
| 4     | F4   | 4      | Course Over Ground (COG) in radians                          |
| 5     | F4   | 4      | Altitude in meters                                           |
| 6     | F4   | 4      | Quaternion (w) - Orientation in true NED (North, East, Down) frame |
| 7     | F4   | 4      | Quaternion (x)                                               |
| 8     | F4   | 4      | Quaternion (y)                                               |
| 9     | F4   | 4      | Quaternion (z)                                               |

Payload Total Size: 44 bytes



### `0x03` Declination

The declination message is logged infrequently containing only the current declination offset and the current lat/lon pair.

| Index | Type | Length | Description                                              |
| ----- | ---- | ------ | -------------------------------------------------------- |
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC                       |
| 1     | F4   | 4      | Declination offset in radians                            |
| 2     | I4   | 4      | Latitude as a 32-bit integer in 10<sup>-7</sup> degrees  |
| 3     | I4   | 4      | Longitude as a 32-bit integer in 10<sup>-7</sup> degrees |

Payload Total Size: 20 bytes

### `0x04` Race Timer Event

Race timer events are logged when he timer is set, reset, synced, or incremented or the races began or ended. The packet only has the timer event type and the timer value when that button was hit.

| Index | Type | Length | Description                                                 |
| ----- | ---- | ------ | ----------------------------------------------------------- |
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC                          |
| 1     | U1   | 1      | Event type ID 8-bit unsigned integer, see event table below |
| 2     | I4   | 4      | Timer value in seconds                                      |

Payload Total Size: 13 bytes

#### Timer Event Table

| Value | Event      | Description                                |
| ----- | ---------- | ------------------------------------------ |
| 0     | RESET      | Timer has been reset to its default value  |
| 1     | START      | Timer has been started                     |
| 2     | SYNC       | Timer has been synced to a different value |
| 3     | RACE_START | Timer has reached 0, race has begun        |
| 4     | RACE_END   | User has indicated the race has ended      |



### `0x05` Line Position

The line position packet is logged when the user sets the pin or boat. These are simply the position type (pin or boat) and the position at lat/lon.

| Index | Type | Length | Description                                              |
| ----- | ---- | ------ | -------------------------------------------------------- |
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC                       |
| 1     | U1   | 1      | Line end type, see table below                           |
| 2     | F4   | 4      | Latitude as a 32-bit float.                              |
| 3     | F4   | 4      | Longitude as a 32-bit float.                             |

Payload Total Size: 17 bytes

#### Line End Type Table

| Value | Description  |
| ----- | ------------ |
| 0     | Pin (left)   |
| 1     | Boat (right) |



### `0x06`  Shift Angle

The shift angle packet determines the angle of either the port or starboard tack. The packet logs whether that tack was set using the auto shift tracking UI or manually set. It contains the true heading (never magnetic) and average speed on that tack in knots.

| Index | Type | Length | Description                                         |
| ----- | ---- | ------ | --------------------------------------------------- |
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC                  |
| 1     | U1   | 1      | Tack ID, 0 = starboard, 1 = port                    |
| 2     | U1   | 1      | Set by manual or auto process. 0 = auto, 0 = manual |
| 3     | F4   | 4      | True heading in degrees                             |
| 4     | F4   | 4      | Speed over Ground in knots                          |

Payload Total Size: 18 bytes



### `0x08`  Device Configuration

| Index | Type | Length | Description                                 |
| ----- | ---- | ------ | ------------------------------------------- |
| 0     | U8   | 8      | Unused.                                     |
| 1     | X4   | 4      | Bitfield of configuration states, see table |
| 2     | U1   | 1      | Telemetry logging rate in Hz                |

Payload Total Size: 13 bytes

#### Configuration Bitfield

| Bit Index | Description                            |
| --------- | -------------------------------------- |
| 0         | Device Fixed to Body Frame, true/false |
| 1-31      | Unused                                 |


### `0x0A` Wind

Reading from a Calypso Wind Sensor. Both wind speed and wind direction are apparent.

| Index | Type | Length | Description                               |
| ----- | ---- | ------ | ------------------------------------------|
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC        |
| 1     | F4   | 4      | Wind direction, in degrees                |
| 2     | F4   | 4      | Wind speed in m/s                         |

Payload Total Size: 16 bytes


### `0x0B` Speed Through Water

Speed through water reading from a transducer

| Index | Type | Length | Description                                 |
| ----- | ---- | ------ | --------------------------------------------|
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC          |
| 1     | F4   | 4      | Speed of water in forward direction, m/s    |
| 2     | F4   | 4      | Speed of water in horizontal direction, m/s |

Payload Total Size: 16 bytes


### `0x0C` Depth

Depth reading from a transducer

| Index | Type | Length | Description                                 |
| ----- | ---- | ------ | --------------------------------------------|
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC          |
| 1     | F4   | 4      | Depth, in meters                            |

Payload Total Size: 12 bytes


### `0x10` Temperature

Temperature reading from a transducer

| Index | Type | Length | Description                                 |
| ----- | ---- | ------ | --------------------------------------------|
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC          |
| 1     | F4   | 4      | Temperature, in degrees C                   |

Payload Total Size: 12 bytes


### `0x0F` Load

Load reading from a Cyclops load cell

| Index | Type | Length | Description                                 |
| ----- | ---- | ------ | --------------------------------------------|
| 0     | U8   | 8      | Unix timestamp in milliseconds UTC          |
| 1     | S4   | 4      | Sensor short name                           |
| 2     | F4   | 4      | Load amount                                 |

Payload Total Size: 16 bytes

#### Measurement Units



## Vakaros Internal Messages

The following rows are used internally but because VKX is fixed-size they need to be parsed correctly.

| Key    | Payload Size in Bytes |
| ------ | --------------------- |
| `0x01` | 32                    |
| `0x07` | 12                    |
| `0x0E` | 16                    |
| `0x20` | 13                    |



### Revision History

| Revision | Revision Code | Date        | Comments     |
| -------- |---------------| ----------- | ------------ |
| R01      | `0x00`        | 24-May-2021 | VKX 1.0 Spec |
| R02      | `0x01`        | 14-Dec-2022 | VKX 1.1 Spec |
| R03      | `0x02`        | 01-Mar-2023 | Fix size of `0x0E` message. |
| R04      | `0x03`        | 27-Apr-2023 | VKX 1.2. Temp message has new ID of `0x10` |
| R05      | `0x04`        | 04-Oct-2023 | VKX 1.3 |
| R05-Fix  | `0x04`        | 04-Mar-2024 | Fix unit type in doc to match what's actually being exported |



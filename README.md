GM1365 USB Protocol Reverse Engineering
---------------------------------------

Data is to be interpreted as Big Endian.

# Write Metadata

Send 27 bytes to send to endpoint `0x03`.

| Field               | Size | Type                          |                              Unit |
|:--------------------|:----:|:-----------------------------:|----------------------------------:|
| `0x03`              | 1    | `u8`                          |                                   |
| `sample_period`     | 2    | `u16`                         |                            second |
| `record_interval`   | 2    | `u16`                         |                            second |
| `upper_temp_limit`  | 4    | [`temperature`](#temperature) |                           celcius |
| `lower_temp_limit`  | 4    | [`temperature`](#temperature) |                           celcius |
| `upper_hum_limit`   | 3    | [`humidity`](#humidity)       |      relative humidity percentage |
| `lower_hum_limit`   | 3    | [`humidity`](#humidity)       |      relative humidity percentage |
| `temperature_scale` | 1    | `u8`                          | `0x00` Celcius / `0x01` Farenheit |
| `current_date_time` | 7    | [`datetime`](#date-time)      |                                   |

# Read Metadata

Send `0xb4 0x4b` to endpoint `0x03`.

Response comes by polling endpoint `0x82` asking 64 bytes at a time and stopping when the reply is empty.

Answer is 2048 byte long.

| Field                    | Size     | Type                                           |
|:-------------------------|:--------:|:----------------------------------------------:|
| `last_switched_on_state` | 256      | [`power_state`](#power-state)                  |
| `last_switched_off_state`| 256      | [`power_state`](#power-state)                  |
| `session_metadata`       | 12       | [`session_metadata`](#record-session-metadata) |
| ...                      | ...      | ...                                            |
| `session_metadata`       | 12       | [`session_metadata`](#record-session-metadata) |

# Clear EEPROM
Send `0xe1 0x1e` to endpoint `0x03`.

# Read Data
Send `0xd2 0x2d` to endpoint `0x03`.

Response comes by polling endpoint `0x82` asking 64 bytes at a time and stopping when the reply is empty.

This is an array of pairs of `uint16`. First value of the pair is the temperature, second value is the humidity.

| Field         | Size  | Type  |
|:--------------|:-----:|:-----:|
| `temperature` | 2     | `u16` |
| `humidity`    | 2     | `u16` |

Current formula for temperature: `Celcius = 0.002680183077 * value - 46.83085034`.\
Current formula for humidity: `%RH = 0.001907290207 * value - 6.002171676`.

## Power State

| Field               | Size   | Type                          |                              Unit |
|:--------------------|:------:|:-----------------------------:|----------------------------------:|
| `record`              | 2    | `u16`                         |                                   |
| `record_session_index`| 1    | `u8`                          |                                   |
| `sample_period`       | 2    | `u16`                         |                            second |
| `record_interval`     | 2    | `u16`                         |                            second |
| `upper_temp_limit`    | 4    | [`temperature`](#temperature) |                           celcius |
| `lower_temp_limit`    | 4    | [`temperature`](#temperature) |                           celcius |
| `upper_hum_limit`     | 3    | [`humidity`](#humidity)       |      relative humidity percentage |
| `lower_hum_limit`     | 3    | [`humidity`](#humidity)       |      relative humidity percentage |
| `temperature_scale`   | 1    | `u8`                          | `0x13` Celcius / `0x31` Farenheit |
| ...                   | 15   |                               | 15 unknown bytes                  |

 ## Record Session Metadata
 Each record is 12 byte long.

| Field               | Size | Type                          |                              Unit |
|:--------------------|:----:|:-----------------------------:|----------------------------------:|
| `session_index`     | 1    | `u8`                          |                                   |
| `sample_period`     | 2    | `u16`                         |                            second |
| `record_interval`   | 2    | `u16`                         |                            second |
| `current_date_time` | 7    | [`datetime`](#date-time)      |                                   |

## Temperature

Encoded as 4 `uint8`:
 - `0x13` for positive temperature, `0x31` for negative
 - digit of tens
 - digit of units
 - digit of tenth

 e.g.
  -  `80.0` is encoded as `0x13 0x08 0x00 0x00`
  - `-21.2` is encoded as `0x31 0x02 0x01 0x02`

## Humidity
Encoded as 3 `uint8`:
 - digit of tens
 - digit of units
 - digit of tenth

 _e.g._
  -  `99.9` is encoded as `0x09 0x09 0x09`
  - `12.3` is encoded as `0x01 0x02 0x03`

## Date Time
Encoded as 7 `uint8`:
 - Current year minus 2000
 - Month ordinal
 - Day ordinal
 - Hour ordinal in 24h format
 - Minute ordinal
 - Second ordinal
 - `0x3c`

_e.g._ The date `2021-10-18 22:51:23` would be encoded as `0x15 0x0a 0x12 0x16 0x33 0x17 0x3c`.
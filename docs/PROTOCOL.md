# CVR DSP amp UDP protocol (as used by this plugin)

Notes on the subset of CVR's native protocol the plugin implements. Ported from [OpenAmpCore/ampcore](https://github.com/OpenAmpCore/ampcore) and verified against firmware 1.1.8 / 1.1.9. All multi-byte values are little-endian.

## Transport

- UDP. The amp listens on **45455**; the controller listens on **45454** (configurable in the plugin).
- No discovery is used. The controller sends directly to the amp's IP.

## Datagram layout

Each datagram is a 10-byte **network header** followed by a **frame**.

### Network header (10 bytes)

| Offset | Size | Field | Notes |
|---|---|---|---|
| 0 | u16 | flag | Always `0xD903` |
| 2 | i16 | machine_mode | Amp-level state (see state table) |
| 4 | u8 | packets_count | > 1 means a fragmented message; ignored by this plugin |
| 5 | u16 | lastlen | Length of the frame |
| 7 | u8 | step | |
| 8 | u8 | data_state | 0 = data, 1 = ACK |
| 9 | u8 | pad | |

### Frame

| Offset | Size | Field | Notes |
|---|---|---|---|
| 0 | u8 | start | `0x55` |
| 1 | u8 | function code | 0 = BASIC_INFO, 6 = HEARTBEAT |
| 2 | u8 | status | 2 = request |
| 3 | u8 | channel | |
| 4 | u8 | segment | |
| 5 | i32 | link | |
| 9 | u8 | in/out | |
| 10 | … | body | Response payload |
| end−3 | 3 | checksum | `len_hi, len_lo, sum` |

**Checksum:** `n = frame_length` (inner bytes + 3). `len_hi = n >> 8`, `len_lo = n & 0xFF`, `sum = (len_hi + len_lo + Σ inner bytes) & 0xFF`. As in the reference implementation, only `len_lo` and `sum` are checked on receive.

## Handshake

- **Query:** the controller sends a frame with no body and status 2 for the wanted function code.
- **ACK:** every inbound datagram with `data_state = 0` must be acknowledged by echoing its 10-byte network header with `data_state` set to 1. Without this the amp stops replying.
- Inbound datagrams with `data_state ≠ 0` are the amp's ACKs of our queries and are ignored.

## FC 0 — BASIC_INFO

Body is 75–87 bytes (79 and 87-byte bodies carry 4 trailing bytes that are stripped).

| Offset | Field |
|---|---|
| 0–23 | Firmware string (NUL-terminated); model is matched from this |
| 32–(mac) | Device name |
| 72 or 64 | MAC address (6 bytes; 72 tried first, 64 if that's all zeros) |
| mac+7 | Analog input count |
| mac+8 | Digital input count |
| mac+9 | Output count |
| mac+10 | Machine state |

## FC 6 — HEARTBEAT

Supported body lengths: **72, 76, 88, 92, 96** bytes. Two layouts exist; the plugin tells them apart by checking which offset holds four plausible state bytes (each ≤ 11).

### Legacy layout (state bytes at 52)

| Offset | Content |
|---|---|
| 0 | 5 × f32 temperatures (ch A–D, PSU) |
| 20 | 4 × f32 output voltage |
| 36 | 4 × f32 output current |
| 52 | 4 × u8 output state |
| 56 | 4 × f32 input voltage |
| 72 | 4 × f32 limiter gain reduction (≥ 88 bytes) |
| 88 | 4 × i8 input state (≥ 92 bytes) |
| 92 | f32 fan voltage (96 bytes) |

### Shifted layout (state bytes at 36)

| Offset | Content |
|---|---|
| 0 | 4 × f32 output voltage |
| 20 | 4 × f32 output current |
| 36 | 4 × u8 output state |
| 40 | 4 × f32 input voltage |
| 56 | 4 × f32 limiter gain reduction |
| 72 | 4 × i8 input state (≥ 76 bytes) |

Input state: 0 = clip, 1 = none.

## State values

| Value | Name | Treated as fault |
|---|---|---|
| −1 | Offline | |
| 0 | Normal | |
| 1 | Standby | |
| 2 | Fault | ✓ |
| 3 | Open | ✓ |
| 4 | Overload | ✓ |
| 5 | Clip | |
| 6 | DCP | ✓ |
| 7 | Power Error | ✓ |
| 8 | Run | |
| 9 | Over Temp | (Temp LED) |
| 10 | Limit | (Limit LED) |
| 11 | Sleep | |

## Metering math

- Input level: `20·log10(V / 1 V)` → dBV.
- Output level: `20·log10(V / rated Vrms)` → dB re full rated output.
- Load: `V / A` when current > 0.01 A.

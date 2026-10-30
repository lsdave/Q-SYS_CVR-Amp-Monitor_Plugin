# Controls reference

All controls marked *pin* can be exposed as control pins on the plugin block. Array controls have one instance per channel (inputs 1–4, outputs A–D map to index 1–4).

## Connection and amp

| Control | Type | Pin | Description |
|---|---|---|---|
| IPAddress | Text | Input | Amp IP address. Changing it restarts the connection if Connect is on. |
| Connect | Toggle button | Both | Starts/stops polling. |
| Status | Status | Output | Overall plugin status (see README). |
| Online | LED | Output | Lit while heartbeats are arriving. |
| Model | Text | Output | Detected or selected model and its rated Vrms. |
| DeviceName | Text | Output | Name configured on the amp. |
| Firmware | Text | Output | Firmware string reported by the amp. |
| MAC | Text | Output | Amp MAC address. |
| AmpState | Text | Output | Amp-level machine mode (Normal, Standby, Sleep, Fault…). |
| PSUTemp | Text | Output | Power-supply temperature, °C. |
| FanVolts | Text | Output | Fan drive voltage (only on 96-byte heartbeats). |

## Inputs (×4)

| Control | Type | Range | Description |
|---|---|---|---|
| InLevel | Meter | −60 … +20 dBV | Input level, smoothed. |
| InLevelText | Text | | Input level as text, e.g. `-12.4 dBV`. |
| InVolts | Text | | Raw input voltage. |
| InClip | LED | | Input clip, with hold. |

## Outputs (×4, A–D)

| Control | Type | Range | Description |
|---|---|---|---|
| OutLevel | Meter | −60 … +3 dB | Output voltage in dB relative to rated Vrms. 0 dB = full rated output. |
| GainReduction | Meter | 0 … 24 dB | Limiter gain reduction, positive dB. Empty at 0, so it only lights when the limiter is working. |
| OutLevelText | Text | | Output level as text, or `no model` if no rating is known. |
| GRText | Text | | Gain reduction as text. |
| OutVolts | Text | | Output voltage, V. |
| OutAmps | Text | | Output current, A. |
| OutOhms | Text | | Calculated load, V ÷ A (shown when current > 0.01 A). |
| OutTemp | Text | | Channel temperature, °C. |
| ChState | Text | | Channel state name. |
| Limit | LED | | Limiting (state = Limit, or GR ≥ threshold), with hold. |
| Clip | LED | | Output clip, with hold. |
| Fault | LED | | Fault, Open, Overload, DCP or Power Error, with hold. |
| TempWarn | LED | | Over-temperature, with hold. |

## Diagnostics (not pinnable)

| Control | Description |
|---|---|
| RxCount / TxCount | Valid packets received / packets sent. |
| BadFrames | Packets that failed the checksum. |
| HBRate | Measured heartbeat replies per second vs the requested rate. |
| BodyLength | Heartbeat body length in bytes (72, 76, 88, 92 or 96). |
| HeartbeatLayout | `legacy` or `shifted` (see PROTOCOL.md). |
| RawOutStates / RawInStates | Raw per-channel state bytes. |
| RawMachineMode | Raw machine-mode value from the network header. |
| LocalPort | UDP port the plugin is listening on. |
| RawHex | Hex dump of the last heartbeat body. |

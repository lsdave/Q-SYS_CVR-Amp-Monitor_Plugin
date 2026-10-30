# Changelog

## 1.1.1 — 2026-10-30
- Rebranded to **Atem Audio**. The plugin now appears under *Plugins → Atem Audio*.
- Plugin Id unchanged, so existing designs upgrade in place.

## 1.1.0
- Meters are drawn by a separate render timer with attack/release ballistics, so they glide between heartbeat samples instead of stepping.
- New properties: Meter Refresh, Meter Attack, Meter Release.
- Default poll interval lowered to 50 ms.
- Diagnostics: measured heartbeat rate.

## 1.0.x
- Initial release: input/output metering, gain reduction, channel state, temperatures, V/A/Ω readouts, status output and diagnostics page for CVR DSP amps on firmware 1.1.8/1.1.9.

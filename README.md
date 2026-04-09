# Crane Scale Logger

A browser-based app for connecting to the **Crane AES-CDSC-1 Bluetooth Diagnostic Scale** (and likely other Crane/ALDI CDSC variants) via Web Bluetooth, logging weight readings, and exporting data as CSV.

> **No install. No server. No account.** Just open `index.html` in Chrome or Edge.

![Status](https://img.shields.io/badge/status-working-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## Features

- Connects directly to the scale via Web Bluetooth (BLE)
- Auto-logs every reading with timestamp
- Displays weight in **kg**, **lbs**, and **stones/lbs** simultaneously
- Session stats: count, min, max, average
- One-click **CSV export**
- Raw BLE feed for debugging
- No dependencies, no frameworks, single HTML file

---

## Requirements

| Requirement | Detail |
|---|---|
| Browser | **Chrome or Edge** (desktop or Android) |
| Protocol | Firefox and Safari do **not** support Web Bluetooth |
| Connection | HTTPS or `localhost` required (or open local file directly) |
| Scale | Crane AES-CDSC-1 / AU5-CDSC-1 / AE5-CDSC-1 (ALDI Bluetooth Diagnostic Scale) |

---

## Usage

1. Clone or download the repo
2. Open `index.html` in Chrome or Edge
3. Wake the scale by stepping on it briefly
4. Click **Connect to Scale**
5. Select your Crane scale from the browser's device picker
6. Step on the scale — readings log automatically

> On subsequent visits the browser remembers the scale. If connection fails, wake the scale first then click Connect.

---

## How it works

The scale uses a **Texas Instruments CC254x** BLE chipset with a proprietary GATT service.

| | UUID |
|---|---|
| Service | `0000ffe0-0000-1000-8000-00805f9b34fb` |
| Characteristic | `0000ffe1-0000-1000-8000-00805f9b34fb` |

The app subscribes to BLE notifications on the characteristic. Weight packets are identified by the header bytes `E7 58` and decoded as:

```
Weight (kg) = ((byte[3] << 8) | byte[4]) / 20
```

This was reverse engineered by sniffing BLE traffic with **nRF Connect** and correlating byte values to known weights.

### Packet types

| Header | Type | Notes |
|---|---|---|
| `E7 58` | Weight reading | Parsed and logged |
| `E7 59` | Body composition | Fat %, water %, muscle %, bone mass — requires user profile to be set (all zeros otherwise) |

---

## CSV Export

Exported files are named `crane-scale-YYYY-MM-DD.csv` and contain:

```
No, Timestamp, Weight (kg), Weight (lbs), Weight (st lbs), Raw Bytes
```

---

## Extending

Possible additions:
- **Body composition parsing** — the `E7 59` packet contains fat/water/muscle/bone data once a user profile (age, height, sex) is written to the characteristic
- **Trend chart** — Chart.js line graph over session readings
- **Multiple users** — profile selection before weighing
- **LocalStorage persistence** — retain readings across browser sessions

Pull requests welcome.

---

## Compatibility

Tested with:
- Crane AES-CDSC-1 (article number 92125)
- Chrome on Windows

May also work with other ALDI/Crane-branded scales using the same TI CC254x chipset (AU5-CDSC-1, AE5-CDSC-1).

---

## Reverse Engineering Notes

The BLE protocol was identified using:
1. **nRF Connect** (Android) to discover services and characteristics
2. Subscribing to notifications on `0000ffe1` and correlating raw bytes to scale display readings
3. Two data points used to derive the weight formula: 89.9 kg and 90.1 kg

The `f000ffc0-0451-4000-b000-000000000000` service (also advertised) is a TI CC254x OAD (Over-Air Download) service and is not used for measurements.

---

## License

MIT — do whatever you like with it.

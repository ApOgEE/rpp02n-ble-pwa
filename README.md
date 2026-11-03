# RPP02N Web Bluetooth PWA

A lightweight Progressive Web App (PWA) for **printing ESC/POS receipts directly to an RPP02N Bluetooth Low Energy (BLE) thermal printer**, exploring GATT services, and inspecting standard BLE device identity information.

**[Launch the PWA](https://apogee.github.io/rpp02n-ble-pwa/)** · **[Repository](https://github.com/ApOgEE/rpp02n-ble-pwa)**

> This is a browser-based proof of concept, not a production printer driver or a validated BMS monitoring application.

## Features

| Page | Function |
| --- | --- |
| [Printer](index.html) | Connect to an RPP02N, print a test message, or send a simple ESC/POS text receipt. |
| [GATT Explorer](gatt.html) | List accessible GATT services and characteristics, read values, subscribe to notifications/indications, and export JSON logs. |
| [Device Identity Inspector](identity.html) | Read standard identity characteristics such as device name, serial number, system ID, model, firmware and manufacturer **when available**, and export JSON. |

Built with plain HTML, CSS, and JavaScript. No build tools, backend, or native Android bridge are required.

## Quick start

1. Open **https://apogee.github.io/rpp02n-ble-pwa/** in a Web Bluetooth-capable browser, such as Chrome on supported Android devices.
2. Enable Bluetooth and power on the RPP02N printer.
3. Select **Connect** and choose the printer from the browser's Bluetooth device picker.
4. Select **Print TEST PWA**, or edit the receipt and choose **Print resit**.
5. Disconnect before using GATT Explorer or Device Identity Inspector with the same device.

Web Bluetooth requires **HTTPS or localhost**, compatible hardware, browser support, and user permission. Some Brave builds may require enabling Web Bluetooth in browser settings or flags.

## Printing protocol

The tested RPP02N path uses:

| Parameter | Value |
| --- | --- |
| GATT service | `0xFF00` |
| Write characteristic | `0xFF02` |
| Format | ESC/POS initialization (`ESC @`), UTF-8 text, and line feeds |
| Transmission | 20-byte chunks, 50 ms delay between chunks |

The app uses a supported GATT write method to transmit the data. A successful BLE write does not guarantee the receipt physically printed. Advanced ESC/POS functions and printer-specific character encodings are outside this proof of concept.

## GATT Explorer

The [GATT Explorer](gatt.html) displays UUIDs and properties for services and characteristics the browser is authorized to access. Use **Read** for characteristics advertising `read`, or **Subscribe** for characteristics supporting `notify` or `indicate`. **Export JSON** saves discovery results and event logs.

The explorer does not issue application-level characteristic writes. Only services requested or permitted by Web Bluetooth may be visible, so the results may not represent the device's complete GATT database.

## Device Identity Inspector

The [Device Identity Inspector](identity.html) attempts read-only access to standard characteristics:

| Service | Characteristics |
| --- | --- |
| Generic Access (`0x1800`) | Device Name (`0x2A00`), Appearance (`0x2A01`) |
| Device Information (`0x180A`) | System ID (`0x2A23`), Model (`0x2A24`), Serial Number (`0x2A25`), Firmware (`0x2A26`), Manufacturer (`0x2A29`) and other standard fields |

These characteristics are optional. On the tested RPP02N, Device Name was readable, but Device Information Service (`0x180A`) was not available.

**Web Bluetooth does not expose a raw Bluetooth MAC address.** Its `BluetoothDevice.id` is a browser-generated identifier, not a MAC address. Devices sharing the same name or GATT UUIDs cannot necessarily be distinguished without a unique readable identity value. The JBD-named filter is an inspection option, **not** a validated JBD BMS integration.

## Project structure

```text
index.html            Printer and receipt printing
gatt.html             GATT service and characteristic explorer
identity.html         Read-only device identity inspector
manifest.webmanifest  PWA manifest
sw.js                 Service worker and offline asset cache
icon.svg              App icon
README.md             Project documentation
```

## Run locally

```bash
git clone https://github.com/ApOgEE/rpp02n-ble-pwa.git
cd rpp02n-ble-pwa
python3 -m http.server 8000
```

Open **http://localhost:8000/** on the same machine in a compatible browser. For Android testing, use the HTTPS GitHub Pages deployment or another HTTPS environment; plain HTTP via a LAN IP is generally not a secure context.

To publish your fork, enable GitHub Pages for the repository root on the appropriate branch. Reload after deployment if the PWA service worker has cached an older version.

## Limitations and safety

- Web Bluetooth support, permissions, service visibility, and connection persistence depend on the browser and device.
- Navigating between pages or suspending the browser may disconnect GATT; reconnect when needed.
- GATT Explorer and Identity Inspector do not send characteristic write commands; notification subscriptions still use standard GATT subscription operations.
- The Printer page **does** intentionally send ESC/POS print data.
- Never send exploratory control writes to safety-critical equipment. Do not use this proof of concept to change settings on an operational off-grid BMS.
- Exported JSON may contain device identifiers or telemetry; review before sharing.

## Contributing

Issues and pull requests are welcome. Include your browser, operating system, BLE device model, and sanitized logs when reporting compatibility issues.

## License

This repository currently has no license file. Do not assume that its code is licensed for redistribution or reuse unless the maintainer adds an explicit license.

## Graphic print test

The Printer page includes a monochrome 384 × 360 pixel preview with a QR code linking to the hosted PWA, simple battery/power/Bluetooth icons, and a formatted heading. **Refresh preview** regenerates the bitmap and **Print graphic test** sends an ESC/POS `GS v 0` raster image through the existing `0xFF02` BLE characteristic using 20-byte chunks with 50 ms pacing. The QR matrix is embedded for the project's fixed GitHub Pages URL, so no external QR service is required. The graphic print path is experimental: support for `GS v 0` depends on the RPP02N firmware, and transmission can take approximately a minute. The preview is not a guarantee of printer output.

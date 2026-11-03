# RPP02N Web Bluetooth PWA

A lightweight single-page Progressive Web App (PWA) for **printing ESC/POS receipts directly to an RPP02N Bluetooth Low Energy (BLE) thermal printer**, exploring GATT services, and inspecting standard BLE device identity information.

**[Launch the PWA](https://apogee.github.io/rpp02n-ble-pwa/)** · **[Repository](https://github.com/ApOgEE/rpp02n-ble-pwa)**

> This is a browser-based proof of concept, not a production printer driver or a validated BMS monitoring application.

## Features

| Page | Function |
| --- | --- |
| [Printer](index.html#print) | Connect to an RPP02N, print a test message, or send a simple ESC/POS text receipt. |
| [GATT Explorer](index.html#gatt) | List accessible GATT services and characteristics, read values, subscribe to notifications/indications, and export JSON logs. |
| [Device Identity Inspector](index.html#identity) | Read standard identity characteristics such as device name, serial number, system ID, model, firmware and manufacturer **when available**, and export JSON. |\n| [Printer Diagnostics](index.html#diagnostics) | Read standard BLE Battery Level when available, list permitted GATT service/characteristic metadata, monitor FF01/FF03 notifications and export JSON without vendor writes. |

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
identity.html         Read-only device identity inspector\ndiagnostics.html      Read-only printer diagnostics
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

### Adjustable graphic print settings

Before printing, choose BLE delay (0/5/10/20/50 ms), bitmap height (160/240/360 px), and BLE chunk size (20/40/100/150/200/220/237 bytes) in **Graphic BLE settings**. Defaults are 50 ms, 360 px and 20 bytes. Changes are stored in browser localStorage. A transfer estimate is shown and actual transfer time and throughput are logged after sending. Chunk sizes above 20 bytes depend on negotiated BLE MTU and printer behavior; 100 and 237 bytes were tested successfully on the documented unit. The bitmap preview scales uniformly to keep QR modules square; smaller heights can reduce QR readability. Text receipt printing retains the original 20-byte / 50-ms pacing.

### MTU and 200-byte graphic benchmark

An nRF Connect Android session with the tested RPP02N reported **ATT MTU 240 bytes** when 517 was requested. A normal ATT Write therefore has a **theoretical maximum application payload of 237 bytes** (MTU minus 3). This measurement applies to that nRF Connect connection; Web Bluetooth does not expose negotiated MTU and may use a different value.

The graphic print UI now offers 150, 200, 220 and 237-byte chunks alongside the original 20, 40 and 100-byte options. These new sizes are experimental until tested with the PWA. Use **Benchmark 200 bytes · 0ms** to select **200-byte chunks, 0ms delay and 360px bitmap height**, then transmit one complete graphic test and log the measured transfer time and throughput. The existing **Print graphic test** button still uses the selected custom settings, and text printing remains at 20 bytes / 50ms.

A previously tested 100-byte / 0ms configuration transmitted 17,293 bytes in 173 writes in approximately 1.35–1.76 seconds across five consecutive visually complete prints. A successful GATT write does not guarantee the entire receipt was printed; inspect the physical output and verify the QR code after each benchmark. If the 200-byte transfer fails, return to the known working 100-byte setting.

## Verified 237-byte BLE graphic benchmark (2026-11-03)

The tested device is labelled **MP-58N** (58 mm paper, 5V/1A) and advertises as **RPP02N** over BLE. In a separate nRF Connect Android connection, a requested MTU of 517 resulted in **ATT MTU 240**, corresponding to a theoretical **237-byte ATT Write payload**. The PWA cannot directly inspect its negotiated MTU.

Five consecutive **384 × 360 px** graphic receipts were sent through Web Bluetooth using characteristic `0xFF02`, `writeWithoutResponse`, **237-byte chunks**, and **0ms added delay**. Each transfer was **17,293 bytes in 73 writes**.

| Print | Transfer time | Throughput |
| --- | ---: | ---: |
| 1 | 1.21 s | 13.93 KiB/s |
| 2 | 1.21 s | 13.95 KiB/s |
| 3 | 1.14 s | 14.75 KiB/s |
| 4 | 1.26 s | 13.42 KiB/s |
| 5 | 1.24 s | 13.61 KiB/s |
| **Average** | **1.21 s** | **13.93 KiB/s** |

**Observed physical result:** all five receipts printed completely and **all five QR codes scanned successfully**. This is a device/browser-specific observation, not a universal printer capability guarantee. The earlier tested 100-byte / 0ms configuration averaged approximately 1.47 s over five consecutive prints. The 237-byte setting is approximately **18% faster in measured BLE transfer time**. Neither metric includes mechanical print completion time.

**Recommended for this tested setup:** 237 bytes / 0ms for fast graphic printing, with 100 bytes / 0ms as a tested fallback and 20 bytes / 50ms as the conservative default. Other devices, connections, browsers or printer firmware may need smaller chunks.

## Printer Diagnostics (read-only)

The [Printer Diagnostics](diagnostics.html) page connects to an RPP02N with optional permission for standard **Battery Service `0x180F`** and printer service `0xFF00`. If Battery Level characteristic `0x2A19` exists, is readable, and returns a value in the 0–100 range, the page displays the percentage. Otherwise it explicitly displays **Unknown** and the reason in the diagnostic log.

The page also lists **permitted** GATT service UUIDs, characteristic UUIDs and properties without reading vendor-specific values, and can export a JSON snapshot and log. Web Bluetooth may not reveal all services. **Paper-out, charging and printer error states are shown as Unknown** until a verified response protocol is available; the page does not send ESC/POS status queries or perform vendor-specific writes. Notification subscriptions are optional and user-initiated. Disconnect other BLE clients before inspecting the printer.

### BLE Notification Monitor (FF01 / FF03)

On the [Printer Diagnostics](diagnostics.html) page, select **Subscribe FF01 + FF03** after connecting. The monitor uses the tested RPP02N service `0xFF00`, attempts GATT `startNotifications()` on `0xFF01` and `0xFF03` independently, and reports individual failures without stopping the other subscription. Each received packet is displayed and exported with an ISO-8601 timestamp, characteristic UUID label, byte count, uppercase HEX and Base64. Use **Unsubscribe** to call `stopNotifications()` and remove event handlers, or **Disconnect** to clear the session. **Clear notifications** clears captured packets but does not change the active subscriptions. **Export JSON** includes the notification records alongside the existing diagnostics and logs.

The monitor sends **no vendor-specific commands, ESC/POS status requests or characteristic value writes**. GATT notification subscription itself configures the client's CCCD, so it is not entirely passive. Notifications might not arrive until a vendor-defined trigger occurs; absence of packets does not prove the device lacks status reporting. Notification contents are **raw/undecoded** and must not be interpreted as battery percentage or paper status without a verified protocol. The monitor has not yet been validated on the physical printer.

## Single-page shared BLE connection (2026-11-03)

The PWA now has **Print**, **Diagnostics**, **GATT Explorer** and **Identity Inspector** tabs inside `index.html`. The BLE connection, write characteristic `FF02`, and `FF01`/`FF03` notification subscriptions live in the same JavaScript document; switching tabs does not reload the page or disconnect GATT. Previous standalone URLs redirect to their corresponding `index.html#...` tabs. A full page reload, browser suspension, Bluetooth disconnect, or closing the tab can still end the BLE connection.

1. **Connect** once in the shared connection header (select RPP02N).
2. Open **Diagnostics** and select **Subscribe FF01 + FF03**.
3. Switch to **Print**, select **237-byte chunks / 0ms** in Graphic BLE settings, and print a graphic receipt. The separate **Benchmark 200 bytes · 0ms** button still uses 200 bytes; for 237 bytes use **Print graphic test**.
4. Return to **Diagnostics** to see notifications captured during the print, with ISO timestamps, HEX and Base64. **Export JSON** includes the raw notification history and event timeline.
5. Use **Unsubscribe** or **Disconnect** to end monitoring.

The event timeline records `PRINT_START`, `PRINT_TRANSFER_COMPLETE` and `PRINT_ERROR` with ISO timestamps and transfer details. `PRINT_TRANSFER_COMPLETE` means the browser finished BLE writes, **not** that the physical printer completed printing. Notification payloads are not decoded into battery or paper status. The standard Battery Service is still absent on the tested unit.

**Scope:** GATT Explorer retains permitted-service discovery and safe READ operations; standard identity characteristics are inspected using the same printer connection. The previous generic GATT notification-subscription UI is replaced by the dedicated FF01/FF03 monitor; the previous identity device picker for non-printer/JBD devices is not included in this printer-focused single-connection workflow. No vendor-specific write commands were added. Device-side validation and regression testing are required before merging.

# RPP02N BLE PWA — printer + GATT explorer

Two pages: `index.html` (existing ESC/POS printing) and `gatt.html` (GATT inspection). Both pages have navigation links.

## Deploy to GitHub Pages

Copy the **contents** of this directory to the root of your existing `rpp02n-ble-pwa` Git repository, preserving the existing Git history. Commit and push. The site should serve `/rpp02n-ble-pwa/index.html` and `/rpp02n-ble-pwa/gatt.html`. Refresh after deployment to update the service-worker cache.

## GATT explorer

1. Disconnect the printer from the print page / nRF Connect.
2. Open GATT Explorer and tap Connect & Discover. Select RPP02N.
3. Explore accessible services and characteristics. Tap Read only on characteristics advertising READ; optionally Subscribe to NOTIFY/INDICATE.
4. Export JSON for analysis. **No characteristic writes are performed by the GATT page.**

`optionalServices` includes known RPP02N services `0x18F0`, `0xFF00`, `0xFEE7` and some standard services. Other services may be inaccessible unless explicitly authorized. Reading unknown characteristics may return errors or sensitive values; share exported logs selectively.

## Limitations

Web Bluetooth does not expose BLE MAC addresses. Browser device IDs are browser-origin scoped. Disconnect/reconnect between pages; a page navigation does not preserve its active GATT connection. BLE requires HTTPS or localhost and browser support (Brave may need its Web Bluetooth flag).

## Device Identity Inspector

Open `identity.html` from the navigation. This read-only tool requests access to standard Generic Access (`0x1800`) and optional Device Information (`0x180A`) GATT services. It attempts to read Device Name, Appearance, System ID, Model Number, Serial Number, firmware/hardware/software revisions, Manufacturer Name, regulatory data and PnP ID when supported, and exports a JSON report.

It never sends GATT writes or BMS control commands. Web Bluetooth browser IDs are not Bluetooth MAC addresses; a characteristic may be absent or not readable. In particular, no unique identifier is guaranteed for RPP02N or JBD BMS. Select the JBD filter only when testing near the BMS, and do not alter an operating off-grid power system.

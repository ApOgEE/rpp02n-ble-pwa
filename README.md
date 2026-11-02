# RPP02N BLE PWA test

Deploy all four files to the same HTTPS directory. Open `index.html` in Chrome Android, disconnect nRF Connect, then tap Connect and select RPP02N. Tap Print TEST PWA.

BLE GATT service 0xFF00, characteristic 0xFF02, confirmed previously using nRF Connect. This page sends ESC/POS text in 20-byte chunks with a 50ms delay. TextEncoder uses UTF-8; use ASCII for initial testing. BLE writes do not guarantee physical printing.

The PWA service worker caches the page for offline reuse after first load. Some Chrome/Android versions may require reselecting the device after reopening. Bluetooth cannot be used from an insecure HTTP origin other than localhost.

---
title: Bluetooth Device Not Detected
parent: Troubleshooting
nav_order: 6
---

# My bluetooth trainer is not detected

## Why Zwift can't use bluetooth directly

Zwift talks to bluetooth devices through the Windows Runtime bluetooth API, which Wine does not implement yet: scanning for
devices, writing to them and subscribing to their data are all stubs. Zwift's bluetooth library fails to load, and `Log.txt`
shows:

```text
ERROR: BLE_LOAD_FAIL; context="calling LoadLibrary", library="BleWin10Lib.dll", ...
Could not load the Windows BLE DLL BleWin10Lib_fallback.dll due to the following error -529697949
```

Setting [DISABLE_BLUETOOTH](../../configuration/options/#disable_bluetooth) to `0` does not change this. Use the Zwift
Companion app, or bridge the trainer to Zwift over the network as described below.

## Bridge the trainer over Direct Connect

Zwift can also reach trainers over the network with Wahoo's Direct Connect protocol, which works in the container.
`bin/zwift-dircon-bridge.py` connects to your trainer through the host's bluetooth stack (BlueZ) and offers it to Zwift as a
Direct Connect device. It works with trainers that support the standard Fitness Machine or Cycling Power bluetooth services,
including ERG mode and resistance control.

1. Run the Zwift container with host networking, so Zwift sees the bridge on the network:

   ```bash
   NETWORKING="host"
   ```

2. Wake your trainer by pedaling, make sure no other app (such as the Zwift Companion app) is connected to it, and start the
   bridge on the host. It needs [uv](https://docs.astral.sh/uv/), which installs its Python dependencies on first run:

   ```bash
   ./bin/zwift-dircon-bridge.py
   ```

   The bridge connects to the one trainer it finds nearby. If it finds several, it lists them; pass the address of yours:

   ```bash
   ./bin/zwift-dircon-bridge.py D6:F8:D9:7F:99:4B
   ```

3. Start Zwift and open the pairing screen. The trainer shows up under its bluetooth name for power, cadence and controllable.

Keep the bridge running while you ride. It reconnects on its own if the trainer drops, and Zwift reconnects to it. Pass `-v`
to log every message between Zwift and the trainer. If Zwift does not find the bridge, see
[Wi-Fi Device Not Detected](../wifi-devices/), since the same firewall rules apply.

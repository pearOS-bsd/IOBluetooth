# IOBluetooth

IOBluetooth (`/System/Library/Frameworks/IOBluetooth.framework`, public) and the daemon behind it (`bluetoothd`): controller power, discovery, pairing, connections, services.

**Status: planned. No code yet.** (The earlier Control Center had a Bluetooth panel with fake data only, `TOPBAR_FAKE_BT` in `apps/topbar/WifiPanel.m`.)

## Layer
Layer 2. Depends on: `Foundation`, `IOKit` (user side), `libxpc`; talks over XPC to `bluetoothd` (layer 4 daemon). Used by: ControlCenter (Bluetooth tile, page, menu-bar item), Settings, HID (`bthidd`), audio routing (CoreAudio, later).

## API the Control Center uses (`apps/controlcenter-cocoa/CCControls.m`)
- `IOBluetoothPreferenceGetControllerPowerState()` / `IOBluetoothPreferenceSetControllerPowerState(int)` (C functions, dlsym)
- `+[IOBluetoothDevice pairedDevices]`, `-name`, `-isConnected`, `-openConnection`, `-closeConnection`
**Not exercised: no Bluetooth controller on the Mac VM.** The real ControlCenter's entitlements for it: `com.apple.bluetooth.system`, `com.apple.bluetooth.internal`, `com.apple.security.device.bluetooth`, `com.apple.security.temporary-exception.shared-preference.read-write` for `com.apple.Bluetooth`; it also uses `com.apple.AudioAccessoryServices` and `com.apple.HearingModeService` (AirPods) over XPC (its startup trace).

## Plan (FreeBSD backend)
1. Controller: netgraph (`ng_ubt` USB, `ng_hci`, `ng_l2cap`, `ng_btsocket`); power = the `bluetooth` rc service / the `hci` node present (`ngctl list`, `service bluetooth onestart|onestop` as in `apps/controlcenter-cocoa/CCPlatform.c`, tested with fakes only), `hccontrol(8)` for inquiry and reading the controller.
2. `bluetoothd` in `d`: inquiry, pairing (`hcsecd(8)` keys), connection state, SDP; Mach service names as the real one (read with REA / trace on a Mac with Bluetooth).
3. Client classes (`IOBluetoothDevice`, `IOBluetoothHostController`, `IOBluetoothDeviceInquiry`, `IOBluetoothSDPServiceRecord`) over that daemon; the preference functions above first (smallest surface the Control Center needs).
4. HID and audio profiles later (`bthidd`, A2DP into CoreAudio).

## Existing code to reuse
`apps/controlcenter-cocoa/CCPlatform.c` (`ccp_bluetooth_*`).

## Rules for this repository
- Our code: BSD-2-Clause. Apple files are never copied here: reference material is read from the user's own Mac (`macdeps`), notes and measured values are written in our own words (`docs/`).
- No GPL/LGPL code. Upstream code, if any, only under `upstream/` with a `PROVENANCE.md`.
- Before writing a function: search the code index and `where-is.sh` (rule 3); do exactly what Apple does, with evidence recorded here (rule 4).
- Test on the FreeBSD VM; CI is the shared `hygiene` workflow (`pearOS-bsd/.github`).

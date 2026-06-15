---
name: remote-adb-tailscale
description: Connect Android or Samsung Galaxy devices to ADB through Tailscale and always finish by switching from Wireless Debugging bootstrap to TCP ADB on port 5555. Use when the user mentions Android ADB over Tailscale, Galaxy wireless debugging, remote ADB, mobile-data ADB, adb pair/connect/tcpip 5555, or keeping ADB reachable after Wi-Fi is turned off.
---

# Remote ADB Tailscale

## Overview

Use this skill to bootstrap Android Wireless Debugging over Tailscale, then switch the device to classic TCP ADB on port 5555 so it can remain reachable when the phone leaves Wi-Fi but stays online through Tailscale.

Wireless Debugging is only the bootstrap path. The final target is always `<tailscale-ip>:5555`.

## Completion Target

Always complete the full sequence:

1. Pair Wireless Debugging if needed.
2. Connect to `<tailscale-ip>:<wireless-debug-port>`.
3. Run `adb -s <tailscale-ip>:<wireless-debug-port> tcpip 5555`.
4. Connect to `<tailscale-ip>:5555`.
5. Verify shell execution through `<tailscale-ip>:5555`.

Do not report completion when only `<tailscale-ip>:<wireless-debug-port> device` works. That is only the bootstrap transport. Completion requires `<tailscale-ip>:5555 device` and a successful shell command through that transport.

If `<tailscale-ip>:5555` is `Connection refused`, keep working from the bootstrap transport: rerun `adb -s <tailscale-ip>:<wireless-debug-port> tcpip 5555`, then retry `adb connect <tailscale-ip>:5555`.

## Requirements

- Confirm `adb` and `tailscale` are installed on the host.
- Confirm Tailscale is running on both the host and Android device.
- Confirm the host and Android device are logged into the same Tailscale tailnet.
- Confirm Android Developer Options, USB debugging, and Wireless Debugging are enabled.
- If the host shell sandbox blocks the ADB server listener on port 5037, rerun the command using the active environment's permitted command path.

Use:

```bash
which adb
adb version
which tailscale
tailscale status
```

If `adb` is missing, install or guide installation before continuing. On macOS, prefer:

```bash
brew install android-platform-tools
```

If Homebrew is unavailable, network access is blocked, or the environment cannot install packages, tell the user to install Android SDK Platform Tools from the official Android developer site and ensure `adb` is on `PATH`. Do not continue until `adb version` works.

If `tailscale` is missing, install or guide installation before continuing. On macOS, use the official Tailscale app or a package-manager install. Confirm the user is logged in before relying on `tailscale status`.

If installing software is not possible in the active environment, stop the connection attempt and give the exact install commands/manual steps the user must run.

## Verify Same Tailnet

Before attempting ADB, verify both devices are in the same tailnet:

```bash
tailscale status
tailscale ip -4
```

The Android device should appear in `tailscale status` with a Tailscale IP, hostname, OS `android`, and the same account/tailnet identity as the host. If it does not appear:

- Ask the user to open the Tailscale app on Android.
- Confirm Tailscale is connected, not paused, and logged into the same account/tailnet as the desktop.
- Ask the user for the Android Tailscale hostname or 100.x.y.z IP if the status list is large.
- Retry `tailscale status` and `tailscale ping -c 3 <android-hostname-or-ip>`.

Do not diagnose ADB until Tailscale reachability is proven. Tailscale reachability is proven only when the device appears in `tailscale status` and `tailscale ping` returns `pong`.

## Find The Device

Locate the Android device in the tailnet:

```bash
tailscale status
tailscale ping -c 3 <android-hostname>
```

Record the Tailscale IP, for example `100.88.77.66`. Use the Tailscale IP for remote commands, not the local Wi-Fi IP shown by Android or mDNS.

If the device cannot be found or is offline in Tailscale, fix Tailscale first. If Tailscale is reachable but ADB cannot connect, move to Wireless Debugging pairing.

## Pair Wireless Debugging

Ask the user to open:

```text
Developer options -> Wireless debugging -> Pair device with pairing code
```

They must provide the pairing code and the pairing port. The IP displayed on the phone may be a local Wi-Fi IP; use the port with the Tailscale IP.

Use this pairing flow whenever:

- The desktop has never paired with this Android device.
- `adb connect` reports a certificate/authentication error.
- The device is reachable through Tailscale but does not appear in `adb devices`.
- The current Wireless Debugging ports expired or changed.
- The user says this is the first attempt with the skill.

Check mDNS when available:

```bash
adb mdns services
```

Typical output:

```text
adb-DEVICEID  _adb-tls-pairing._tcp  192.168.x.x:40343
adb-DEVICEID  _adb-tls-connect._tcp  192.168.x.x:34981
```

Pair through Tailscale:

```bash
adb pair <tailscale-ip>:<pairing-port> <pairing-code>
```

Example:

```bash
adb pair 100.88.77.66:40343 199205
```

Expected success:

```text
Successfully paired to 100.88.77.66:40343
```

Pairing ports and codes expire quickly. If pairing fails with a protocol fault, read error, or connection refused, ask for a new pairing dialog/code and retry with the new port.

## Connect Wireless Debugging Over Tailscale

Find the `_adb-tls-connect` port:

```bash
adb mdns services
```

Connect with the Tailscale IP and the connect port:

```bash
adb connect <tailscale-ip>:<connect-port>
adb devices -l
adb -s <tailscale-ip>:<connect-port> shell getprop ro.product.model
```

Example:

```bash
adb connect 100.88.77.66:34981
adb -s 100.88.77.66:34981 shell getprop ro.product.model
```

If `adb devices -l` shows both an mDNS transport and a Tailscale-IP transport, target the Tailscale-IP transport with `-s` to avoid ambiguity.

## Switch To Port 5555

Wireless Debugging often closes when Wi-Fi is turned off, even if Tailscale remains active on mobile data. After establishing Wireless Debugging over Tailscale, always switch to TCP ADB:

```bash
adb -s <tailscale-ip>:<wireless-connect-port> tcpip 5555
adb connect <tailscale-ip>:5555
```

If `adb connect <tailscale-ip>:5555` reports `Connection refused`, TCP ADB is not enabled. Do not stop at this point. Reconfirm that the Wireless Debugging bootstrap transport is still `device`, rerun `adb -s <tailscale-ip>:<wireless-connect-port> tcpip 5555`, then retry `adb connect <tailscale-ip>:5555`.

Verify the new transport:

```bash
adb devices -l
adb -s <tailscale-ip>:5555 shell getprop ro.product.model
```

Expected final state:

```text
<tailscale-ip>:5555  device  model:<model>
```

After `tcpip 5555`, the old Wireless Debugging transport may become `offline`. Remove stale transports:

```bash
adb disconnect <tailscale-ip>:<old-wireless-port>
adb devices -l
```

## Final Tailscale Admin Guidance

After `<tailscale-ip>:5555` is connected and verified, include a final note telling the user to restrict access in Tailscale Admin:

- Open the Tailscale Admin Console.
- Go to Access controls / ACLs.
- Limit all Tailscale connectivity to the Android device so only the intended desktop account, desktop device, or tagged admin device can reach it.
- Do not recommend a port-only ACL for `5555`; restrict the Android node as a whole so other Tailscale services on that device are not accidentally exposed.
- Save/apply the ACL and then re-test `adb -s <tailscale-ip>:5555 shell getprop ro.product.model`.

Give this guidance after the ADB connection is complete. Do not postpone `adb tcpip 5555` for this guidance.

## Verify After Wi-Fi Is Off

After the user turns off Wi-Fi on the phone, verify three layers:

```bash
tailscale status
tailscale ping -c 3 <android-hostname>
adb devices -l
adb -s <tailscale-ip>:5555 shell getprop ro.product.model
```

Success means:

- Tailscale reports the Android device as active.
- `tailscale ping` returns `pong`.
- `adb devices -l` shows `<tailscale-ip>:5555 device`.
- ADB shell commands return real output, such as the product model.

Do not claim success from `tailscale ping` alone. Tailscale reachability and ADB reachability are separate.

## Troubleshooting

Use `nc` to separate network reachability from ADB authentication:

```bash
nc -vz -w 3 <tailscale-ip> <port>
```

If `adb connect` reports `SSLV3_ALERT_CERTIFICATE_UNKNOWN`, the host is not paired with the phone. Run `adb pair` with a fresh pairing code.

If `adb connect <tailscale-ip>:5555` reports `Connection refused`, return to the bootstrap transport and rerun `adb -s <tailscale-ip>:<wireless-connect-port> tcpip 5555`.

If a Wireless Debugging pairing/connect port reports `Connection refused`, the phone likely rotated or closed that Wireless Debugging port. Refresh Wireless Debugging, re-run `adb mdns services`, and use the current pairing/connect port.

If the device is `offline`, remove stale transports and reconnect:

```bash
adb disconnect <ip>:<port>
adb connect <ip>:<port>
adb devices -l
```

If `adb pair` fails after the user supplied a code, assume the code expired or the pairing dialog closed. Ask for the new pairing code and port.

If `adb -s <tailscale-ip>:5555 shell ...` returns `error: closed`, immediately re-check:

```bash
adb devices -l
nc -vz -w 3 <tailscale-ip> 5555
adb connect <tailscale-ip>:5555
```

## Disable TCP ADB

To switch the device away from TCP mode when connected:

```bash
adb -s <tailscale-ip>:5555 usb
```

This usually disconnects the remote session.

## Example

```bash
tailscale status
tailscale ping -c 3 galaxy-a36-5g

adb mdns services
adb pair 100.88.77.66:40343 199205
adb connect 100.88.77.66:34981
adb -s 100.88.77.66:34981 shell getprop ro.product.model

adb -s 100.88.77.66:34981 tcpip 5555
adb connect 100.88.77.66:5555
adb -s 100.88.77.66:5555 shell getprop ro.product.model

adb devices -l
```

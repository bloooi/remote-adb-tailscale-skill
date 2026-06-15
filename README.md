# Remote ADB Tailscale Skill

Codex skill for connecting to an Android device with remote ADB over Tailscale.

This skill guides the full flow from Android Wireless Debugging bootstrap to stable TCP ADB on port `5555` through the device's Tailscale IP.

## What This Is

`remote-adb-tailscale` is for Android remote ADB connections where the Android device is reachable through a Tailscale tailnet.

The intended final connection target is:

```text
<android-tailscale-ip>:5555
```

Wireless Debugging is used only as the bootstrap step. After pairing and connecting through the temporary Wireless Debugging port, the skill switches ADB to TCP mode on port `5555` and verifies shell execution through that final transport.

## Requirements

- Android device with Developer Options enabled.
- USB debugging and Wireless Debugging enabled on the Android device.
- `adb` installed on the host computer.
- Tailscale installed and running on the host computer.
- Tailscale installed and running on the Android device.
- The host computer and Android device must be added to the same Tailscale account or tailnet.

The Android device must have the Tailscale app installed. This only works when the Android device is connected to the same Tailscale account or tailnet as the host that will run ADB.

## Security Notes

This workflow uses your Tailscale tailnet. Do not treat port `5555` as harmless just because it is not exposed to the public internet.

Before relying on this setup, open the Tailscale Admin Console and review Access controls / ACLs. If the Android device is reachable by everyone in the tailnet, change the ACL rules so only the intended account, desktop device, or trusted admin-tagged device can access it.

In other words:

- Do not leave the Android device open to every user or device in the tailnet.
- Restrict access at the Tailscale ACL level to the specific account or device that should use remote ADB.
- Prefer restricting the Android node as a whole instead of allowing broad access and only thinking about port `5555`.
- After changing ACLs, re-test `adb -s <android-tailscale-ip>:5555 shell getprop ro.product.model`.

## Repository Contents

```text
SKILL.md
agents/openai.yaml
README.md
LICENSE
```

## License

MIT License. See [LICENSE](LICENSE).

# deeyo releases

Firmware images for the deeyo card reader, and the manifest that points at them.

This repository is public for one reason: the televisions that fetch from it hold
no credentials, and should not. A token on an appliance in someone's house is a
token that leaks, expires, or has to be rotated across devices nobody can reach.

## How an update travels

```
here  --(the television fetches, over the internet)-->  television
                                     television  --(the reader pulls, over the LAN)-->  reader
```

The television is in the middle because it already has an internet connection, is
already discoverable on the local network, and already serves HTTP. The reader
needs none of those things.

The reader checks at boot and every six hours. A television fetches this manifest
when its app starts, so a release reaches a device nobody has restarted only when
someone asks it to look:

```bash
curl -X POST http://<tv>:8080/firmware/refresh
```

## Why a public binary is not a loose one

Every image here is signed, and the reader's bootloader verifies that signature
before booting the new slot. The public half of the key is baked into the
firmware already on the device; the private half is not in this repository and
never will be. A corrupted download, a hostile mirror, or something on the local
network impersonating the television all fail closed at the reader.

A new image is also on trial when it first boots: it is discarded on the next
restart unless it cancels the rollback, and it only does that after reaching a
television — not merely after booting. The failure worth protecting against is
not a crash, which announces itself. It is firmware that runs perfectly and
cannot talk to anything.

## Versions

Compared for **difference**, not order. That is deliberate: it makes downgrading
work. Serve an older image here and every reader walks back to it.

| Version | Notes |
|---|---|
| 0.7.2 | Hands the radio back to WiFi: coexistence prefers WiFi, and the keyboard connection asks the host for an 80-120 ms interval. Before it, a reader on its own LAN measured 250 ms median ping and an eleven-minute update |
| 0.7.1 | **Fixes 0.7.0 for readers updated over the air.** 0.7.0 renamed the NVS namespace credentials live under, so an upgraded reader booted with no WiFi and sat in its setup portal. 0.7.1 carries the old namespace forward, and an image on trial that cannot reach the television now restarts itself so the rollback actually happens |
| 0.7.0 | Accepts both the old and new service names, in both mDNS and the health check. **Do not serve this to a reader that was ever on 0.6.0** |

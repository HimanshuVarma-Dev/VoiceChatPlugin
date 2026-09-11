# VoiceChat Client (Fabric)

Proximity voice chat client mod for Minecraft Java Edition (Fabric), built to
interoperate with the VoiceChat Paper server plugin.

- **Transports:** UDP primary, TCP fallback, automatic reconnect with capped backoff
- **Audio:** Opus (preferred) with in-band FEC + PLC, ADPCM/PCM graceful downgrade
- **Auth:** per-session secret over `voicechat:auth` plugin messaging, HMAC-SHA256
  challenge/response, mutual server proof, AES-128-GCM voice encryption
- **Voice pipeline:** mic → preprocess → VAD → Opus → encrypt → net → decrypt →
  decode → jitter buffer → spatializer → mixer → output
- **Game integration:** proximity attenuation, 3D directional audio, per-player
  volume/mute, PTT / voice-activity / always-on, HUD, diagnostics screen,
  `/voicechat` client command
- **Android first-class:** PojavLauncher/ARM64 audio backend (reflection-only,
  no native code), Bluetooth SCO routing, power profiles
- **Version independence:** the voice protocol is versioned separately from
  Minecraft — a 1.21 client voices with a 1.21.1 server (e.g. via ViaVersion)

## Modules

| Module     | Contents |
|------------|----------|
| `:opus`    | Vendored Concentus (pure-Java Opus), no native libraries |
| `:protocol`| Wire codecs: packets, handshake, auth, capabilities, sequencing |
| `:core`    | Engine: audio, DSP, VAD, codecs, crypto, net, pipelines, config, diagnostics (zero Minecraft dependencies) |
| `:fabric`  | Thin glue: adapter, plugin channel, keybinds, HUD, screens, command |

All Minecraft-version-specific code lives in `:fabric` (see
`dev.voicechat.fabric.compat`), behind the `MinecraftAdapter` interface. CI
compiles the mod against both 1.21 and 1.21.1.

## Building

Requirements: JDK 21+ and network access to Maven Central / Fabric Maven.

```sh
./gradlew build            # compile + all tests + remapped mod jar
./gradlew :fabric:remapJar # mod jar only -> fabric/build/libs/
```

The release jar (`fabric/build/libs/voicechat-client`-style `fabric-*.jar`) is
self-contained: `:core`, `:protocol` and `:opus` are embedded (Jar-in-Jar).
Builds are reproducible (`reproducibleFileOrder`, no timestamps).

Run all tests:

```sh
./gradlew :protocol:test :opus:test :core:test
```

`:core` tests include network-simulation coverage (loss/duplication/reorder
proxies, handshake and reconnect integration tests).

## Installing

1. Install Fabric Loader ≥ 0.16 + Fabric API on Minecraft 1.21.x.
2. Drop the mod jar into `mods/`.
3. Join a server running the VoiceChat Paper plugin — voice connects
   automatically once the server sends the auth secret.

## Defaults

| Action | Key |
|--------|-----|
| Push to talk | `V` |
| Mute microphone | `M` |
| Deafen | `N` |
| Voice settings | `G` |
| Diagnostics screen | `F8` |

Settings live in `config/voicechat-client.json` and apply live. The
`/voicechat` command offers `config`, `mute`, `deafen`, `debug`, `status`
and `reconnect`.

## Protocol

Wire details for plugin interop are in [docs/PROTOCOL.md](docs/PROTOCOL.md).

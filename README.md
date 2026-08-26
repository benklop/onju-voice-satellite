# Onju Voice satellite

ESPHome firmware that turns an [Onju Voice](https://github.com/justLV/onju-voice) PCB (a drop-in replacement board for a 2nd generation Google Nest Mini) into a Home Assistant voice satellite.

This repository is a maintenance fork of [tetele/onju-voice-satellite](https://github.com/tetele/onju-voice-satellite). The product layer is [Idskov/onju-voice-esphome v1.5.1](https://github.com/Idskov/onju-voice-esphome); duplex I2S is [n-IA-hane/esphome-audio-stack](https://github.com/n-IA-hane/esphome-audio-stack).

## Status

| Now | Next |
|-----|------|
| Dual-mic AFE + I2S keep-alive on the shared bus | Loud-music barge-in / VAD / AGC — software AFE is not XMOS |

One `esp_audio_stack` owns the shared I2S bus so both SPH0645s and the amp run together. ESP-SR dual-mic AFE (`mic_num: 2`, Speech Enhancement) feeds wake word and Assist. Loud music can still mask the wake word — tap the center pad if it misses. Overlay `rx_mic_slots: [right, left]` if BSS sounds inverted.

Guides for alarms, wake words, music, and touch controls: **[User Manual](MANUAL.md)**.

## Requirements

- Onju Voice PCB installed in a Google Nest Mini (2nd gen)
- Home Assistant 2024.7.0 or newer, with a voice assistant pipeline (STT + TTS)
- ESPHome **2026.5.0** or newer
- First flash over USB (BOOT button held). Later updates are OTA.

If you are upgrading from tetele's Arduino-based config, OTA will likely fail because the framework is now ESP-IDF. USB flash once, OTA after that.

[PCB transplant video](https://youtu.be/VaQkc-sgc04) · [Nest Mini disassembly](https://www.ifixit.com/Teardown/Google+Nest+Mini+(2nd+generation)+Teardown/130974)

## Install

### ESPHome Dashboard

```yaml
substitutions:
  name: my-onju-voice
  friendly_name: Living Room Voice

packages:
  onju_voice: github://benklop/onju-voice-satellite/esphome/onju-voice.yaml@main

esphome:
  name: ${name}
  name_add_mac_suffix: false
  friendly_name: ${friendly_name}

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

api:
  encryption:
    key: !secret api_encryption_key

ota:
  - platform: esphome
    password: !secret ota_password
```

After Wi-Fi is up, adopt the device in ESPHome / Home Assistant as usual.

### First USB flash

1. Hold **BOOT** on the Onju PCB and plug in USB **before** reassembling the Mini.
2. Flash from [web.esphome.io](https://web.esphome.io) or the ESPHome dashboard.
3. The device raises a Wi-Fi AP named "Onju Voice" for provisioning.

## Controls

| Control | Action |
|---------|--------|
| Left touch | Volume down (hold to repeat) |
| Right touch | Volume up (hold to repeat) |
| Center touch | Push-to-talk / dismiss timer / stop music |
| Mute switch | Toggle wake word listening |

## License

The PCB is **not** this repo and has [its own license](https://github.com/justLV/onju-voice/blob/master/LICENSE). This firmware is MIT. See [`LICENSE`](LICENSE).

Notification sounds in `res/` are [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) (UNIVERSFIELD).

## Credits

- [Justin Alvey](https://github.com/justLV) — Onju Voice PCB
- [Tudor Sandu](https://github.com/tetele) — original ESPHome satellite config
- [Nico Idskov](https://github.com/Idskov) — native ESPHome rewrite this tree is based on
- [Roy Meissner](https://github.com/rmeissn) — proven Onju duplex + software AEC recipe this tree ports
- [n-IA-hane](https://github.com/n-IA-hane/esphome-audio-stack) — `esp_audio_stack` / `esp_afe` backend
- [Kevin Ahrendt](https://github.com/kahrendt) — microWakeWord
- Home Assistant / ESPHome voice teams

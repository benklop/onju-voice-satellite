# Testing

## Automated Tests

The `tests/` directory contains Python simulations of the firmware's state machines. These catch regressions in LED logic, state transitions, and duplex wake-word behavior without needing hardware.

```bash
pip install pytest
pytest tests/ -v
```

### What's covered

| Test file | What it models |
|-----------|----------------|
| `test_led_states.py` | LED output for all 1,536 state combinations, priority order (volume > alarm clock > timer alarm > voice pipeline > playing > announcing > timer countdown > wake word > off), full pipeline sequences, volume overlay behavior |
| `test_state_transitions.py` | Media player + voice assistant state machines, wake word start/stop logic, duplex (MWW stays running during playback), center touch behavior per state |

### When to run

- Before every release
- After any change to `reset_led`, media_player triggers, voice_assistant triggers, or center touch logic
- When adding new LED states or device states

### Adding tests

If you add a new feature, model the state transitions and expected LED output in the test files. The simulators mirror the firmware's YAML logic — keep them in sync.

## Manual Verification

Automated tests verify logic correctness but cannot confirm physical behavior (LED colors, audio output, touch responsiveness). Use this checklist after flashing to confirm hardware works as expected.

### Prerequisites

- Device flashed and connected to Home Assistant
- Voice assistant pipeline configured (STT + conversation + TTS)
- Media player entity visible in HA

### Checklist

**Idle state**
- [ ] Purple gentle twinkle when wake word is active
- [ ] LED off when muted or wake word disabled

**Voice pipeline** (say wake word, ask a question)
- [ ] Purple → white (listening) → blue (processing) → green (speaking) → purple

**Push-to-talk** (center touch, don't speak, wait 15s)
- [ ] White listening LED → times out → purple

**Media playback** (send music URL via `media_player.play_media`)
- [ ] Teal flicker while playing
- [ ] Audio from speaker

**Volume during playback** (touch vol+/vol- while music plays)
- [ ] White volume bar shown during touch
- [ ] Returns to teal after 2s (not purple)

**Center touch stops music**
- [ ] Music stops, LED returns to purple

**Voice interrupts music** (wake word during playback)
- [ ] Wake word is heard while music plays (no center tap)
- [ ] Music pauses → voice pipeline runs → music resumes
- [ ] LED: teal → white → blue → green → teal
- [ ] If wake word misses during loud music, center tap still starts listening

**Mute switch**
- [ ] ON: LED off, wake word disabled
- [ ] OFF: purple twinkle returns

**Music Playback Light toggle** (switch in HA)
- [ ] OFF: LED off while music plays
- [ ] ON: teal flicker returns

**Adopt flow** (new device only)
- [ ] Captive portal creates AP
- [ ] ESPHome dashboard discovers and adopts device
- [ ] All features work after adopt

### LED Reference

| State | Color | Effect |
|-------|-------|--------|
| Boot (no WiFi) | Orange | Slow pulse |
| Boot (WiFi, no HA) | Green | Pulse |
| Idle (WW active) | Purple | Gentle twinkle |
| Idle (WW off/muted) | Off | — |
| Listening | White | Twinkle |
| Processing | Blue | Scan |
| Speaking (TTS) | Green | Flicker |
| Playing (media) | Teal | Flicker |
| Volume adjust | White | Proportional bar |

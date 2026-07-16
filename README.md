# VR Doctor Fish

**Experience a relaxing, virtual fish therapy from the comfort of your home.**

VR Doctor Fish recreates a doctor fish foot spa as a multi-sensory VR experience. The user sits with their feet in a real bucket of water while a Meta Quest 3 renders a virtual pool around their legs. Immersive visuals, spatial audio and localised vibrotactile feedback from VibraForge actuators worn on the legs combine to simulate gentle water flow, small fish nibbling, a big fish bite and jellyfish stings, before the water calms again and the session ends in relaxation.

## Project context

Developed at the **IVE 2026 Winter School: Haptics x XR**
13 – 17 July 2026 | Mawson Lakes, Adelaide University, Kaurna Land

## Team

- Mimi Yoshii-Podger
- Cerella YueRu Chen
- Anneysha Sarkar

## The experience

Real-world setup: sit on a chair, remove your shoes, put your feet into a real bucket of water and put on the Meta Quest 3. The experience then moves through a staged sequence, with sight, sound and touch coordinated at every stage.

| Stage | You see | You hear | You feel |
| --- | --- | --- | --- |
| 1. Welcome | Clear water flowing around your legs | Gentle water flow and soft music | A gentle wave of vibration rising slowly along both legs |
| 2. Small fish | Small fish swim toward your feet | Tiny nibbling sounds | Soft, ticklish vibrations at random points on your legs |
| 3. Big fish | A big fish approaches and bites | A strong splash and water movement | A strong, quick vibration |
| 4. Jellyfish | Jellyfish glide by your legs | Whooshing and electric sounds | Sharp, high-frequency vibrations like tiny stings |
| 5. Calm | The water becomes calm again | Gentle water flow and relaxing music | Vibrations fade away |

At the end the user takes off the headset and lifts their feet out of the bucket, ideally with a relaxed body and a happy mind.

## System architecture

The Unity application on the host computer runs the VR environment (virtual body, bucket and water, small fish, large fish, jellyfish, lighting and effects) and an **experience state manager** that steps through the stages above:

```
Water (gentle flow)
  -> Small fish  (low frequency, long duration)
  -> Large fish  (high frequency, high amplitude, short duration)
  -> Jellyfish   (very high frequency, high amplitude, short duration)
  -> Ending      (relaxation)
```

Each state drives three controllers in parallel:

- **Visual controller** – water effects, fish animations, jellyfish effects, scene transitions
- **Audio controller** – water flow, fish, jellyfish and background music (see [`audio/`](audio/README.md))
- **Haptic controller** – one vibration pattern per state, sent through the VibraForge Unity API plugin as `SendCommand(address, start_stop, intensity, frequency)` calls (see [`haptics/`](haptics/README.md))

### Haptics hardware pipeline

```
Unity (host computer)
  -> BLE -> ESP32 control unit   (receives and validates commands, converts to UART, manages chains)
  -> UART -> 2 chains x 5 vibration units
  -> PIC16F18313 MCU per unit    (receives UART command, drives H-bridge, controls actuator)
  -> DRV8837 driver + LRA/VCA actuator on the user's legs
```

The headset provides head tracking, the VR display and spatial audio; all haptic output runs through the [VibraForge](https://github.com/pokemon9757/VibraForge) toolkit.

### Haptic design

Two actuator types are combined with contrasting textures so each creature feels distinct:

- **Weak actuators, low intensity, high frequency** – the ticklish nibble of small fish
- **Strong actuators, high intensity, low frequency** – the big fish bite
- **Strong actuators, very high frequency, short bursts** – jellyfish stings

The 10-second welcome pattern sweeps a continuous sensation down the leg through five actuator nodes with linear crossfades between neighbours. The full node timeline, pattern files and generator are documented in [`haptics/README.md`](haptics/README.md).

## Repository structure

| Folder | Contents |
| --- | --- |
| `audio/` | Background music and sound effects, with usage notes per file |
| `haptics/` | VibraForge vibration patterns (JSON command files), the welcome-experience CSV spec and the Python generator |

The Unity project has not been pushed yet; the pattern files are authored so they can move into the project unchanged.

## Credits

- Sound effects: CC0, sourced via Pixabay and edited pitch via Audacity
- Background music: Sound motifs were generated via ACE Music and Gemini, edited and composed via GarageBand
- Haptics hardware and Unity plugin: [VibraForge](https://github.com/pokemon9757/VibraForge)

## License

MIT – see [LICENSE](LICENSE).

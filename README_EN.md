# MIDIShip Bridge

MIDIShip Bridge 0.5.0 —
a VST3 wrapper instrument for Cubase that loads
another VST3 instrument inside itself and links its MIDI CCs to the four
motorized touch faders of the MIDIShip M in HUI mode.

The plugin is designed for `Instrument Track`. Notes, pitch bend, and other MIDI data
pass through to the hosted Kontakt, Omnisphere, or other VST3i, while four selected
CCs are simultaneously exposed as Bridge parameters. Cubase MIDI Remote converts
them into HUI for the motors. In the opposite direction, fader movement via a separate
loopMIDI port becomes ordinary MIDI CC again, which Cubase records into the
MIDI part. No MIDI Send on the Instrument Track is required.

Before first testing, save a copy of your Cubase project.

## Current version

- Name `MIDIShip Bridge`, vendor `Andrey Zubets`, manufacturer code `AnZu`,
  plug‑in code `MSB1`, bundle id `ru.andreyzubets.midishipbridge`.
- Compact interface with four real strips of the MIDIShip M and the factory
  preset `CC1 / CC2 / CC11 / CC21`.
- VST3 browser with search string, list refresh, and manual file or bundle selection.
- Up to 12 favourite instruments, drag‑and‑drop reordering, and global
  import/export of Favourites and custom CC presets.
- The instrument selected in the browser or from favourites loads immediately.
- The `Auto-open instrument` option opens the native editor of the child VST3
  alongside the compact Bridge window by default. The Bridge window remains
  available for changing the instrument, favourites, and CCs; a closed editor can
  be reopened with the `Open instrument` button.
- For Kontakt 8.11, a process‑wide cache of VST3 descriptions has been added. The first Bridge
  performs a slow flat‑module scan once, and subsequent instances with the same path
  are created from the saved `PluginDescription` without re‑scanning via
  `findAllTypesForFile` when Kontakt is already running.
- The Inno Setup 6 installer project preserves the updatable MIDIShip Bridge and its
  current MIDI Remote profile. The installer does not remove obsolete user MIDI Remote profiles.

Since the VST3 ID changed in the 0.2 branch, Cubase treats MIDIShip Bridge as a new instrument. Instances
of the old `HUI CC Bridge` in existing projects are not replaced automatically.
Before migrating, save the old project and transfer tracks deliberately.

## Workflow diagram

```mermaid
flowchart LR
    MIDI["MIDI notes + CC<br/>Instrument Track"] --> Bridge["MIDIShip Bridge VST3"]
    Bridge --> Child["Hosted VST3i<br/>Kontakt / Omnisphere"]
    Child --> Audio["Stereo audio to Cubase"]
    Bridge -->|"4 CC mirror values<br/>4 assignments"| Remote["Cubase MIDI Remote"]
    Remote -->|"HUI Output"| Surface["MIDIShip M<br/>4 motorized touch faders"]
    Surface -->|"HUI Input + touch"| Remote
    Remote -->|"CC Loopback Output"| Loop["loopMIDI"]
    Loop --> MIDI

Bridge runs inside the Cubase process: the host sees it as an instrument, and Bridge
creates an instance of the child VST3i. This is not a separate process and does not provide crash isolation.

The physical HUI path and the virtual CC loopback must remain separate.
For the MIDIShip M, Windows shows the same name TinyUSB Device in both input and output lists.
This is normal: select it for both HUI Input and HUI Output; the direction is determined by the logical port itself. CC Loopback Output
must be assigned to a separate loopMIDI port only.

The physical TinyUSB Device must be excluded from All MIDI Inputs. In HUI mode
the device sends control‑change and heartbeat messages; without exclusion they may
reach the Instrument Track directly and interfere with the CC loopback.
How the active instance is selected

HUI serves only one Bridge. Transmission is active when the MIDI Remote profile is loaded and simultaneously:

    an Instrument Track with MIDIShip Bridge is selected;

    Record Enable is active on the selected track.

With ten instances, the motors follow the selected and armed track. Other Bridges continue to play their instruments but do not send values to
the physical surface. It is advisable to keep only the selected track armed, listening to the common loopMIDI port.
Quick start

The full up‑to‑date setup is described in the
docs/MIDIShip_Bridge_User_Manual_EN.pdf.

Briefly:

    Install MIDIShip Bridge.vst3 and the Cubase MIDI Remote profile.

    Create a loopMIDI port, e.g. MIDIShip Bridge CC.

    In MIDI Remote assign TinyUSB Device to HUI Input and HUI Output, and
    MIDIShip Bridge CC to CC Loopback Output.

    Exclude TinyUSB Device from All MIDI Inputs.

    Create an Instrument Track with MIDIShip Bridge; its window opens
    automatically. Click the green + tile, find and select the VST3i.

    Select the track, enable Record Enable, and route its MIDI input to
    loopMIDI or to a correctly configured All MIDI Inputs.

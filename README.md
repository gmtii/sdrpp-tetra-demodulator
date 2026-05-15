## New features
 
### Per-timeslot audio filter
 
TETRA carriers use TDMA with 4 timeslots per channel. By default the
plugin decodes and plays audio from all timeslots simultaneously. The
timeslot filter lets you isolate a single timeslot so that only its
voice traffic is sent to the audio output.
 
In the OSMO-TETRA decoder panel there are two separate rows:
 
- **Audio TS** - four checkboxes (TS1-TS4). Tick one to restrict
  audio output to that timeslot only. Ticking the active checkbox
  again deselects it and returns to all-timeslot mode.
- **Timeslots** - colour-coded status indicators showing the current
  content of each slot in real time (UL / DATA / NDB / SYNC / VOICE).
  These are read-only and update independently of the selector above.
 
The selected timeslot is saved in tetra_demodulator_config.json and
restored on next launch.
 
### Encrypted traffic muting
 
When a cell has air encryption enabled and no decryption key is
available, the plugin silently discards encrypted voice blocks instead
of playing garbled noise. An orange indicator "MUTED: encrypted
traffic (no key)" appears in the panel when this condition is active.
 
If a keystore file is loaded and decryption succeeds, audio plays
normally and the indicator disappears.
 
### Signal constellation toggle
 
A "Show constellation" checkbox allows hiding the constellation
diagram to save CPU on slower machines such as Raspberry Pi 4.
When hidden the constellation sink handler skips the buffer copy
entirely. State is persisted in tetra_demodulator_config.json.
 
### ARM64 portability fix
 
All new fields added to tetra_mac_state are declared as int rather
than bool. Using bool in a C struct causes different memory padding
on ARM64 vs x86, which corrupts subsequent struct fields and breaks
audio output on Raspberry Pi 4. Declaring fields as int guarantees
consistent 4-byte alignment across all architectures.
 
Tested on x86_64 and ARM64 (Raspberry Pi 4).
 

### Original readme.md

## Tetra demodulator plugin for SDR++

Designed to fully demodulate and decode TETRA downlink signals

Thanks to osmo-tetra authors for their great library

Signal chain:

VFO->Demodulator(AGC->FLL->RRC->Maximum Likelihood(y[n]y'[n]) timing recovery->Costas loop)->Constellation diagram->Symbol extractor->Differential decoder->Bits unpacker->Osmo-tetra decoder->Sink

Binary installing:

Visit the Actions page, find latest commit build artifacts, download tetra_demodulator.so and put it to /usr/lib/sdrpp/plugins/, skipping to the step 4. Don't forget to install libtalloc!

Building:

  0.  If you have arch-like system, just install package sdrpp-tetra-demodulator-git with all dependencies.

      OR 

  1.  Install SDR++ core headers to /usr/include/sdrpp_core/, if not installed. Refer to https://cropinghigh.github.io/sdrpp-moduledb/headerguide.html about how to do that

      OR if you don't want to use my header system, add -DSDRPP_MODULE_CMAKE="/path/to/sdrpp_build_dir/sdrpp_module.cmake" to cmake launch arguments

      Download and patch ETSI TETRA codec(in this repository):

          cd src/decoder/etsi_codec-patches
          ./download_and_patch.sh

      Install libtalloc-dev/talloc via package manager

  2.  Build:

          mkdir build
          cd build
          cmake ..
          make
          sudo make install

  4.  Enable new module by adding it via Module manager

Usage:

  1.  Find TETRA frequency you want to receive

  2.  Move demodulator VFO to the center of it

  3.  After some time, it will sync to the carrier and you'll likely see 4 constellation points(sync requires at least ~20dB of signal)

  4.  If the channel is unencrypted, just wait for the voice activity and listen to it!

 

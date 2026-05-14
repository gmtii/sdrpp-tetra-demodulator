# sdrpp-tetra-demodulator

## Timeslot audio filter
 
TETRA carriers use TDMA with 4 timeslots per channel. By default the
plugin decodes and plays audio from all timeslots simultaneously. The
timeslot filter lets you isolate a single timeslot so that only its
voice traffic is sent to the audio output.
 
### Usage
 
In the OSMO-TETRA decoder panel you will find two separate rows:
 
- **Audio TS** — four checkboxes (TS1-TS4). Tick one to restrict audio
  output to that timeslot only. Ticking the active checkbox again
  deselects it and returns to all-timeslot mode.
- **Timeslots** — the original colour-coded status indicators showing
  the current content of each slot in real time (UL / DATA / NDB / SYNC
  / VOICE). These are read-only and update independently of the selector
  above.
 
The selected timeslot is saved in tetra_demodulator_config.json and
restored on next launch.
 
### Implementation notes
 
The osmo-tetra voice callback (put_voice_data) has been extended with
an explicit tn argument carrying the TDMA timeslot number (1-4) from
t_phy_state.time.tn. Filtering is applied inside the callback before
the PCM data reaches the resampler: if a specific timeslot is selected
and tn does not match, the callback returns early without writing to
the output ring buffer. This avoids any interference with the decoder's
internal timeslot tracking logic.

--- Orig readme.md

Tetra demodulator plugin for SDR++

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

 

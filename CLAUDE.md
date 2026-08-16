# NativeTS-kara — working notes

A specialized fork of [TabulaSonora/NativeTS](https://github.com/TabulaSonora/NativeTS), for
rendering the **Sega Super Prologue 21** karaoke corpus in the `kara-pc-port` project. Read
[`README.md`](README.md) for what the fork is and why it exists; this file is the operating context.

`upstream` remote points at `TabulaSonora/NativeTS`. Keep `main` a clean fast-forward of
`upstream/main` and do fork work on branches, so an upstream-bound change can be rebased out cleanly.
A branch holding a duplicate of a commit upstream already carries under a different hash is the one
reliable way to manufacture a conflict later — delete such branches once merged rather than keeping
them.

## Three engines that get conflated

| | what it is |
|---|---|
| **VSC** | Roland Virtual Sound Canvas, `RVI04.dll`/`.sys`/`.DAT`, c. 1996–2003. What *SegaKara for PC* actually drove. Rates 44100/22050/11025 only. Float is `int16/40000`, full scale ±0.8192, hard-railed inside the synth. |
| **SCVA** | Roland SOUND Canvas VA 1.1.6, `SCCore.dll`, 2015. Renders internally at **32 kHz — the hardware's rate** — with an SRC to host rate on output. Full scale ±1.0. |
| **NativeTS** | kode54's C++20 reimplementation of **SCVA**, reading `SCCore.dll` as data. This fork's base. |

**NativeTS is not quiet.** Measured 2026-08-16 on an identical stimulus (GS reset, `C0 00`,
`90 3C 64`, 32 kHz, same map): SCVA peak 0.050568 / rms 0.013033 against NativeTS 0.051331 /
0.013139 — **−0.13 dB peak, −0.07 dB rms**. `kara-pc-port` measured VSC/NativeTS at **+7.87 dB**
with a 7.77 dB spread across 33 songs; that is the VSC↔SCVA product gap, not a NativeTS defect.

Consequence: **never "correct" this engine toward VSC.** It would move it away from its own oracle
and regress upstream by construction. `kara-pc-port`'s trim constant is unit conversion, needed
because their ported output stage was calibrated in VSC's units — not a patch over a bug.

## What each reference can answer

Confusing these is how a fork drifts.

| reference | answers | cannot answer |
|---|---|---|
| **`SCCore.dll`** via `scdec` | what a value should be — **the derivation reference** | anything SCVA itself gets wrong versus hardware |
| **Roland's GS documentation** (`../roland-gs-docs/`) | anything *specified* — defaults, ranges, semantics | coefficient tables, timbre |
| **SP21 direct-feed capture** | whether a whole render sounds like the unit — **acceptance only** | what to change; it is a whole-unit mix through the SP21's own output chain |
| **VSC** | a coarse screen: where two independent Sound Canvas implementations disagree, there is a question | which of them is right |
| **Upstream's `TS_Corpus`** | the regression gate before proposing anything upstream | karaoke-target fidelity |

Two asymmetries worth holding onto:

- **Disagreement informs; agreement proves little.** VSC and SCVA are both Roland's, modelling the
  same hardware family, so they may share design decisions *and* the same departures from the real
  board. Mine the disagreements; do not read agreement as a clean bill of health.
- **Send SysEx questions to the documents before the ears.** They are usually specified. The
  `40 4x 20` EQ default came out of Roland `2-5` and was right against what *both* engines appeared
  to do.

Absent a source from that table, a detected difference is **recorded as a known deviation, not
patched out**. Fitting to one observation through an unknown output chain is exactly the failure the
upstream house style warns about.

## Why accuracy is defined against the board, not against VSC

Matt's framing, and it inverts the obvious reading:

- Users generally hold **SCVA to reproduce the physical Sound Canvas better than VSC did**, so
  building on an SCVA reimplementation is not a compromise against the original app's engine.
- The target is **playback as it would have sounded on the SP21's SC-88 board**, not how a
  25-year-old VST approximated that board. VSC is another approximation of the same hardware, not
  the reference.
- SCVA had known uniformity problems across platforms, hosts and the model range it emulated — part
  of why Roland discontinued it, and plausibly why upstream sees improvements and regressions travel
  together across a heterogeneous corpus.
- This corpus does not carry that burden: one publisher's tracks, one SC-88-based board, one
  decoder, one application, one target platform (Ubuntu 24.04 / amd64).

So a change that deviates from `SCCore.dll` **toward the hardware** can be legitimate here — but only
with an independent source. A physical SC-88 is planned within a few months (budget-limited as of
2026-08-16) and becomes the true derivation oracle when it lands. **Buy the SC-88, not the Pro**: all
200 corpus files send CC#32 = 2.

## Corpus facts worth not rediscovering

- **All 200 files select the SC-88 map themselves** — CC#32 = 2, 5,101 writes; `--map 3` on the
  command line is overridden and renders byte-identically to `--map 2`.
- **No insertion EFX anywhere.** `40 03` does not exist in Roland's SC-88 implementation; EFX arrived
  with the SC-88Pro. `scan-efx` printing nothing is the correct result, not a failure.
- **Ports:** most files need `--ports 2`; some address part 33 and need `--ports 4`, which a 2-port
  render silently drops.
- **The guide melody** (ガイドメロディ) is muted by `40 1x 08 00` — Rx. NOTE MESSAGE off — with CC7
  preserved so a software "melody on" restores it intact. Note `40 1n` block numbering: block 0 is
  **part 10**, block 1 is part 1.

## The handoff loop

`kara-pc-port` identifies a need → handoff here → change made and validated in this fork → handoff
back → they validate in the port → results handoff → branch on the main NativeTS fork →
regression-test against `TS_Corpus` → PR upstream only if clean. A change being right here and wrong
upstream is a **normal, cheap outcome**, not a failure. Handoffs live in
`../../TBDKaraoke/kara-pc-port/docs/handoffs/`.

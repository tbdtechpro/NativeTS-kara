# NativeTS-kara

**A specialized fork of [TabulaSonora/NativeTS](https://github.com/TabulaSonora/NativeTS).**

> If you are looking for a Roland Sound Canvas VA engine to use, **go upstream.** That is the real
> project, it is the one under active development, and it is almost certainly better for your
> purpose than this fork. Nothing here is an improvement on it in the general case, and some of it
> is deliberately worse.

---

## Credit where it belongs

NativeTS is the work of **Christopher Snowhill ([kode54](https://github.com/kode54))** and
**Kevin López Brante ([kddlb](https://github.com/kddlb))**, with contributions from others. Between
them they wrote effectively all of it — the reverse engineering, the DSP, the oracle-driven
verification method, the test corpus and the tooling that makes any of this checkable at all. This
fork is a thin layer of specialization on top of a large body of someone else's careful work.

The engineering standard upstream sets is worth naming, because it is what makes a fork like this
viable: claims are gated on machine-checkable measurements against the reference DLL rather than on
review, and the difference between what is measured and what is inferred is stated plainly. That is
unusually rigorous, and it is why a downstream user can tell whether a change actually helps.

Licensed BSD 3-Clause, copyright (c) 2026 Kevin López Brante — see [`LICENSE`](LICENSE), retained
unchanged. Upstream's own documentation is preserved at
[`README-upstream.md`](README-upstream.md) and remains the authoritative description of the engine.

## What this fork is for

One thing: rendering the **Sega Super Prologue 21** karaoke corpus for a 1:1 port of *SegaKara for
PC* (`SEGAKARA.EXE` v4.01, 2004), whose servers went dark around 2007.

That corpus is unusually narrow. One publisher's tracks, authored for one **SC-88-based** sound
board, decoded by one decoder, driven by one application, on one target platform. Every file in it
selects the SC-88 tone map itself, and none of them use insertion effects — the SC-88 has none.

## Why a fork instead of contributing upstream directly

This is the part worth explaining honestly, because "I made a fork" usually means "I could not be
bothered to upstream."

Upstream has to be right across a large and diverse corpus, and its changes are rightly gated on not
regressing any of it. Mine is the opposite shape: narrow and uniform, SC-88 targeted throughout.

Those two goals can genuinely conflict. A change that measurably improves fidelity against my
material can regress upstream's, and that is not a defect in either project — it is what happens
when one engine serves two different targets. Sound Canvas VA itself was reportedly inconsistent
across platforms, hosts and the model range it emulated, which is part of why Roland discontinued
it; reimplementing something that was not uniform to begin with is a genuinely hard problem, and one
that gets harder the more material you have to satisfy at once.

So the arrangement is:

1. Karaoke-specific changes land **here** and are validated against the karaoke target.
2. Anything that looks like it might help generally gets tested against **upstream's own test
   corpus**.
3. If it improves things there with **no regressions**, it is offered upstream as a contribution.
4. If it does not, it stays here, and that is a normal outcome rather than a failure.

The fork is a staging area for contributions, not a place to diverge for its own sake. As time
permits I intend to work through step 2 for anything that lands here.

That intent is not hypothetical. Work from this workspace has already gone upstream and been merged:
the oracle's SC-88 and SC-55 compatibility-bank redirect, the per-key drum reverb send being the
kit's value scaled by the part's, the harness lookup accepting either sibling layout, and most
recently the system EQ defaulting on — `40 4x 20` is documented as `01 ON` and parts were resetting
it to zero, which cost `roland_sc88_y03` 4.58 dB of bass. Everything that belongs upstream should go
upstream; this fork is for what does not.

### Which direction step 2 actually protects

It would be easy to read that regression test as protecting this fork's freedom to diverge. It
matters more in the other direction.

Upstream's corpus is full of files people have specific attachments to — `canyon.mid` among them,
which is in there at four tone maps and in two container formats. The oracle, the ratchet, and the
per-song tolerance table where every widened bound has to carry a written reason all exist so that
those keep sounding the way they did. Someone who just wants `canyon.mid` to sound like it did when
they were thirteen has a claim on this engine that an obscure karaoke corpus does not get to
override.

Step 2 is what guarantees it cannot. Nothing from here reaches upstream until it is shown to leave
every one of those files alone — and if it cannot, it stays in this repository, which is what this
repository is for.

## What that means for you

Probably that this repository is not useful to you. Changes made for a single, unusual corpus can
make the engine **less** correct for general MIDI, and I will accept that trade when it serves the
karaoke target. There is no intention to maintain this as a general-purpose engine, no support, and
no guarantee it tracks upstream closely.

It is public anyway, in the spirit the upstream project was shared in: if any of it is useful to
someone, or if seeing what a specialized target demands is informative, that is better than keeping
it private. Bug reports about general-purpose behaviour will be redirected upstream, where they
belong.

## How accuracy is judged here

Upstream's oracle is `SCCore.dll` itself, and that remains the reference for deriving any value.
It is joined by a few others that answer different questions:

| reference | answers |
|---|---|
| **`SCCore.dll`** via the `scdec` harness | what a value should be — the derivation reference |
| **Roland's published GS documentation** | anything *specified*: defaults, ranges, semantics |
| **Direct-feed capture of a real SP21** | whether a whole render sounds like the unit — acceptance, not derivation |
| **Upstream's test corpus** | the regression gate before anything is proposed upstream |

The distinction matters. A hardware capture is a mix through the unit's entire output chain, so it
can show that something is wrong without being able to say what to change. A difference detected
that way is recorded as a known deviation rather than patched out by guesswork — the same discipline
upstream applies, for the same reason.

## Status

Freshly forked from upstream `main`, no divergence yet. Everything in
[`README-upstream.md`](README-upstream.md) applies, including how to build it and where the ROM comes
from.

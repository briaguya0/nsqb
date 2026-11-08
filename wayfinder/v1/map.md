# Map: nsqb and its playground

## Destination

A buildable spec for the playground working end to end (open a `.qb` or a
`.prx`, view a script as NeverScript, edit, compile back, byte-identical
round trip) and the nsqb library underneath it. Building happens outside the
map; publishing to npm comes later.

## Notes

- Domain: THUG2 / THUG Pro scripting (QB bytecode, QB keys, NeverScript).
  THUG Pro is a THUG2 mod: two sets of scripts, THUG2's own and the ones
  THUG Pro ships in `thugpro_qb.prx`.
- Background and earlier decisions: doakickflip's `notes/qb-lib.md`
  (direction, playground, prior art) and `notes/formats.md`.
- Tracker: local markdown. Tickets in [issues/](issues/), research write-ups
  in `research/`, prototypes in `prototypes/`. Research stays on the docs
  branch; nothing pushed without asking.
- Nothing derived from THUG2 / THUG Pro is committed: no `.qb` fixtures, no
  key lists. Describe findings in our own words.
- One question at a time when grilling. Grilling tickets: use the grilling
  and domain-modeling skills.
- Tickets 01 and 02 also answer doakickflip's `wayfinder/v1` tickets 01 and
  02; link the answers back there.
- Scope, decided while charting (2026-11-08):
  - THUG2 and THUG Pro scripts only.
  - Every `.qb` in both sets decompiles and compiles back byte-identical.
  - One package for the browser (playground) and Node (doakickflip's
    Electron main): bytes and text in and out, no file access of its own.
  - NeverScript only.
  - The playground opens `.prx` files too, via prxhawk (a playground
    dependency, not a library one).
  - Real scripts are round-trip checked locally only; CI runs our own
    hand-written `.ns` samples.
  - Already decided in `qb-lib.md`: no key lists shipped; playground on
    React + Vite, Shiki then Monaco, our own NeverScript TextMate grammar
    living in the playground; a hand-written sample for the first run.

## Decisions so far

## Not yet specified

- nsqb's public API (hangs on what it builds on).
- How compile errors are reported, in the library and the playground.
- Editing in Monaco beyond basic highlighting.

## Out of scope

- Other games (THPS4, THAW, ...).
- Neversoft-style `.q` text: `.q` files get converted to `.ns` once.
- Publishing to npm.
- Hosting the playground: not for the first release, not ruled out later.
- CI against real game files (ROM-in-secrets style, as Zelda64Recomp does):
  logs could leak game data.

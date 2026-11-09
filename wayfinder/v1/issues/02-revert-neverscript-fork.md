# Which repo is Revert's NeverScript fork, what's its licence, and does it decompile?

Type: research
Status: resolved

## Question

Revert's mods are compiled by a patched NeverScript fork described as a byte-perfect recompiler. Which repo is it, what licence covers it (upstream NeverScript has none), what does its LIMITATIONS.md list, and does it add a `.qb` → `.ns` decompiler? Also answers doakickflip's wayfinder/v1 ticket 02.

Research: [research/revert-neverscript-fork.md](../research/revert-neverscript-fork.md)

## Answer

violetvandal/NeverScript, vendored as a submodule of thugkit (itself a submodule of revert). No licence; about 73% of its lines are still byxor's, so its code can't be used. LIMITATIONS.md: THUG2 PC only, 226 of ~257 files byte-identical, decompiler coverage the main gap, one compiler hang, new symbols can't be added. Upstream already had a partial decompiler (`ns -d`); the fork rewrote it, THUG2 only. thugkit's PRE codec is MIT. Details: research/revert-neverscript-fork.md.

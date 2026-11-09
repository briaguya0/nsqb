# Does NeverScript's grammar cover every construct in THUG2 and THUG Pro scripts?

Type: research
Status: resolved

## Question

Does NeverScript (byxor's, and Revert's fork if it differs) cover every construct in THUG2's scripts and THUG Pro's scripts, notably `switch`, `elseif` and `Random*`? List the gaps. Context: doakickflip notes/qb-lib.md and notes/formats.md. Also answers doakickflip's wayfinder/v1 ticket 01.

Research: [research/neverscript-grammar-coverage.md](../research/neverscript-grammar-coverage.md)

## Answer

Upstream byxor/NeverScript doesn't cover THUG2 (no `switch`, missing random and loop forms, a `while` guard the games lack, unfinished decompiler). Revert's fork covers nearly everything that occurs; gaps: Begin with a repeat count has no syntax, THUG Pro's line-number markers are dropped, its own open mismatches / errors, never tested on THUG Pro. `elseif` is not a gap (its opcode never occurs; games nest if/else). Naming clash: the fork's `random2` is a different opcode from our notes' "Random2". Details: research/neverscript-grammar-coverage.md.

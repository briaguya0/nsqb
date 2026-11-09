# NeverScript grammar coverage vs THUG2 / THUG Pro scripts

Ticket: [01](../issues/01-neverscript-grammar-coverage.md). Researched 2026-11-08.

## Answer

- **Upstream NeverScript doesn't cover THUG2's scripts.** Its compiler has
  no `switch`, no counted loops and no `RandomRange`, and emits only one of
  the three random opcodes that occur. Its decompiler is unfinished.
- **Revert's fork covers nearly every construct that actually occurs**,
  including `switch`, all three random variants that occur, and
  `RandomRange`. What's left: one loop form (begin `0x20` with a repeat
  count) has no syntax of its own, THUG Pro's line-number newlines (`0x02`)
  are dropped, and the fork's own open bugs.
- **`elseif` is a non-issue.** The `0x27` opcode never occurs in either
  game, and both compilers lower `else if` to nested if/else, which is the
  form the games' scripts themselves use.
- Neither compiler emits the other opcodes our token table lists (`0x11`,
  `0x13`, `0x15`, `0x18`, `0x27`, `0x34`, `0x37`, `0x38`). None of them occur
  in either game, so that costs nothing.

## Sources (pinned)

- Upstream: [byxor/NeverScript @ `77baa23`](https://github.com/byxor/NeverScript/tree/77baa233a58c25e271399c534fc6ef89bbb43c06)
  (2023-05-01, HEAD of `master`).
- Revert's fork: [violetvandal/NeverScript @ `f233604`](https://github.com/violetvandal/NeverScript/tree/f2336046b966ea70a344e8c9164e18ba14314522)
  (2026-06-13). This is the commit thugkit vendors as the submodule
  `third_party/neverscript` ([thugkit `.gitmodules` @ `09f4228`](https://github.com/violetvandal/thugkit/blob/09f4228d2bdd199a1594b8e1f039b1596313d438/.gitmodules)),
  and the fork's `master` (`73fe0f9`, merge of PR #1) has the same tree.
  Revert links thugkit as a submodule ([revert `.gitmodules` @ `422c1b9`](https://github.com/violetvandal/revert/blob/422c1b9b0e438792db0b57b40b07f33b5fc985b1/.gitmodules)),
  and its docs call it "our patched compiler fork (vendored into thugkit as a
  submodule)" ([codecs.md:41-47](https://github.com/violetvandal/revert/blob/422c1b9b0e438792db0b57b40b07f33b5fc985b1/docs-site/codecs.md#L41-L47)).
  An open, unmerged PR #2 (`a89c4fd`, 2026-11-01) adds only a lenient
  decompile mode (`cmd/ns/ns.go`, `decompiler/decompiler.go`), so it doesn't
  change the grammar. The fork has no licence file either. Ticket 02 covers
  the fork itself.
- Opcode inventory: doakickflip `notes/formats.md:25-65` and
  `tools/qbdec.py:99-178`.
- Occurrence counts: a throwaway token walker over doakickflip's
  `extracted/thug2` (380 `.qb`) and `extracted/thugpro` (430 `.qb`), which
  steps over operands the same way `qbdec.py` does. Counts only, nothing else
  taken from the files.

Line refs below are `path:line` at the pinned commits: **U** = upstream,
**F** = fork.

## Which opcodes occur

Opcodes that occur (count in THUG2 / THUG Pro):

| opcode | construct | THUG2 | THUG Pro |
|---|---|---|---|
| 47 / 48 | fast if / else | 12827 / 3443 | 15020 / 4168 |
| 28 | endif | 12827 | 15020 |
| 3C / 3D / 3E / 3F | switch / endswitch / case / default | 428 / 428 / 1729 / 146 | 397 / 397 / 1774 / 151 |
| 49 | short jump (always inside a switch) | 3322 | 3453 |
| 2F | Random | 395 | 784 |
| 40 | RandomNoRepeat | 200 | 156 |
| 41 | RandomPermute | 4 | 8 |
| 2E | jump (all are random branch jumps: count = Σ(branches − 1)) | 2095 | 2690 |
| 30 | RandomRange (always followed by a pair, `1F`) | 583 | 515 |
| 20 … 21 | begin `0x20` … repeat, no count | 476 | 723 |
| 20 … 21 n | begin `0x20` … repeat **with count** | 356 | 319 |
| 00 … 21 [n] | begin `0x00` … repeat (51+53 / 80+63 without/with count) | 104 | 143 |
| 02 | newline + line number | 0 | 783 (5 files) |
| 1C | local string | 3052 | 3222 |
| 12 / 14 | `<` / `>` | 214 / 584 | 197 / 585 |
| 32 / 33 / 39 / 42 | `or` / `and` / `NOT` / `:` | 186 / 62 / 2993 / 2823 | 183 / 30 / 3191 / 2772 |

Also present in both: everything else from `01` to `2D` that our table has,
except the ones below. `2D` is always followed by a name (`16`), so the bare
`<>` form never occurs.

**Never occur in either game:** `0x11` (`==`), `0x13` (`<=`), `0x15` (`>=`),
`0x18` (hex int), `0x25` / `0x26` (old if/else), `0x27` (elseif), `0x34`
(xor), `0x37` (Random2), `0x38` (RandomRange2). No unknown opcodes.

## Coverage by construct

| construct | upstream compiler | fork compiler | fork decompiler |
|---|---|---|---|
| if / else (`47`/`48`/`28`) | yes, `47`/`48` for `-targetGame thug2` (U `compiler/output.go:563-606`) | yes (F `compiler/output.go:697-728`) | yes, short and long forms (F `decompiler/decompiler.go:501-531`) |
| elseif | lowered to nested if/else (U `compiler/output.go:427-462`); no `0x27` | same lowering (F `compiler/output.go:561`) | n/a: no `0x27` handler, none needed |
| switch / case / default | **no**: no lexer keyword (U `compiler/lexer.go:392-417`) | yes, byte-identical incl. `0x49` offsets (F `compiler/output.go:360-412`; F `LIMITATIONS.md:79-114`) | yes (F `LIMITATIONS.md:82-86`, 137-141) |
| Random `2F` | yes (U `compiler/output.go:196-261`) | `random` (F `compiler/output.go:251-261`) | yes (F `decompiler/decompiler.go:1129-1136`) |
| RandomNoRepeat `40` | **no** | `random2` keyword (F `compiler/lexer.go:470`, `output.go:259`) | yes, as `random2` |
| RandomPermute `41` | **no** | `random3` keyword (F `compiler/lexer.go:467`, `output.go:257`) | yes, as `random3` (commit `faba8d6`) |
| non-canonical random jump targets | no | `random <delta> {` syntax (F commit `7c66fdb`; `decompiler/decompiler.go:1236-1242`) | yes |
| RandomRange `30` | **no** keyword (U `compiler/lexer.go:392-417`) | `randomrange (a, b)` (F `compiler/parser.go:308-316`, `output.go:130-132`) | yes (F `decompiler/decompiler.go:1244-1254`) |
| begin `20` … repeat, no count | `while`, but for THUG2 it injects a guard variable and `if … break` (U `compiler/output.go:263-336`) | `while { }`, plain `20 … 21` (F `compiler/output.go:338-348`) | yes (F `decompiler/decompiler.go:623-646`) |
| begin `20` … repeat **n** | **no** | **no dedicated syntax**: the counted form always emits `00` (F `compiler/output.go:349-359`) | the `while` path stops at `21` and doesn't read a count (F `decompiler/decompiler.go:623-646`) |
| begin `00` … repeat [n] | **no** | `Begin { } Repeat [n]` (F `compiler/parser.go:1503-1550`, `output.go:349-359`) | yes (F `decompiler/decompiler.go:647-680`) |
| newline + line number `02` | no | no (`0x02` is never written in F `compiler/output.go`) | read, number discarded (F `decompiler/decompiler.go:260-264`) |
| local string `1C` | **no** (strings always `1B`, U `compiler/output.go:101-109`) | `%"…"` (F `LIMITATIONS.md:126-127`) | yes |
| `(…)` with several operators | one operator per pair of parens | flat expression, byte-identical (F `LIMITATIONS.md:180-204`) | yes |
| `<=` `>=` `!=` | lowered to `NOT` + the opposite comparison (U `compiler/output.go:128-160`) | same, except inside a flat `(…)` chain, where it writes `13`/`15` (F `compiler/output.go:150-185`, 776-805) | — |

### Upstream decompiler

The README strikes the decompiler out and says "Not implemented yet"
(U `.github/README.md:19`, 58-60). The code has no `0x41`, `0x47`, `0x48` or
`0x49` (U `decompiler/decompiler.go:14-67`). It also reads a random as count
then offsets, with no weights (U `decompiler/decompiler.go:744-785`), while
THUG2's randoms carry `n × u16` weights before the offsets
(`notes/formats.md:59`). So it can't read THUG2 randoms. The fork rewrote it.

## Gaps

### Upstream (byxor)

`switch`/`case`/`default`, RandomNoRepeat, RandomPermute, RandomRange,
counted and `0x00` loops, and local strings are all missing. The THUG2
`while` output isn't the game's form. In practice upstream can't express
about a quarter of THUG2's files: 95 of 380 contain a switch alone.

### Fork (Revert)

1. **Begin `0x20` with a repeat count** (356 in THUG2 / 319 in THUG Pro).
   `while` has no count and `Begin … Repeat n` always emits `0x00`, so this
   form has no syntax of its own. The decompiler leaves the count for the
   following statement. Whether that happens to recompile byte-identically
   wasn't checked: running their code was out of scope. The fork lists "a
   Begin/Repeat `0x00` vs While `0x20` loop-opcode choice" as a remaining
   mismatch class (F `LIMITATIONS.md:58-60`).
2. **`0x02` line-number newlines** (THUG Pro only, 783 in 5 files): the
   decompiler throws the number away and the compiler can't write `0x02`,
   so those files can't round-trip byte-identically.
3. **Open bugs per the fork's own notes:** at its last sweep, 13 mismatches
   and 18 errors out of about 257 THUG2 files (F `LIMITATIONS.md:25-30`; some
   later commits fix more). There's also a compiler infinite loop on a
   parenthesised value that subscripts a local and then takes a member
   (F `LIMITATIONS.md:162-168`). Its round trips were only measured on THUG2
   archives, and only one level was tested in-game (F `LIMITATIONS.md:206-210`).
   Nothing in the fork mentions THUG Pro.
4. **Not gaps:** the opcodes that never occur (`0x11`, `0x13`, `0x15`,
   `0x18`, `0x25`/`0x26`, `0x27`, `0x34`, `0x37`, `0x38`).

### Naming hazard for nsqb

The fork's keyword `random2` means opcode `0x40` (RandomNoRepeat), and
`random3` means `0x41` (F `decompiler/decompiler.go:1132-1135`). Our notes
call `0x37` "Random2" (`notes/formats.md:59`). Any nsqb syntax that stays
compatible with the fork has to keep its spellings, and our docs need to say
so to avoid confusion. Keywords are matched case-sensitively, as prefixes
(F `compiler/lexer.go:425-476`).

## Not done

- Didn't build or run either NeverScript (untrusted code), so nothing here
  is a measured round trip.
- Construct-level patterns inside expressions (e.g. how often the
  infinite-loop shape occurs) weren't counted, only opcodes and loop and
  random structure.

# Revert's NeverScript fork

Answers [issues/02](../issues/02-revert-neverscript-fork.md). Researched 2026-11-08 from clones of the repos below; nothing was built or run.

Pinned commits used for citations:

- fork: `violetvandal/NeverScript` @ `73fe0f981f9572931bea33fd089d0c415e9d51b3` (master, = merge of `thug2-runtime-safe-recompiler`)
- upstream: `byxor/NeverScript` @ `77baa233a58c25e271399c534fc6ef89bbb43c06` (last commit, 2023-05-01)
- `violetvandal/thugkit` @ `09f4228d2bdd199a1594b8e1f039b1596313d438`
- `violetvandal/revert` @ `422c1b9b0e438792db0b57b40b07f33b5fc985b1`
- `violetvandal/revert-mods` @ `52b303c7c366db2e569825be6d67201caf5db0a6`

## Which repo

**[github.com/violetvandal/NeverScript](https://github.com/violetvandal/NeverScript)**, a GitHub fork of byxor/NeverScript. It is not vendored as source into revert. The chain is:

- revert has `tools/thugkit` and `mods` as git submodules (revert `.gitmodules`; gitlinks `tools/thugkit` → `09f4228`, `mods` → `52b303c`).
- thugkit has `third_party/neverscript` as a submodule pointing at the fork, branch `thug2-runtime-safe-recompiler`, gitlink → `f2336046b966ea70a344e8c9164e18ba14314522` (thugkit `.gitmodules`). That commit has the same tree as the fork's master merge `73fe0f9` (`git diff --stat f233604 73fe0f9` is empty).
- thugkit `go.mod`: `replace github.com/byxor/NeverScript => ./third_party/neverscript`, so it compiles `.ns` in-process ([thugkit README.md:58-64](https://github.com/violetvandal/thugkit/blob/09f4228d2bdd199a1594b8e1f039b1596313d438/README.md#L58-L64); [revert docs-site/codecs.md:41-48](https://github.com/violetvandal/revert/blob/422c1b9b0e438792db0b57b40b07f33b5fc985b1/docs-site/codecs.md#L41-L48)).
- revert-mods' standalone `apply-mods.sh` route expects an `ns` binary built from the fork ([revert-mods README.md:72-74](https://github.com/violetvandal/revert-mods/blob/52b303c7c366db2e569825be6d67201caf5db0a6/README.md#L72-L74)).

## Licence

**None.** The fork has no LICENSE file at `73fe0f9` (`git ls-files` shows none). Its README is upstream's, unchanged by any fork commit (`git diff 77baa23 73fe0f9 -- .github` is empty), and it still says "There's no license, you don't owe me a thing" ([.github/README.md:110](https://github.com/violetvandal/NeverScript/blob/73fe0f981f9572931bea33fd089d0c415e9d51b3/.github/README.md#L110)).

The MIT licences on revert, revert-mods and thugkit (`LICENSE`, "Copyright (c) 2026 violetvandal" in each) cover those repos. The fork is only referenced from them as a submodule, so those licences don't cover it. revert's THIRD-PARTY-NOTICES.md lists DXVK, WidescreenFixesPack and Ultimate ASI Loader but not NeverScript ([THIRD-PARTY-NOTICES.md:1-90](https://github.com/violetvandal/revert/blob/422c1b9b0e438792db0b57b40b07f33b5fc985b1/THIRD-PARTY-NOTICES.md)).

How much of it is upstream code (`git blame --line-porcelain` over every tracked `.go` file at `73fe0f9`):

| | byxor (upstream) | VandalViolet (fork) |
|---|---|---|
| all `.go` lines | 7,951 | 2,942 |
| `compiler/parser.go` (2,247) | 1,424 | 823 |
| `compiler/output.go` (807) | 548 | 259 |
| `compiler/ast.go` (350) | 221 | 129 |
| `compiler/lexer.go` (510) | 442 | 61 |
| `decompiler/decompiler.go` (1,654) | 11 | 1,643 |
| `cmd/ns/ns.go` (238) | 231 | 7 |

The fork adds 33 commits on top of upstream's 261, all dated 2026-06-10 to 06-13 (`git log`). `git diff --stat 77baa23 73fe0f9`: 9 files, +3,173 / −1,251. So about 73% of the Go lines are still byxor's, and the compiler is mostly upstream code with patches. The decompiler file is almost all fork lines, but it is a rewrite of upstream's 1,041-line `decompiler.go` in place, not a new file, so blame counts don't show it is independent of upstream. Even if the fork had a licence, it couldn't cover the upstream lines. As it stands, neither part has one.

## LIMITATIONS.md

Summarised from [LIMITATIONS.md](https://github.com/violetvandal/NeverScript/blob/73fe0f981f9572931bea33fd089d0c415e9d51b3/LIMITATIONS.md). Script names and opcode details are left out on purpose.

- Scope: the fork patches "NeverScript v0.9" to decompile and recompile THUG2 (PC) QB that the retail game loads (L3-6).
- §1 / §1a byte-identity: a bulk round-trip sweep (decompile → recompile → `cmp`) over about 257 `.qb` files went from 177 to **226 byte-identical, 13 mismatched, 18 erroring** (L25-30). One large level script is still 1 byte off (L20-21). The leftover mismatches are a loop-opcode choice, some decompile gaps, and recompiler hangs on a family of scripts (L58-60).
- §2 decompiler coverage is "the main real gap". The doc says to fix gaps reactively, one script at a time (L62-77).
- §2b native `switch` round-trips byte-identically. The earlier if/elseif lowering was removed as runtime-unsafe (L79-86).
- §2d front-end scripts: all six decompile, five of six round-trip. One still sends the compiler into an infinite loop on a parenthesised subscript-then-member argument (L162-168). Decompiling a file doesn't mean it will recompile (L176-178).
- §2c chained parenthesised expressions are fixed (L180-204).
- §3 only one level has been validated in-game (L206-210).
- §4 a mod can't add brand-new symbols. The trailing name table is copied in the original order, and that hash order hasn't been reproduced. Whether the order matters at runtime is unconfirmed (L212-221).
- §5 the engine is out of scope (L223-226).
- Pre-existing repo problems: `compiler/tests` has two `main`s, `newcompiler` doesn't build, and the consistency test needs `roq.exe` (L228-231).

## Decompiler (`.qb` → `.ns`)

- **Upstream already had a partial one.** byxor's `decompiler/decompiler.go` (1,041 lines, commits 2019-04-28 to 2022-10-29) exposes `Decompile(qb []byte)` ([decompiler.go:70](https://github.com/byxor/NeverScript/blob/77baa233a58c25e271399c534fc6ef89bbb43c06/decompiler/decompiler.go#L70)), and `ns -d` already calls it ([cmd/ns/ns.go:164-180](https://github.com/byxor/NeverScript/blob/77baa233a58c25e271399c534fc6ef89bbb43c06/cmd/ns/ns.go#L164-L180)). The README still crosses it out and says "Not implemented yet" ([.github/README.md:19,60](https://github.com/byxor/NeverScript/blob/77baa233a58c25e271399c534fc6ef89bbb43c06/.github/README.md#L60)). qb-lib.md's line "decompiler not implemented" reflects the README, not the code.
- **The fork rewrites it** (blame above) into a round-trip decompiler for THUG2: switch, random variants, counted loops, flat expressions, and preserved non-canonical checksums and name tables (fork commit subjects `8cf9379`..`f233604`). The only `ns.go` change is dropping the header comment so the round trip stays byte-identical ([cmd/ns/ns.go:176-180](https://github.com/violetvandal/NeverScript/blob/73fe0f981f9572931bea33fd089d0c415e9d51b3/cmd/ns/ns.go#L176-L180)).
- **How complete:** see the LIMITATIONS figures above (226 of ~257 THUG2 files round-trip byte-identically, plus known decompile and recompile gaps).
- **THUG2 only.** `-targetGame` still defaults to `thug2` and only accepts thps3/thps4/thug1/thug2 ([cmd/ns/ns.go:35,81,120-121](https://github.com/violetvandal/NeverScript/blob/73fe0f981f9572931bea33fd089d0c415e9d51b3/cmd/ns/ns.go#L120-L121)). The fork's Go and Markdown files never mention THUG Pro (grep for `thugpro`/`thug pro`/`thug_pro` finds nothing), so it has not been tested on THUG Pro.

## thugkit PRE/PRX codec

`thugkit/prx` (`prx.go`, `lzss.go`) is MIT, all 13 thugkit commits are by violetvandal, and the code says it is a Go port of the toolkit's own `tools/prx/*.py` ([prx/prx.go:1-13](https://github.com/violetvandal/thugkit/blob/09f4228d2bdd199a1594b8e1f039b1596313d438/prx/prx.go#L1-L13)). Nothing in the repo says it came from NeverScript's unlicensed `pre_generator`.

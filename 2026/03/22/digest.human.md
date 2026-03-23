Here's the daily digest for March 22, 2026:

## The day in brief

March 22 saw moderate activity level with 57 emails across 18 threads, featuring ongoing discussions about graph rendering limits, promisor file handling, and test modernization. The most notable developments include Pablo Sabater's v3 of the `--graph-lane-limit` feature and Lorenzo Pegorari's GSoC work on promisor file consolidation during repacks.

## Notable threads

### Graph column limiting reaches v3

Pablo Sabater's series to limit `git log --graph` output width progressed to v3 with significant design refinements. The patch now uses `--graph-lane-limit` as the final option name and implicitly enables `--graph` as Junio suggested. Johannes Sixt raised new concerns about the implicit behavior potentially complicating future configuration, reopening discussion about the UX design. The implementation remains technically sound, with thorough test coverage demonstrating 2-3 column limits in merge scenarios. The series addresses a 2008 TODO in graph.c while deliberately avoiding gitk-style rearrangement.

### Promisor file handling improvements

Lorenzo Pegorari's GSoC work on promisor file consolidation during repacks advanced to v2 with significant implementation overhaul. Responding to Eric Sunshine's review, the new version replaces O(n²) duplicate checking with a strset-based O(1) solution and fixes memory leaks. The series now includes four patches: documentation, the core consolidation helper, repack integration, and new test coverage. The changes properly preserve ref-to-packfile debug information that was previously lost during repacks, addressing a `NEEDSWORK` comment from 5374a290. The test addition in t7700-repack.sh provides concrete validation of the debug info preservation.

### Remote-curl segfault fix converges

The thread about `git ls-remote` segfaults outside repository contexts reached consensus on a solution. K Jayatheerth's v3 patch skips hash-algorithm checks when `the_repository` is NULL, addressing the root cause Jeff King identified. Junio and Peff agreed these checks are meaningless without a repository context. The test case was simplified using Git's `nongit` helper per maintainer feedback. This resolves a subtle architectural edge case affecting operations with global fetch refspecs configured.

## In brief

**strbuf_getwholeline optimization concludes** -- Junio and Peff agreed to leave the long-stable function unchanged beyond the original NULL check removal goal, despite deeper behavioral questions raised during discussion.

**Backfill argument validation refined** -- Siddharth Shrimali's v3 makes `git backfill` properly reject unexpected arguments with cleaner error messages and test coverage, though Derrick Stolee later raised concerns about forward compatibility with planned revision arguments.

**Test modernization continues** -- Multiple contributors replaced raw `test -f` checks with `test_path_is_missing` across several test scripts, though some patches needed revision to maintain test framework setup.

**STRBUF_INIT_CONST RFC** -- New contributor Mateo Patino inquired about reviving this stalled optimization, with Eric Sunshine providing historical context about fundamental API compatibility issues.

**Branch flag validation debated** -- Junio put K Jayatheerth's `git branch --show-current -v` rejection patch on hold after Phillip Wood and Sergey suggested the flags could have meaningful combined behavior.

## On the radar

**Geometric repacking edge cases** -- Taylor Blau's series addressing MIDX bitmap failures with excluded-but-open packs may need adjustments to handle NULL pack pointers in object walking logic.

**Rustification implications** -- Randall Becker's concerns about NonStop platform support remain unresolved as Rust integration work continues in other threads.
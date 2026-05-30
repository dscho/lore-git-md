# Here's the Git mailing list daily digest for May 29, 2026:

## The day in brief

Friday, May 29 saw moderate traffic with 46 emails across 16 threads, featuring several performance optimizations reaching maturity, documentation refinements, and workflow discussions. The standout developments include Taylor Blau's bitmap optimization series concluding with Jeff King's approval, Michael Montalbo's v3 of the `diff.<driver>.process` RFC, and a significant index-pack speedup from a one-line change.

## Notable threads

### Bitmap optimization series concludes

Jeff King (Peff) provided final approval for Taylor Blau's eight-patch series optimizing pack-bitmap-write performance. The changes achieve a 60% speedup and 72% storage reduction for large repositories while maintaining backward compatibility. Discussion revealed that proper test setup was initially masking the full benefits - when explicitly configured, path-walk shows 98.8% time reduction for recent fetches. The series now moves toward integration after thorough performance validation from both Blau and Derrick Stolee.

### diff.<driver>.process reaches v3

Michael Montalbo posted v3 of his RFC enabling external tools to inject diff hunks into Git's pipelines. The revision adds subprocess management refactoring, improved error handling, and comprehensive documentation with packet diagrams. The series now supports both diff and blame operations, allowing tools to mark files as unchanged (affecting blame attribution) while preserving features like word-diff. Junio Hamano's earlier concerns about error reporting and global state have been addressed, though performance impact (~17% blame overhead) remains a consideration.

### Index-pack delta resolution optimization

A one-line change to index-pack's delta handling yielded 15-16% faster wall times in benchmarks. The patch removes a premature `free_base_data()` call that was forcing redundant reconstruction of delta bases still needed for child resolutions. The existing delta cache now handles reuse naturally, with only 1.9% RSS increase in the worst case. The well-validated optimization demonstrates how small, focused changes can produce significant performance wins in core operations.

## In brief

**Line-log integration merged** -- D. Ben Knoble confirmed the final version of Michael Montalbo's line-log series looks good as merged to `next`, concluding the effort to unify `-L` with Git's standard diff pipeline.

**HTTP packfile leak fix finalized** -- Jeff King approved Lorenzo Pegorari's fix for memory leaks in HTTP pack index handling after exploring alternative tempfile management strategies, confirming the simpler solution is correct.

**Commit list API cleanup** -- Kristoffer Haugsbakk's removal of deprecated commit list functions gained Patrick Steinhardt's Acked-by, while prompting philosophical discussion from Peff about Git's deprecation workflow efficiency.

**macOS linker warning resolved** -- Harald Nordgren's v2 patch addresses Xcode 16.3+ warnings by conditionally adding `-fno-common` in config.mak.uname rather than modifying source.

**Windows pathspec test fix** -- Kristofer Karlsson's workaround for backslash pattern issues in `t3070-wildmatch.sh` was queued after confirming the behavior dates to 2018, though deeper pathspec handling questions remain.

**merge --message documentation** -- Junio noted undocumented `--no-message` behavior while reviewing a patch to document `--message`, revealing broader gaps in merge option documentation.

**Ancient timestamp workaround** -- Luna Schwalbe and Kristoffer Haugsbakk identified `@` prefix syntax as a solution for sub-1973 dates, with Junio confirming this intentionally added but never documented feature.

## On the radar

**git son vs submodules debate** -- Claus Schneider proposed enhancing submodules with branch-tracking as an alternative to Evan Haque's `git son` command, shifting the discussion from implementation to fundamental design.

**Hook argument handling** -- Wesley Schwengle's report of unexpected argument passing in config-based pre-push hooks evolved into broader documentation gaps discussion, with Peff and D. Ben Knoble clarifying native vs wrapper hook contracts.
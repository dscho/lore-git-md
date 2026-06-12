Here's the daily digest for June 11, 2026:

## The day in brief

A busy day with 98 emails across 30 threads, featuring significant progress on several fronts. Key highlights include Junio Hamano's release of Git v2.55.0-rc0 with major new features, finalization of security hardening for ANSI escape sequence handling, and substantive discussions about performance optimizations and architectural changes. The day also saw multiple documentation improvements and the conclusion of several long-running patch series.

## Notable threads

### Git v2.55.0-rc0 released

Junio Hamano announced the first release candidate for Git 2.55, containing 397 non-merge commits from 70 contributors. This major update includes parallel hook execution, improved terminal control sequence handling, new `git format-rev` and `git history fixup` commands, Linux fsmonitor daemon support, and significant internal refactoring around object storage and performance paths. The release also marks Rust support being enabled by default (though still optional) and continued progress on removing `the_repository` global variable.

### Security hardening timeline accelerated

Junio Hamano accelerated the rollout of security fixes for ANSI escape sequence injection vulnerabilities (CVE-2024-32002, CVE-2024-52005). Originally planned for Git 3.0, the secure-by-default behavior (only allowing ANSI color sequences) will now ship in Git 2.55 based on successful production validation in Git for Windows and Red Hat environments. The change removes a transitional compatibility layer while preserving legitimate ANSI color sequences used by pre-receive hooks.

### Performance optimization tradeoffs

Multiple performance optimization threads reached key decision points:
- Arijit Banerjee's index-pack delta resolution optimization will use the simpler v2 implementation after Jeff King noted v3's additional complexity provided no measurable benefit
- Tamir Duberstein's `git ls-files` optimization was approved to remove its single-pathspec restriction entirely, accepting minor theoretical regressions for cleaner code
- Discussion continued on revision walking optimizations, with Junio agreeing to coordinate merging of `--max-count-oldest` with Mirko Faina's related patch series

### `git history drop` implementation

Patrick Steinhardt's 10-patch series introducing a `git history drop` subcommand reached v5, with all preparatory refactoring complete and extensive test coverage added. The series modernizes reset machinery and advances `the_repository` removal while implementing the core drop operation. Junio identified one remaining issue regarding dry-run behavior in reference updates that needs addressing before final merge.

## In brief

**Trailer parsing fixes** -- Kristoffer Haugsbakk and Jeff King finalized the approach for handling URLs in commit trailers, modifying `find_separator()` to exclude lines containing `://` from being parsed as trailers.

**Documentation standardization** -- Jean-Noël Avila and Tuomas Ahola completed v3 of their AsciiDoc markup fixes, resolving octothorpe escaping issues through backtick usage after testing revealed problems with backslash escaping.

**Ref backend modernization** -- Patrick Steinhardt began a 9-part series to eliminate chdir_notify_reparent() usage in ref backends, transitioning to consistent absolute path handling. Jeff King noted potential performance considerations from historical optimizations using relative paths.

**Test hygiene fix** -- Junio Hamano fixed a test issue in t1400 where lingering FIFO files could cause hangs, with Patrick Steinhardt acknowledging the cleanup.

**Contributor documentation** -- Weijie Yuan proposed adding email etiquette advice about trimming quoted text to MyFirstContribution.adoc, which Junio approved as well-placed and clear.

**Timestamp precision discussion** -- brian m. carlson proposed runtime configuration for timestamp precision (similar to untracked-cache) as a solution for worktree diff slowdowns, building on earlier analysis by D. Ben Knoble and Jeff King.

## On the radar

**Config includes sandboxing** -- Derrick Stolee and Jeff King continue discussing whether to implement a binary disable mechanism or pursue comprehensive sandboxing for Git config includes, with Stolee providing concrete build system use case details.

**Reachability function consolidation** -- Kristofer Karlsson's effort to unify `get_reachable_subset()` and `tips_reachable_from_bases()` is reconsidering approach after Derrick Stolee suggested automatic traversal mode selection rather than configurable behavior.

**SubmittingPatches guidelines** -- Kristoffer Haugsbakk's documentation series about commit conventions awaits v2 incorporating Junio's feedback, particularly around DCO requirements for `Based-on-patch-by` trailers.
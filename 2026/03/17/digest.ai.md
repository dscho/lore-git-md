Here's the Git mailing list digest for March 17, 2026:

---

### The day in brief
March 17, 2026 saw active development across multiple fronts with 89 emails across 38 threads. The day featured significant progress on several major features including `git replay` revert capability, HTTP 429 rate limiting, and `git backfill` enhancements, along with ongoing work on the `the_repository` removal effort. Junio Hamano's "What's cooking" report provided a comprehensive snapshot of the project's current state.

---

### Notable threads

**`git replay` gains revert capability**
The long-running effort to add revert functionality to `git replay` reached its final stages with v4 patch series addressing all substantive review feedback. Phillip Wood confirmed the implementation meets expectations, with only minor documentation nits remaining. The feature enables commit-by-commit reversal directly on bare repositories while preserving individual commit messages, using a distinct operation mode that routes through Git's standard sequencer infrastructure.

**HTTP 429 rate limit handling**
Vaidas Pilkauskas's HTTP client improvements reached v6, implementing comprehensive rate limit support with Retry-After header handling. The series now cleanly separates preparatory strbuf fixes (already merged) from the core HTTP functionality, which includes configurable retry limits and extensive test coverage. The implementation supports both delay-seconds and HTTP-date formats while maintaining compatibility with older curl versions.

**`git backfill` enhancements for partial clones**
Derrick Stolee proposed a 5-patch series expanding `git backfill` to support revision arguments and pathspec filtering, enabling more targeted blob downloads in partial clones. The changes allow operations like `git blame` and `git log -L` to efficiently fetch only needed blobs. Junio Hamano validated the approach while raising considerations about future edge cases like cross-file copy detection.

**In brief**
- **GSoC proposals** -- Multiple students refined proposals including prioritized promisor fetching (Abraham Adekunle), `git repo info` improvements (Jayatheerth Kulkarni), and repository analysis (Mansi Singh)
- **Line-log diff improvements** -- Michael Montalbo's series routing `git log -L` through standard diff machinery was merged with positive review
- **Subcommand autocorrection** -- Jiamu Sun's parse-options API extension addressed final review questions about edit distance thresholds
- **Worktree semantics** -- Phillip Wood provided additional justification for `the_repository` removal changes in worktree code
- **Reference transaction hooks** -- Eric Ju's v3 patch adding a "preparing" phase received maintainer approval
- **Commit-graph fix** -- Patrick Steinhardt corrected generation number calculation for dates beyond 2038

**On the radar**
- **Exit code standardization** -- Discussion continues on changing help request exits from 129 to 0, with maintainer support but compatibility considerations
- **`the_repository` removal** -- Coordination needed between parallel refactoring efforts in the add-patch subsystem
- **Documentation improvements** -- Multiple threads concluded with documentation patches for cherry-pick conflicts and format-patch options

---

The day's activity showed Git's development progressing steadily across both user-facing features and internal refactoring, with particular focus on partial clone optimizations and the ongoing architectural effort to eliminate global state. Several long-running threads reached completion while new contributors continued to engage with the project through GSoC proposals and focused bugfixes.
# The Git Mailing List Daily Digest -- 2026/07/04

## The day in brief.

A quiet Saturday on the list (17 emails in 8 threads), but with two long-running series reaching key milestones: Pablo Sabater’s `git log --graph` cascading indentation feature lands its architecturally resolved v7, and Johannes Schindelin’s Coverity-driven leak fixes address all review feedback. The day’s most consequential news is the v7 submission of the graph indentation series, which is now technically sound and ready for final review.

---

## Notable threads

### **Cascading indentation for `git log --graph` lands v7**
Pablo Sabater’s feature series implementing cascading indentation for visual roots in `git log --graph` reached its seventh iteration, replacing the earlier peek-based abstraction with Kristofer Karlsson’s lookahead buffer design. The redesign resolves the core fragility of earlier versions by ensuring commits are fully simplified before the graph layer inspects them, eliminating race conditions in the revision walker. The series now consists of three patches: a test infrastructure refactoring (unchanged from v6), the lookahead buffer implementation, and the visualization logic itself.

The lookahead buffer is small (two commits), deterministic (timestamps are tweaked to guarantee stable output), and compatible with Jeff King’s planned removal of `revs->commits`. The visualization logic indents visual roots (except the first in a cascade) by two characters per level and connects them to their children with an edge on a new row. All three `test_expect_failure` cases from v6 now pass, and the test coverage includes skewed merges and history simplification. One documented edge case remains—redundant indentation when filtered parents are present—but it is marked NEEDSWORK and deemed acceptable for now.

Junio Hamano and Phillip Wood’s review of the visualization logic is the next step. The series is now technically sound and ready for integration, pending final sign-off.

---

### **Coverity-driven leak fixes converge**
Johannes Schindelin’s 13-patch series addressing Coverity-flagged resource leaks and error-path bugs reached convergence, with all substantive review feedback addressed. The series targets memory, file-descriptor, and process-handle leaks across core Git, including fixes for `load_one_loose_object_map` (loose.c), `start_command` (run-command.c), and the fsmonitor daemon’s startup sequence.

Key resolutions include:
- **Patch 1/13**: Replacing an unsafe `errno`-based error check with `ferror(fp)` to reliably detect file-reading errors, addressing Junio Hamano’s correctness concern.
- **Patch 5/13**: Dropped entirely after Patrick Steinhardt identified it as ineffective; the leak was already fixed by an earlier commit.
- **Patch 7/13**: Clarified that ownership of allocations transfers to the caller, resolving a completeness concern.
- **Patch 8/13**: Consolidated cleanup into a single exit path to eliminate a double-free risk, addressing Patrick’s maintainability feedback.

Junio has already signaled he will queue the series for merging, and the remaining patches carry Reviewed-by tags from Patrick Steinhardt and Jeff King. The series is now ready for integration, with no open technical questions.

---

### **Rustification build system adjustments finalized**
Shardul Natu and Koji Nakamaru’s two-patch series refining Git’s build system for Rust components on macOS reached its fourth iteration. The series resolves a parallel build race condition in `git-credential-osxkeychain` and adds support for macOS Universal Binaries via the `RUST_TARGETS` environment variable. The latter allows users to specify multiple target triples (e.g., `aarch64-apple-darwin x86_64-apple-darwin`), compiling separate static libraries per target and combining them into a single Universal Binary using `lipo`.

Junio Hamano’s requested style tweaks (removing `@` from `mkdir` and replacing `[` with `test`) are incorporated, and Patrick Steinhardt’s earlier concerns about Universal Binary completeness are resolved in principle. The series is now converged and ready for integration, with no substantive objections remaining.

---

### **Promisor remote enhancements documentation tweak**
Kristoffer Haugsbakk noted a stylistic preference in the AsciiDoc formatting of Christian Couder’s now-merged promisor remote auto-configuration series. The patch uses an unordered list to describe the four possible values for `promisor.acceptFromServer`, but Kristoffer suggests a description list might be more idiomatic for Git’s config documentation. The comment is purely presentational and carries no technical weight, as the patch itself has already been merged into `next`.

---

## In brief

**`git rev-parse --parseopt` exit code fix clarification** -- Jeff King (Peff) closed the loop on Junio Hamano’s explanation of why the Perl version check (`$SVN::Core::VERSION gt '1.1.0'`) behaves unexpectedly: Perl treats dotted version strings as ASCII character sequences, not numeric values. The follow-up patch to simplify the `SVN` test prerequisite remains the next step.

**Memory leak fixes in `format-patch` and test harness** -- Karthik Nayak pointed out a minor typo in Jeff King’s cover letter ("Kaartic" instead of "Karthik"), but the thread’s substantive discussion (leak fixes, CI improvements) remains unchanged.

**CI PID limit adjustment for private repositories** -- Johannes Schindelin sent a v2 update correcting the commit message’s framing ("raise" instead of "reduce") for his patch adjusting GitHub Actions workflows to address PID limits in private repositories. The technical approach (conditional container options) remains uncontested.

**macOS UTF-8 filename crash fix** -- Ihar Hrachyshka’s v2 patch replaces the Perl-based test with a shell-only version using `printf` and `tr`, addressing Junio Hamano’s procedural concern. The core fix (converting `d_name` to a flexible array member) remains unchanged and ready for merging.

---

## On the radar

**`git rev-parse --parseopt` refactoring gap** -- Jeff King’s proposed fix for `usage_with_options_internal()` to consistently return `PARSE_OPT_HELP_ERROR` remains unaddressed. The change is not urgent but would improve the refactoring’s consistency.
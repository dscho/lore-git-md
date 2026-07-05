# The Git Project Mailing List Daily Digest
**2026/07/04 (Saturday)**

**The day in brief**
A quiet Saturday on the list—17 emails across 8 threads—but with two significant patch series landing: Pablo Sabater’s **cascading indentation for `git log --graph`** (v7) and Johannes Schindelin’s **Coverity-driven resource leak fixes** (v1 with follow-ups). The day’s traffic was dominated by follow-up clarifications and minor revisions rather than new proposals, with no major controversies or surprises.

---

## Notable threads

### Cascading indentation for `git log --graph` lands in v7
Pablo Sabater’s long-running series implementing **cascading indentation for visual roots in `git log --graph`** reached its seventh iteration, replacing the earlier peek-based abstraction with Kristofer Karlsson’s **lookahead buffer** design. The architectural shift—now approved by Junio Hamano—resolves the core fragility of peeking at unsimplified commits by maintaining a two-entry buffer populated by `get_revision_internal()`. This ensures commits are fully processed before the graph layer inspects them, eliminating the nondeterminism that blocked earlier versions.

The series is now technically complete: all three `test_expect_failure` cases from v6 now pass, and the test coverage includes skewed merges and history simplification. One documented edge case remains (redundant indentation when filtered parents are present), but it is marked NEEDSWORK and deemed acceptable for now. Junio’s lingering concern about test brittleness (exact output matching) is mitigated by the lookahead buffer’s deterministic behavior. The next step is final review of the visualization logic in patch 3/3, which implements the actual indentation rules. The series is on track for inclusion once Junio and Phillip Wood sign off.

---

### Coverity-driven leak fixes converge
Johannes Schindelin’s **13-patch series plugging resource leaks and error-path bugs** flagged by Coverity Scan saw follow-up activity addressing the two substantive review concerns. Patch 1/13, which Junio Hamano identified as carrying a **critical correctness bug** (unsafe `errno` checks), will be revised to use `ferror(fp)` for reliable error detection. Patch 5/13, which Patrick Steinhardt found ineffective (the `dpath` variable was never assigned), will be dropped entirely—its leak was already fixed by an earlier commit. The remaining patches (2–4, 6–13) carry Reviewed-by tags or have resolved their review concerns, and Junio has already signaled he will queue the series for merging. The fixes touch a wide range of subsystems (loose objects, run-command, submodules, reftable, fsmonitor) but are purely internal correctness improvements with no user-visible behavior changes.

---

### macOS Universal Binary support for Rust components
Shardul Natu and Koji Nakamaru’s **build system adjustment for Rustification on macOS** reached its fourth iteration, converging on a design that enables **Universal Binary support** for Rust components via the `RUST_TARGETS` environment variable. The series also fixes a parallel build race condition in the `git-credential-osxkeychain` helper. The technical approach—compiling separate static libraries per target triple and combining them with `lipo`—bridges the gap between Apple’s C toolchain (which natively supports multi-architecture builds) and Cargo (which requires separate invocations). Junio’s requested style tweaks are incorporated, and Patrick Steinhardt’s earlier concerns about Universal Binary completeness are now resolved in principle. The series is ready for integration, with no substantive objections remaining.

---

### Promisor remote enhancements: post-merge documentation tweak
Kristoffer Haugsbakk’s **documentation review comment** on Christian Couder’s now-merged promisor remote auto-configuration series was the day’s only activity in this thread. The comment suggests replacing an unordered list with a description list for the `promisor.acceptFromServer` config values, a purely stylistic observation with no technical implications. The patch itself has already been merged into `next`, so this is effectively a post-approval nit. The thread’s substantive work—URL-based auto-configuration and Large Object Promisors (LOP)—is complete, with the LOP series expected to merge imminently.

---

## In brief

**`git rev-parse --parseopt` exit code fix** -- Jeff King (Peff) and Junio Hamano closed the loop on the Perl version-check bug in the `SVN` test prerequisite, confirming the root cause (dotted version strings compared as ASCII) and agreeing to remove the version check entirely. The series, which standardizes help-flag exit codes across parse-options users, is fully merged.

**macOS precompose_utf8 crash fix** -- Ihar Hrachyshka’s v2 patch addressing a crash in `git status` with long UTF-8 filenames now uses a shell-only test (replacing Perl with `printf` and `tr`), addressing Junio’s procedural concern. The fix remains confined to the macOS Unicode normalization layer and is ready for final review.

**CI PID limit adjustment** -- Johannes Schindelin’s patch raising Docker container PID limits for private GitHub repositories saw a v2 update correcting the commit message’s framing ("raise" instead of "reduce"). The technical approach (conditional container options) is uncontested, and the patch is ready for integration.

**Leak fixes in `format-patch` and test harness** -- Karthik Nayak’s typo correction in Jeff King’s cover letter was the day’s only activity in this thread. The series itself—fixing leaks in `git format-patch` and LSan output redirection—is fully converged and ready for merging.

---

## On the radar
None today. The day’s traffic was dominated by follow-ups to existing threads, with no new proposals or stalled topics requiring attention. The two most significant series (cascading indentation and Coverity fixes) are both in their final review stages.
# Git Mailing List Digest - 2025/07/21

**The day in brief.** A moderately busy Monday with 36 emails across 16 threads, dominated by final refinements to the promisor-remote protocol extension series. Key developments include Christian Couder's v6 submission of the promisor-remote enhancements, performance analysis in the xdiff Rust optimization thread, and several features nearing merge readiness.

## Notable threads

**Promisor-remote protocol finalized** -- Christian Couder submitted v6 of his 5-patch series extending the promisor-remote protocol to support configurable validation of remote attributes. The series allows servers to advertise additional fields (like `partialCloneFilter` and `token`) via `promisor.sendFields`, with clients validating these against local config using `promisor.checkFields`. Today's discussion focused on documentation polish and terminology, with Junio Hamano questioning whether "field" terminology conflicts with Git's established "configuration variable" concepts. The series appears technically complete, with all major implementation questions resolved and only naming considerations remaining before merge.

**xdiff optimization tradeoffs** -- Phillip Wood presented benchmark results showing that a 15% performance improvement in xdiff can be achieved through C optimizations alone (8% from whitespace flag check refactoring, 6% from switching to XXH3 hash), matching the gains claimed for the Rust implementation. Junio Hamano noted this raises questions about whether introducing Rust as a hard dependency is justified when similar speedups are possible in C. The discussion turned to platform-specific performance, particularly on architectures without optimized SIMD implementations.

**IMAP folder archiving naming debate** -- Aditya Garg's feature to archive sent emails in IMAP folders via `git send-email` reached its final naming discussion. Junio Hamano suggested more specific terms like "sent" or "fcc" (file carbon copy) instead of the current generic `sendemail.imapfolder` naming, arguing this would better reflect the feature's purpose while leaving room for future IMAP-related enhancements. The implementation itself appears technically sound and ready for merge pending this naming decision.

**`git bisect` interface improvements** -- Following a user report of confusion with `git bisect skip`, Junio Hamano and Christian Couder discussed potential interface enhancements. Ideas included adding confirmation prompts when reclassifying commits and enforcing consistency checks in bisect refs to prevent marking the same commit as both good and bad. While no concrete patches emerged, the discussion identified clear opportunities to make this powerful but sometimes confusing feature more user-friendly.

**`git repo` output formats finalized** -- Lucas Seiki Oshiro and Justin Tobler converged on an output format design for the new `git repo` command's `info` subcommand. They agreed on `porcelain` (default), `keyvalue`, and `null` formats, with `-z` as an alias for null format. The discussion also touched on automatic format switching when piping (similar to `git diff`'s color behavior), though they leaned toward explicit format selection for now.

## In brief

**Diff context configuration** -- Junio Hamano indicated the v4 changes to Phillip Wood's diff context configuration series look good and will be merged to 'next' soon. The series provides consistent diff context configuration for all interactive patch commands.

**MemorySanitizer false positives** -- Jeff King and Karthik Nayak discussed strategies for handling MSan warnings from cryptographic operations, concluding that proper instrumentation of system libraries is the most robust solution despite workarounds being available.

**Test reorganization** -- A 2-patch series finalized the help output test strategy, implementing automated outside-repo testing via `--list-cmds` and properly organizing repository-dependent tests with their command-specific test files.

**`--early-output` removal** -- Junio Hamano acked Jeff King's patch removing the long-unused `--early-output` option from the revision machinery, eliminating 154 lines of untested code.

**Rebase documentation confusion** -- Rodney Bates pointed out a contradiction in `git help rebase` between the text description and diagram showing which branch gets modified. Nico Williams clarified the terminology around "reset" in this context.

## On the radar

**Git for Windows permissions** -- James Gao reported serious permission errors in Git 2.50.1 on Windows 11 Pro preventing basic operations like `init` and `add`, suggesting a potential regression in file handling.

**Reflog memory leak** -- A v3 patch addresses a memory leak in reflog expiration code, adding proper cleanup of allocated `reflog_expire_entry_option` structs. The fix is straightforward and appears ready for merge.
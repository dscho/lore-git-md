# Git Mailing List Digest - 2025/07/16

**The day in brief.** A busy Wednesday with 75 emails across 19 threads saw significant progress on several fronts. The `git last-modified` series reached technical completion, the pseudonym policy documentation was finalized, and performance optimizations for commit traversal were validated. Meanwhile, discussions continued on configuration validation architecture and the `git repo` command's design.

## Notable threads

### `git last-modified` reaches final review

The v5` series introducing a new `git last-modified` plumbing command for tracking tree-level modifications appears technically complete after addressing all major feedback. The command provides forge-friendly modification tracking with Bloom filter optimizations (50% speedup for top-level checks) and an `--extended` output format. Junio raised final design questions about pretty-printing behavior in the extended format, particularly around message termination handling in both newline and NUL-delimited modes. The series awaits resolution on these output format details before merging.

### Pseudonym policy finalized

After six review cycles, the documentation patch establishing Git's policy for pseudonyms in Signed-off-by trailers received final approvals from Jeff King, Patrick Steinhardt, and Junio Hamano. The policy explicitly allows distinctive, non-misleading pseudonyms while prohibiting anonymous contributions, aligning with Linux kernel precedent. This concludes a lengthy discussion balancing legal requirements (DCO compliance, OFAC sanctions) with contributor inclusivity.

### Commit traversal optimization validated

René Scharfe's series converting commit traversal from linked lists to priority queues received thorough technical validation, confirming it maintains stable ordering for commits with identical timestamps while improving worst-case performance from O(n²) to O(log n). Jeff King analyzed the tie-breaking behavior, verifying the new implementation matches the original's FIFO ordering. The optimization particularly benefits operations like `:/` rev-parse and fetch's mark_recent_complete_commits() with many refs.

### `git repo info` output format debate

The design discussion around `git repo info`'s output format continued, with Patrick Steinhardt proposing `--format=key-value` and `--format=nul` options instead of Justin Tobler's suggested `-z` flag for NUL-termination. The debate centers on balancing Git's CLI conventions against future extensibility needs (like potential JSON output). Meanwhile, Junio contributed style cleanups to the implementation, standardizing function parameter formatting and comment style in `builtin/repo.c`.

### Config validation architecture proposal

Lidong Yan proposed a registration-based config validation system where subsystems explicitly declare their known keys and validation functions. The design aims to distinguish core Git config from third-party extensions while catching typos. Johannes Sixt countered that some issues (like the undocumented `pull.autostash`) might be better addressed through documentation improvements rather than technical enforcement. Junio later analyzed deeper architectural questions about configuration inheritance between commands, suggesting Git 3.0 as a potential milestone for breaking changes.

## In brief

**CI test failure reporting fix** -- Junio Hamano corrected a shell syntax error in GitHub Actions test artifact collection that caused "Bad substitution" errors.

**Comment character bugfix series complete** -- Ayush Chandekar's fixes for `core.commentChar="auto"` handling received Phillip Wood's sign-off, with the series now awaiting integration either directly or via Wood's deprecation branch.

**C99 bool standardization finalized** -- Phillip Wood's series formalizing `bool` usage for predicate functions concluded with lighthearted typo corrections ("Guildlines" -> "Guidelines") and a reference to "Muphry's law" about error-containing corrections.

**git-gui SHA-256 support** -- Takashi Iwai's v3 series adding SHA-256 repository support to git-gui was queued by maintainer Johannes Sixt after minor fixups to comments and git command invocation.

**PCRE2 build fixes** -- The meson build system's handling of broken macOS PCRE2 installations was refined with header verification checks, though a late-arriving fix for pkgconf module detection may follow as incremental work.

## On the radar

**Compound literals evaluation** -- The documentation patch establishing a moratorium on new C99 compound literal usage until mid-2026 awaits finalization, with ongoing discussion about resource cleanup patterns.

**Byte-swapping optimizations** -- Sebastian Andrzej Siewior's series standardizing ntohl/ntohll handling is in final review with only typo fixes since v2, likely to merge soon.
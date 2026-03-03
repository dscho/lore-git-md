Here's the Git mailing list daily digest for May 12, 2025:

## The day in brief

May 12, 2025 saw active development across multiple fronts, with 126 emails across 28 threads. The day was dominated by technical discussions around several major topics: finalizing the `whatchanged` deprecation series, resolving design questions in the stash import/export feature, and addressing platform-specific behavior in `git send-email`'s FQDN detection. Notable progress was made on the Change-ID standardization effort with substantive technical exchanges between Junio Hamano and Martin von Zweigbergk.

## Notable threads

**Change-ID standardization advances** -- Junio Hamano and Martin von Zweigbergk exchanged detailed technical proposals about predecessor tracking in the Change-ID system. Junio suggested making predecessor pointers optional during transfer (similar to promisor/partial clone objects) and allowing multiple predecessors to accommodate squashing workflows. Martin clarified Jujutsu's implementation details while showing willingness to adapt the design for Git compatibility. Later, Brian m. carlson raised security concerns about Jujutsu's random Change-ID generation, proposing deterministic alternatives with opt-out configuration.

**Stash import/export nears completion** -- After extensive review, the stash import/export feature received final technical approvals with only minor test case adjustments remaining. Phillip Wood contributed validation improvements ensuring proper topology checks for imported stashes. The thread resolved naming convention questions raised by Jeff King's upcoming ref restrictions, confirming the implementation's use of `refs/stash-export/` is compliant. Junio Hamano humorously debated the feature's necessity before concluding the dedicated commands provide meaningful UX improvements over manual ref transfers.

**FQDN detection platform debates** -- A lengthy discussion unfolded about cross-platform behavior of `git send-email`'s FQDN detection. After initial patches switched from `hostname --fqdn` to `-f` for broader compatibility, Junio Hamano expressed concerns about semantic ambiguity in the `-f` flag. The thread converged on a platform-specific approach using `--fqdn` for Linux and `-f` for macOS/Darwin, with documentation to clarify the intentional differences. Eric Sunshine provided valuable context about BSD lineage systems' consistent `-f` behavior, helping justify the compromise solution.

**Scalar maintenance configuration finalized** -- Derrick Stolee and Junio Hamano reached consensus on simplifying the `scalar reconfigure` interface from a tri-state model to a boolean flag with "keep" as default behavior. The discussion revealed the command's primary use case is scheduler updates (like fixing macOS quirks) rather than fundamental configuration changes. The series is now ready with clear documentation of valid modes following established patterns from similar Git options.

**In brief**

**SHA-256 support for gitk** -- Takashi Iwai continued addressing Johannes Sixt's review feedback, clarifying design decisions around hash length preferences and fixing remaining hardcoded SHA-1 assumptions in five procedures.

**Reftable performance fix** -- A patch from shejialuo addressed a significant performance regression in reftable block reading, making `update-ref` operations 1.62x faster by avoiding unnecessary block decoding.

**Stash branch display fix** -- K Jayatheerth diagnosed and fixed a bug where `list` showed submodule branch names instead of superproject branches, using `xstrdup` to preserve the correct branch name through submodule operations.

**Mailinfo memory leaks** -- Lidong Yan's v3 patch series fixed resource leaks in email header decoding by moving to caller-allocated strbufs, incorporating Junio Hamano's cleanup path suggestions.

**OIDMap refactoring** -- A cleanup series renamed `oidmap_free()` to `oidmap_clear()` for accuracy, added size query functionality, and removed redundant pointer indirection in the replace-objects implementation.

**On the radar**

**`contrib/` directory cleanup** -- Patrick Steinhardt's systematic removal of unmaintained components advanced to v3, though Phillip Wood made a case for keeping `git-contacts` due to its role in contributor onboarding.

**Stash default behavior debate** -- Junio expressed reservations about changing `git stash apply`'s default to restore the index, citing personal experience that the current behavior works better for common workflows. This discussion may resurface when the breaking changes proposal advances.
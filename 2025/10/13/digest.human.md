# Git Mailing List Digest - 2025/10/13

**The day in brief.** A busy Monday with 85 emails across 32 threads saw significant progress on several fronts. The standout developments include the finalization of atomic reference updates for `git replay`, advancement of SHA-1/SHA-256 interoperability work, and a new security series hardening Git against malicious hooks. Policy discussions around AI-generated contributions and the ongoing `git whatchanged` deprecation also generated notable discussion.

## Notable threads

### Atomic reference updates for `git replay` finalized

Siddharth Asthana's series to make `git replay` perform atomic reference updates by default reached its final form after multiple review iterations. The implementation now uses Git's ref transaction API to ensure all-or-nothing behavior while maintaining backward compatibility through a new `--update-refs=<mode>` option (with "yes" for atomic updates and "print" for traditional pipeline output). A companion `replay.defaultAction` config option allows users to set their preferred default mode. The series has addressed all technical feedback, with only minor naming suggestions from Junio Hamano remaining as open items. The comprehensive test suite verifies atomic behavior through lock file checks and includes thorough coverage of config parsing and CLI override scenarios.

### SHA-1/SHA-256 interoperability moves to 'next'

Junio Hamano proposed moving Brian M. Carlson's 9-patch SHA-1/SHA-256 interoperability series to the 'next' branch after a weekend without new feedback. The series includes foundational documentation updates (pack format specifications, loose object format specs), a new `rev-parse --show-object-format=compat` plumbing command, and enhanced `fsck` validation for GPG signature headers. With only minor documentation tweaks since v2 and no outstanding technical concerns, this represents significant progress for the multi-year effort to enable hash algorithm interoperability.

### Security series hardening against malicious hooks

Michael Lohmann submitted a 5-part series addressing arbitrary code execution via Git hooks in untrusted repositories. Building on prior `safe.directory` work, the patches introduce both temporary (`--allow-unsafe` flag) and persistent (`safe.assumeUnsafe` config) mechanisms to prevent automatic trust of repositories owned by the current user. The implementation includes comprehensive tests and thorough documentation of the security implications. The series follows Git's standard practice of separating refactoring (function renaming, check reordering) from behavior changes, with all patches now in their second iteration after incorporating documentation feedback.

### AI contribution policy debate continues

The ongoing discussion about AI-generated contributions saw Junio Hamano and Christian Couder debate policy strictness levels. While acknowledging Couder's concerns that overly firm wording might discourage legitimate uses, Hamano maintained the project must consider legal risks when distributing potentially infringing content. The exchange highlighted practical enforcement challenges, particularly for simple fixes where contributors may not even be aware of AI involvement. The thread remains at v2 with no clear resolution yet, though Hamano expressed a slight preference for keeping his SFC-lawyer-vetted version in the tree as better than no policy at all.

### `git whatchanged` deprecation gathers user feedback

Multiple users reported continued reliance on the deprecated `git whatchanged` command, with Roy Lomicka sharing a PowerShell script that uses it to set file modification times based on commit timestamps. Kristoffer Haugsbakk provided migration guidance, noting `git log --no-merges --raw` produces equivalent output except for empty commits. The thread serves as a data-gathering exercise for the deprecation process, demonstrating at least some real-world use cases that would need to be accommodated if the command is removed.

## In brief

**Reftable optimization infrastructure** -- Karthik Nayak and Patrick Steinhardt finalized a 9-part series introducing a `--required` flag (name still under discussion) to check if reference optimization is needed without performing it, primarily for GitLab's Gitaly service.

**Fast-import/fast-export signature handling** -- Christian Couder completed implementation of symmetric `--signed-tags=<mode>` support across both commands, enabling consistent handling of PGP, X.509, and SSH signatures during repository filtering operations.

**Documentation formatting fixes** -- Jeff King's fixes for nested list rendering issues in reftable format documentation were merged into 'next', resolving rendering problems across multiple man pages.

**Windows include style debate** -- Johannes Sixt and Junio Hamano continued discussing project-relative vs. relative include paths in Windows-specific code, with Hamano suggesting pragmatic acceptance of the current patch despite philosophical differences.

**Test modernization** -- A small patch updated `t2401-worktree-prune.sh` to use modern test helpers (`test_path_is_file`, `test_path_is_dir`) instead of direct shell checks, continuing the test suite cleanup effort.

**Patch-id hash algorithm enforcement** -- An Outreachy participant clarified that `git patch-id` must always use SHA1, removing outdated code and comments while explicitly setting the hash algorithm for consistency with documentation.

## On the radar

**Rust CI integration** -- Patrick Steinhardt's `ps/ci-rust` series adding Windows CI support for Rust appears ready to progress after Junio confirmed no outstanding review feedback.

**Worktree documentation** -- Eric Sunshine blocked merging documentation improvements for `git worktree add` to 'next', citing insufficient justification for examples and incomplete cross-references.

**Const-correctness in hashmap API** -- A discussion about const handling in the patch-id subsystem expanded into broader questions about the hashmap API's design, with Junio Hamano requesting deeper analysis before removing a NEEDSWORK comment.
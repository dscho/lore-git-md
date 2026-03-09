# Git Mailing List Digest - 2025/12/02

**The day in brief.** A busy Tuesday with 67 emails across 24 threads, dominated by two major technical discussions: the ongoing security debate around ANSI escape sequence handling and significant progress on Patrick Steinhardt's `git-history` command series. Other notable developments include documentation updates for Git's hash algorithm defaults and a new debugging feature for lock files.

## Notable threads

**Security debate intensifies on terminal control sequences**  
Johannes Schindelin escalated the discussion about ANSI escape sequence handling in Git's sideband channel with detailed technical arguments against allowing certain control sequences. He demonstrated concrete examples of dangerous sequences like `OSC P 1 0 ; ? ST` that can query terminal state, arguing applications (not terminal emulators) should sanitize untrusted input. The debate remains polarized between those wanting to preserve terminal capabilities and those prioritizing security, with Schindelin proposing a prototype patch for more granular control via `sideband.allowControlCharacters`. This thread has become one of the most technically substantive security discussions in recent months.

**git-history command approaches completion**  
Patrick Steinhardt's v6 series implementing the experimental `git history` command saw extensive review discussion today, particularly around the `split` and `reword` subcommands. Key developments include refined commit collection logic using `--ancestry-path`, intentional branch-only modification behavior, and preservation of commit headers via `commit_tree_extended()`. The thread also revealed interesting workflow benefits from D. Ben Knoble's testimonial about the utility of `drop` and `reorder` subcommands from earlier versions. With most technical feedback addressed, this series appears ready for experimental merging.

**Ref storage backend design consensus emerges**  
The reference storage backend thread reached significant consensus today, with Junio Hamano expressing approval for Patrick Steinhardt's config-based URI syntax proposal (`extension.refStorage`). Karthik Nayak confirmed plans to implement this in v4, including renaming `GIT_REF_URI` to `GIT_REFERENCE_BACKEND` for clarity. The discussion solidified key principles: treating storage locations as backend-specific opaque strings and using config as the primary interface with environment variables as overrides. This represents a major step toward flexible reference storage in Git.

**git replay gains --revert capability**  
A new patch series added `--revert` mode to `git replay`, implementing server-side reversal of commits. The v2 iteration addressed maintainer feedback by making it a standalone operation mode (mutually exclusive with `--onto` and `--advance`) and extracting revert message formatting into a shared helper function. The implementation carefully preserves individual commit messages while reversing changes, with thorough test coverage of basic reverts, reapply behavior, and conflicts. This functionality is particularly targeted at GitLab's Gitaly service needs.

**Documenting hash algorithm defaults**  
Brian m. carlson proposed a 4-patch series updating documentation to reflect that SHA-256 becomes the default hash algorithm when Git is compiled with `WITH_BREAKING_CHANGES`. The changes affect `git`, `git-init`, `git-index-pack`, and `git-show-index` man pages. The thread explored three technical approaches: direct conditional blocks in each man page (original patches), centralized Asciidoc attributes (proposed by Junio Hamano and Patrick Steinhardt), and build system variables (suggested by Brian). This documentation work prepares for Git 3.0's SHA-256 transition.

**Lockfile debugging feature proposed**  
Paulo Casaretto introduced an optional debugging feature that creates `.lock.pid` files to track which process holds Git locks. Enabled via `GIT_LOCK_PID_INFO=1`, the feature enhances error messages to distinguish between active locks and stale ones from crashed processes. The implementation leverages Git's tempfile infrastructure for cleanup and includes comprehensive tests. D. Ben Knoble later suggested making this configurable via Git config in addition to environment variables, sparking discussion about whether the feature should remain opt-in.

## In brief

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**gitk window layout fixes** -- A patch addresses two pane resizing issues in gitk's Tcl/Tk interface, ensuring sash positions persist across window resizing and preventing tiny panes when moving between screens.

**Meson cross-compilation improvements** -- Toon Claes submitted patches to fix ICONV_OMITS_BOM detection during cross-compilation and ignore Meson wrapper lock files.

**Scalar config documentation finalized** -- The series documenting Scalar's configuration settings reached completion after addressing subtle interactions between `index.threads` and `index.recordOffsetTable`.

**Branch advice modernization** -- Kristoffer Haugsbakk updated branch-related error messages to use `git help` instead of `man`, with expanded rationale about platform-agnostic benefits.

**Outreachy intern introduction** -- Olamide Bello announced joining the `the_repository` removal effort as an Outreachy intern, receiving warm welcomes from community members.

## On the radar

**git-last-modified recursive behavior** -- The discussion shifted from documentation to fundamental design questions about whether the command should default to recursive path handling while still marked EXPERIMENTAL.

**Windows test compatibility** -- Junio Hamano and Johannes Schindelin discussed broader testing philosophy around platform-specific compromises, suggesting potential for categorizing tests by platform requirements.
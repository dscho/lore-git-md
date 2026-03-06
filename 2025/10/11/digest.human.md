# Git Mailing List Digest — 2025/10/11

## The day in brief  

A moderately active day with 24 emails across 13 threads, featuring documentation refinements, CI improvements, and ongoing mentorship discussions. The standout items include Junio Hamano's "What's cooking" status update and substantive feedback on several documentation patches. The Outreachy program remains active with new applicants and mentoring exchanges.

## Notable threads  

### Documentation refinements for deprecated options  

Johannes Sixt provided detailed feedback on Kristoffer Haugsbakk's proposed warning text for the deprecated `--committer-date-is-author-date` option in `git-am`. The discussion focuses on making the deprecation warning more forceful and user-visible in the man pages, with suggestions to either use "WARNING" instead of "NOTE" or restructure the documentation flow. This represents the final polish on an already-approved deprecation path that will keep the option available in `git-am` while removing it from `git-rebase`.

### Worktree documentation improvements face pushback  

Eric Sunshine offered substantive review of Michal Suchánek's patches improving `git worktree` documentation, particularly objecting to a proposed warning against nested worktree creation. Sunshine argues there's no technical justification for the warning beyond subjective aesthetic concerns about gitignore noise, suggesting the patch either needs stronger rationale or should be dropped. The parallel patch about bare repository usage received more constructive feedback focused on making examples clearer and more discoverable.

### Credential helper Makefile standardization nears completion  

Thomas Uhle and Junio Hamano finalized discussions about standardizing Makefiles for Git's credential helpers in the contrib directory. The thread resolved remaining questions about variable naming conventions (`$(MAIN)` vs spelled-out targets) and patch organization, with Hamano suggesting focusing first on functional changes (adding install targets) before any macro cleanups. The patches are now queued in 'seen' with only minor stylistic decisions remaining.

### GPG test infrastructure issues persist  

Todd Zullinger provided additional context about flaky GPG2-related tests in t1016-compatObjectFormat.sh, noting they consistently fail in CI environments but rarely fail locally. The discussion highlights a broader test infrastructure challenge where GPG2 prereqs silently skip tests when the GPG home directory doesn't exist. Junio's proposal to leave the broken tests in 'seen' to attract debugging attention appears to be the current approach, acknowledging the limited availability of experts who might diagnose the issue.

### "What's cooking" status update  

Junio Hamano's periodic development report covered numerous ongoing efforts including documentation modernization, CI improvements, ref backend optimizations, and new features like the `git history` command. Notable technical points include the beginning of SHA1-SHA256 interoperability work and multiple series removing `the_repository` global usage. The update serves as both progress report and coordination mechanism, with maintainer commentary on each topic's readiness for integration.

## In brief  

**`stash.index` documentation finalized** -- D. Ben Knoble and Kristoffer Haugsbakk concluded discussion on documenting how the config affects `--autostash` operations, with Junio weighing in on commit reference style and illustrative versus exhaustive command listings.

**Windows include path cleanup** -- Matthias Aßhauer noted two potentially missed cases in Johannes Schindelin's patch converting relative includes to project-relative paths in `compat/mingw.c`.

**Clang warning suppression consensus** -- Patrick Steinhardt confirmed agreement with Johannes Schindelin's approach to handling `-Wunreachable` warnings in `refs/files-backend.c` by applying workarounds at the callsite rather than macro definition.

**Ubuntu sudo-rs CI workaround** -- Patrick Steinhardt explained the current detection logic for falling back to traditional sudo in CI, noting Ubuntu has dropped 32-bit support and no other distros currently face this implementation split.

**Outreachy comment improvements** -- Okhuomon Ajayi submitted a documentation patch clarifying commit re-parsing behavior in `commit.c`, with mentoring feedback about commit message standards from Usman Akinyemi.

## On the radar  

**Rust CI improvements** -- Patrick Steinhardt noted his `ps/ci-rust` series may be ready for progression despite being marked "Expecting an update" in Junio's report, having incorporated Windows support and other improvements.

**Optional path configuration** -- D. Ben Knoble flagged possible test improvements needed for the `:(optional)` pathname prefix support, though Junio suggested handling these as follow-up patches rather than a full reroll.
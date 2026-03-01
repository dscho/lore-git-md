# Git Mailing List Digest — 2025/01/02

## The day in brief

A moderately active day with 31 emails across 10 threads, featuring continued discussion of LSan race condition fixes, build system portability issues, and several technical questions about Git behavior. The most notable developments include Junio Hamano's endorsement of a minimal approach to handling LSan false positives and multiple reports of shell portability problems in the meson test infrastructure.

## Notable threads

### LSan race condition solution finalized

The long-running discussion about handling LSanitizer (LSan) race conditions in threaded test execution reached a conclusion today. Jeff King (Peff) and Junio Hamano converged on a minimal approach that filters false positives from test output rather than modifying Git's threading code to accommodate LSan's limitations. This follows several iterations of attempted solutions, including a now-reverted barrier-based approach. The thread also sparked an interesting side discussion about improving Git's tooling for reverting multiple commits, with Jeff proposing a "squash revert" feature that would generate more informative commit messages when undoing complex changesets.

### Meson build system portability issues

Multiple contributors reported problems with bash-specific process substitution syntax (`<(...)`) in the meson test infrastructure, which fails on shells like dash that don't support this feature. The issue was introduced in Patrick Steinhardt's meson integration series in late December and wasn't caught during review due to subtle shell configuration differences in test environments. Junio Hamano acknowledged this as an oversight that should have been caught, and the thread explored various solutions including making the syntax more portable or explicitly requiring bash. The discussion revealed important details about how shell specifications propagate (or fail to propagate) through make's variable inheritance system.

### Git grep architecture discussion

A feature discussion emerged about integrating alternative grep tools like ugrep` into Git's command set. Matěj Cepl and Jonathan Nieder clarified that Git's grep functionality is implemented internally in grep.c rather than shelling out to an external command, making simple command substitution approaches ineffective. The thread is currently in a requirements-gathering phase, with Nieder asking for more concrete use cases to determine whether the solution would require changes to Git's internals, new subcommands, or modifications to external tools.

## In brief

Build system: Jonathan Nieder confirmed a fix for meson compatibility with Dash shell in Debian builds, providing a Tested-by tag for Patrick Steinhardt's patch that replaces process substitution with temporary directory usage.

Code hygiene: Junio Hamano engaged in a review of Seija Kijin's patch adding 'U' suffixes to integer literals, questioning whether the changes actually prevent undefined behavior in the cases shown or are primarily stylistic.

Educational: Chris Torek provided a detailed explanation of Git's commit model in response to Prasad Pandit's question about branch behavior, clarifying how commits and branches relate in Git's object model.

Containerization: Jörg Sommer asked about relocating Git's common directory in Docker container setups, specifically how to properly adjust paths for shared worktrees when the repository is mounted at a fixed path.

Bug report: Sarah Gastner reported inconsistent behavior in `git log` date filtering where some commits within specified date ranges are unexpectedly omitted, providing reproducible test cases across multiple platforms.

## On the radar

The meson build system portability issue remains unresolved but well-understood, awaiting input from the original author (Patrick Steinhardt) about preferred resolution direction. The grep integration discussion also awaits further clarification of use cases that would motivate modifying Git's internal grep implementation.
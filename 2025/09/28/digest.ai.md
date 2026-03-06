# Git Mailing List Digest - 2025/09/28

**The day in brief.** A moderately active Sunday with 77 emails across 12 threads, dominated by Taylor Blau's massive 49-patch repack refactoring series. Key developments include the completion of D. Ben Knoble's optional filepath series with Junio's approval, progress on gitk window persistence, and ongoing discussions about patch workflow improvements. The repack modularization work represents the most significant architectural change today.

## Notable threads

### Repack refactoring reaches completion

Taylor Blau concluded his monumental 49-patch series refactoring `builtin/repack.c` into modular components while eliminating global variables. The work systematically extracted functionality into 12 new modules under a `repack.h` API, including dedicated units for cruft packs (`repack-cruft.c`), filtered packs (`repack-filtered.c`), and MIDX handling (`repack-midx.c`). The series maintained strict behavioral equivalence while modernizing the codebase's architecture in preparation for future incremental MIDX functionality. Junio Hamano praised the series' organization in a follow-up email, highlighting its clear documentation of dependencies and positioning within the larger `tb/incremental-midx-part-3` effort.

### Optional filepath support approved

D. Ben Knoble's v2 series implementing optional filepath support (`:(optional)` prefix) for both configuration and command-line arguments received final approval from Junio Hamano. The implementation allows paths to be marked as optional without triggering errors when missing, addressing use cases like commit templates and blame ignore files. Junio confirmed the technical soundness of the approach while disagreeing with the suggestion to integrate this as pathspec magic, noting pathspecs already handle non-matching patterns gracefully. The series builds on Junio's original work and appears ready for merging after addressing minor documentation suggestions.

### Gitk window geometry improvements

A coordinated effort between Michael Rappazzo and Johannes Sixt progressed on making gitk's Tags/Heads window positions persistent across sessions. The v3 patch series combines a revert of an outdated Cygwin Tcl/Tk 8.4.1 workaround with new functionality to properly save and restore window geometry. Technical discussions clarified historical context around the original position-restriction compromise, confirming full restoration is now safe with modern Tcl/Tk versions. The changes affect 51 lines of Tcl code with careful attention to cross-platform behavior, particularly around window manager timing issues on Windows and Gnome.

### Patch workflow policy discussion

Junio Hamano and Taylor Blau continued discussing potential improvements to Git's patch review workflow, focusing on defining qualified reviewers. Taylor proposed a REVIEWERS file approach inspired by GitHub's CODEOWNERS system, though acknowledged Git's smaller scale may not warrant per-file reviewers. Junio raised practical concerns about drive-by acks and rubber-stamping, suggesting tighter language focused on reviewers who actively helped polish patches. The discussion remains exploratory, with no clear resolution on how to implement reviewer qualifications while maintaining contributor participation.

## In brief

**Packfile store refactoring acked** -- Taylor Blau provided the final Acked-by for Patrick Steinhardt's v6 packfile store refactoring series, confirming it's ready for merging into 'next' after addressing all review feedback.

**push.default=simple behavior clarified** -- D. Ben Knoble confirmed Julia Evans' analysis of `push.default=simple`'s dual behavior patterns, noting the current implementation has evolved special handling for default pull remotes while being more permissive for other same-name pushes.

**Documentation formatting fix** -- Jean-Noël Avila's patch fixing Asciidoctor rendering issues for nested list items received maintainer approval after providing concrete examples of malformed output in `git-config` and `git-log` documentation.

**Windows path handling investigation** -- Nikolay Shustov confirmed Eric Sunshine's diagnosis that dot-terminated filenames cause `git merge --no-ff` failures on Windows, noting Cygwin's lower-level APIs handle them correctly.

**git whatchanged migration note** -- Neal Miller shared a real-world example showing some `git whatchanged` aliases can be replaced with identical `git log` flags when using `-p` output, validating the deprecation path's flexibility.

**Credential wildcard matching proposal** -- A follow-up email clarified the need for path-prefix matching in Git credential configuration, proposing implicit trailing slash wildcards to distinguish between personal and enterprise credentials on the same host.

**git-patch-id documentation fix** -- Kristoffer Haugsbakk corrected accidental literal blocks in the `git-patch-id` man page's option descriptions, with Junio Hamano acking the changes after minor clarification about paragraph break behavior.

## On the radar

**Rustification effort** -- While not generating traffic today, Ezekiel Newren's Rust integration work remains an active but contentious topic, particularly regarding platform support concerns raised by Randall S. Becker for NonStop systems.

**Documentation synopsis conversion** -- Jean-Noël Avila's ongoing effort to convert all man pages to consistent AsciiDoc synopsis style continues making progress, with today's formatting fixes aligning with this larger initiative.
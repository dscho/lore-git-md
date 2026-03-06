# Git Mailing List Digest - 2025/09/28

**The day in brief.** A moderately busy Sunday with 77 emails across 12 threads, dominated by Taylor Blau's massive 49-patch series refactoring `builtin/repack.c`. The series represents a major architectural shift, modularizing the repack machinery while eliminating global variables. Other notable activity includes D. Ben Knoble's optional filepath support reaching final approval and ongoing discussions about Git's patch review workflow.

## Notable threads

### Repack machinery refactored for modularity

Taylor Blau's 49-patch series (tb/incremental-midx-part-3) systematically breaks down the monolithic `builtin/repack.c` into focused modules while eliminating `the_repository` and `the_hash_algo` global variables. The changes:

1. **New modules** - Extracts functionality into 12 new compilation units (`repack-cruft.c`, `repack-filtered.c`, etc.) with clear interfaces via `repack.h`
2. **Configuration structs** - Introduces structured parameter passing via `pack_objects_args`, `write_pack_opts`, and other dedicated types
3. **Global elimination** - Removes all `the_repository`/`the_hash_algo` usage through methodical parameterization
4. **Behavior preservation** - Maintains existing functionality while enabling future MIDX work

The series follows a disciplined pattern: identify cohesive functionality, parameterize dependencies, extract to modules with minimal changes. Junio Hamano praised the submission's clarity in declaring dependencies and situating within the larger incremental MIDX effort.

### Optional filepath support finalized

D. Ben Knoble's v2 series implementing `:(optional)` file prefixes for both configuration and command-line paths received final approval from Junio Hamano. The changes allow paths that may not exist without triggering errors, useful for templates and ignore files. Key aspects:

- Only checks for missing (not empty) files when handling optional paths
- Adds `is_missing_file()` utility in wrapper.c
- Documents new syntax in config.adoc and gitcli.adoc
- Maintains backward compatibility

Junio confirmed the series achieves its goal while disagreeing with integrating this as pathspec magic, noting pathspecs already handle non-matching gracefully. The implementation appears ready for merging after addressing minor documentation suggestions.

### Patch review workflow discussions continue

The ongoing thread about improving Git's patch review process saw substantive discussion between Junio Hamano and Taylor Blau about reviewer qualifications. Key points:

- Junio expressed skepticism about defining "qualified reviewers," citing risks of drive-by acks
- Taylor proposed a REVIEWERS file approach, inspired by CODEOWNERS but scaled for Git
- Both agreed maintainer bandwidth remains the central constraint
- Discussion focused on incentivizing quality review rather than redistributing workload

The exchange highlighted practical challenges in formalizing review standards while maintaining Git's open contribution model. No concrete resolution emerged, but the dialogue continues to explore balancing quality with participation.

## In brief

**Packfile store refactoring** -- Patrick Steinhardt's v6 series centralizing packfile management in `struct packfile_store` received final Acked-by from Taylor Blau, completing its path to 'next'.

**gitk window geometry** -- Michael Rappazzo and Johannes Sixt's v3 series makes Tags/Heads window positions persistent, incorporating feedback about cross-platform behavior and reverting an obsolete Cygwin workaround.

**Documentation formatting fixes** -- Jean-Noël Avila and Kristoffer Haugsbakk addressed Asciidoctor rendering issues in `git-patch-id` man pages, with Junio confirming the changes resolve the problematic literal block output.

**Windows path handling** -- Nikolay Shustov confirmed Eric Sunshine's diagnosis that dot-terminated filenames cause `git merge --no-ff` failures on Windows, noting Cygwin's lower-level APIs handle them correctly.

**Credential wildcards** -- A feature proposal for path-prefix matching in Git credential configuration gained clarity with a concrete example distinguishing personal vs enterprise credentials under the same domain.

## On the radar

**`git whatchanged` deprecation** -- Neal Miller's confirmation that some existing aliases can directly substitute `git log` flags may simplify migration paths as the deprecation discussion continues.
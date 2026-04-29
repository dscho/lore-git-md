# Git Mailing List Digest - 2026/04/28 (Tuesday)

**The day in brief.** A moderately busy Tuesday with 57 emails across 14 threads, featuring significant progress on several fronts. The standout developments include the conclusion of a 20-iteration autostash feature, a new `format-rev` command proposal, and Windows-specific fixes for large object handling. Junio Hamano provided key reviews across multiple threads, while Johannes Schindelin dominated the Windows compatibility work.

## Notable threads

### Autostash feature reaches completion

The long-running effort to add autostash behavior to `git checkout -m` has concluded after 20 iterations (14 pre-merge and 6 post-merge). The final patches from Harald Nordgren, reviewed by Phillip Wood and Junio Hamano, implement comprehensive conflict resolution customization including:

- Customizable conflict marker labels
- Silent operation capability for programmatic use
- Robust lockfile cleanup in all error paths
- Stash message customization
- Thorough test coverage across all edge cases

The only remaining architectural consideration - potential improvements to stash message handling interfaces - has been explicitly deferred to future work. This series exemplifies Git's thorough review culture, with each iteration refining both implementation and documentation.

### New `format-rev` command proposed

Kristoffer Haugsbakk pivoted from extending `git name-rev` to proposing a new experimental `git format-rev` command in v3 of his series. The standalone command provides on-demand pretty formatting of revisions with two input modes:

- `--stdin-mode=revs` (one revision per line)
- `--stdin-mode=text` (find commits in freeform text)

The implementation shares infrastructure with `name-rev` through refactored common code and supports all standard pretty formats and notes display. Marked as EXPERIMENTAL to allow interface refinement, this addresses use cases where `git log`/`git show` fall short, particularly with mixed/duplicate input.

### Windows large object handling fixes

Johannes Schindelin submitted a six-patch series addressing Windows platform issues with objects >4GB during cloning. The changes systematically replace 32-bit `unsigned long` with 64-bit `size_t` in critical paths:

1. Index-pack/unpack-objects size handling
2. Zlib wrapper updates for 64-bit streams
3. ODB/packfile streaming fixes
4. Delta header handling improvements
5. Test infrastructure for synthetic large packs
6. Regression tests for >4GB object cloning

The series includes comprehensive test coverage and acknowledges remaining work in other subsystems (cat-file, object_info). This addresses a real platform limitation for Windows users working with large repositories.

## In brief

**Remote group push compiler warnings** -- Junio C Hamano provides final polish on Usman Akinyemi's remote group push feature, fixing `-Werror=sign-compare` complaints by adjusting variable types in `builtin/push.c` and `remote.h`.

**HTTP authentication documentation** -- Matthew John Cheetham proposes clarifying `http.emptyAuth` config documentation to explicitly list allowed values (auto/true/false) and their behaviors, complementing the technical fixes already reviewed.

**Index-pack I/O optimization finalized** -- Scott Bauersfeld's patch increasing buffer sizes from 4KB to 128KB is confirmed merge-ready after four iterations, with Junio commending both author and reviewer Derrick Stolee.

**Reintegrate script error reporting** -- Erik Cervin-Edin fixes incorrect warning redirection in the Reintegrate script, properly sending "Huh?" messages about malformed merge commits to stderr instead of stdout.

**Unix socket test fix** -- Jeff King reviews Johannes Schindelin's test fix for Unix domain socket path length limitations, agreeing with the approach but suggesting `mktemp -d` for better security over static /tmp paths.

**Line-log refactoring** -- A 3-patch series makes `git log -L` flow through Git's standard diff pipeline, fixing pickaxe/`--diff-filter` behavior and enabling metadata-only formats while maintaining restrictions on stat formats.

## On the radar

**Tarball reproducibility** -- The discussion continues about making `git archive` output verifiable for packaging workflows, with brian m. carlson proposing versioned reproducible formats and Theodore Tso sharing practical experience from e2fsprogs.

**`--reverse=before` interface debate** -- Junio Hamano strongly advocates for `--max-count-oldest` over the current `--reverse=before` naming, creating a decision point for author Mirko Faina about interface philosophy versus technical accuracy.
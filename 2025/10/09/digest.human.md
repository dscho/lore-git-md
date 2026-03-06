Here's the daily digest for October 9, 2025:

## The day in brief

A busy Thursday with 94 emails across 30 threads, dominated by ongoing discussions about Git's refs subsystem naming conventions, signature handling improvements, and documentation refinements. Key highlights include progress on SHA-1/SHA-256 interoperability documentation, finalization of the packfile store API refactoring, and continued debate about timestamp manipulation warnings in rebase operations.

## Notable threads

### Rebase timestamp manipulation warnings refined

The discussion about warning users against `--committer-date-is-author-date` in rebase operations reached consensus on several points:
- The option will remain for `git-am` where it has valid uses
- Documentation will use strong "lie" terminology to describe timestamp manipulation
- Interactive rebase cases will get explicit warnings
- A stderr warning mechanism is being considered but faces UI challenges

Kristoffer Haugsbakk and Phillip Wood led the discussion, with Junio Hamano weighing in on wording choices. The thread revealed deeper concerns about timestamp ordering violations that weren't fully addressed in the initial patch.

### Ref subsystem naming debate continues

Post-merge discussion about Patrick Steinhardt's refs modernization series focused on naming conventions for `struct reference` versus `struct ref`. Jeff King and Junio Hamano argued the new `struct reference` name is too generic given its iteration-specific context, while Patrick and Toon Claes defended it as a foundational structure. The debate highlights ongoing tension between specificity and generality in core data structures, with no resolution yet on whether to rename either structure.

### SHA-1/SHA-256 interoperability documentation

brian m. carlson's multi-part series adding comprehensive documentation for hash algorithm interoperability saw significant progress:
- New `gitformat-loose.adoc` documenting loose object storage
- Pack index v3 format specifications (8-byte offsets, checksum handling)
- Tag signature behavior clarifications (aligning docs with Git 2.29+ reality)
- Test infrastructure for compatibility mode (COMPAT_HASH prerequisite)

The work lays crucial groundwork for Git 3.0's planned SHA-256 support while maintaining backward compatibility.

### Packfile store API finalized

Patrick Steinhardt's packfile store refactoring series concluded with:
- Introduction of `repo_for_each_pack()` macro
- Removal of deprecated `packfile_store_get_packs()`
- Renaming `get_all_packs()` to `get_packs()`
- Conversion of 20+ callers to new patterns

The changes standardize pack iteration across the codebase while eliminating confusing stateful behavior between pack-getting variants.

## In brief

**Fast-import signature handling** -- Christian Couder's series adding symmetric tag signature support between fast-import/export was approved for merging after addressing review feedback.

**Rust CI improvements** -- Patrick Steinhardt's Rust infrastructure series received maintainer approval pending resolution of the column width debate (80 vs 100 chars).

**Stash config documentation** -- D. Ben Knoble added explicit docs about how `stash.index` affects `--autostash` in merge/rebase/pull.

**Const correctness fix** -- Okhuomon Ajayi addressed a const correctness warning in patch-id comparison logic, removing an obsolete NEEDSWORK comment.

**Windows include cleanup** -- Johannes Schindelin reorganized includes in `compat/mingw.c` to use project-relative paths and alphabetical ordering.

**git whatchanged deprecation** -- Users reported continued use of the deprecated command, with migration guidance provided (`git log --no-merges --raw`).

## On the radar

**Git 3.0 planning** -- Ecosystem coordination continues for SHA-256 support, with new suggestions to include web frontends (gitweb/cgit) in compatibility testing.

**Untrusted repository safety** -- Michael Lohmann synthesized discussion threads about marking repositories unsafe, highlighting the tension between path-based and token-based approaches.

**Privacy in Git's object model** -- An Outreachy applicant began exploring privacy challenges related to immutable author metadata, particularly for marginalized developers.
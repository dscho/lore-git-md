# Git Mailing List Digest — 2025/03/17

**The day in brief.** A moderately busy Monday with 68 emails across 25 threads, featuring significant progress on several fronts. Key developments include final resolution of the protocol v2 fetch behavior series, continued work on Windows file handling and assertion safety, and the release of Git for Windows 2.49.0 with notable deprecations. The day saw productive discussions about cruft pack handling and refspec API cleanup, alongside various documentation and test improvements.

## Notable threads

### Protocol v2 fetch behavior finalized

The long-running discussion about HEAD update behavior during fetches reached resolution today. Jeff King and Junio Hamano converged on strict rules where HEAD updates will only occur when using configured fetch refspecs, not during exact-OID fetches or when fetching specific refs by name. This represents a user-visible behavior change but one both agree is justified by the feature's newness and current behavior's lack of clear rationale. The decision finalizes the architectural principle that HEAD updates should be explicitly configured rather than implicit.

Taylor Blau followed up with a 4-patch series refactoring the refspec API to use boolean flags rather than enums, building on the protocol v2 work. The changes make the API more explicit by replacing generic functions with fetch/push-specific variants. While mechanical, these changes improve code clarity and represent the final step in modernizing the refspec handling after extensive protocol improvements.

### Safer assertions approach approved

Elijah Newren's series introducing safer alternatives to assert() cleared its final hurdle today when the licensing concerns around the CI detection mechanism were resolved. The original author placed the clever two-line technique into the public domain, allowing the series to proceed. The discussion then turned to naming of the new macro (currently BUG_IF_NOT()), with Junio suggesting alternatives like ASSERT() that better convey its purpose as a production-safe assertion. The technical approach - converting 9 existing cases across merge, object storage, and parallel checkout code - has been approved, with only naming and documentation polish remaining.

### Git for Windows 2.49.0 released

Johannes Schindelin announced Git for Windows 2.49.0, which includes two significant deprecations: git-svn support will be phased out due to maintenance challenges, and 32-bit installers (except MinGit) are being discontinued immediately (with MinGit's 32-bit support itself ending in April 2029). The release incorporates upstream Git 2.49.0 changes including the stabilized `--name-hash-version=2` option and new `git backfill` command, plus Windows-specific fixes for symlink handling, terminal freezes, and editor syntax highlighting.

### Cruft pack handling improvements

A new series from Taylor Blau proposes replacing the problematic `--max-cruft-size` behavior with a clearer `--combine-cruft-below-size` option. The changes first reorganize tests (moving cruft-related tests to more appropriate files) before modifying the implementation to combine small cruft packs while leaving larger ones untouched. The approach allows combined packs to exceed the threshold size when aggregating many small packs, focusing on reducing repack frequency rather than strict size limits. The series represents careful, test-driven evolution of cruft pack handling following earlier fixes for object freshening.

## In brief

Documentation: Junio confirmed a small but accurate fix about `git restore --patch` pathspec support will be included, removing outdated Perl-era limitations.

Windows file handling: Patrick Steinhardt and Johannes Schindelin continued discussing how to properly distinguish between file locking and permission errors on Windows, revealing fundamental API limitations in error reporting.

Build system: The unreachable code detection series concluded with a generalized solution using a `NOT_CONSTANT` macro after progressing through platform-specific workarounds.

Test infrastructure: A side discussion emerged about `test_commit` vs raw `git commit` in tests, acknowledging the impracticality of converting thousands of existing instances despite some drawbacks.

Meson build: Patrick Steinhardt explained why manpage handling requires explicit configuration of breaking changes files, even when the option is enabled.

Cygwin compatibility: Discussion continued about dropping NO_REGEX support for Cygwin 1.7 (10 years old) and how to handle test t7815's NUL byte behavior.

## On the radar

The `git switch --force` vs `--discard-changes` behavior discrepancy with untracked files sparked discussion about whether to document the difference or align the implementations. Junio traced the divergence to specific commits but suggested the behavior may now be too established to change safely.

A performance report highlighted significant differences between `git add .` and `git status` in large repositories, with brian m. carlson explaining the technical reasons (`git add .` must scan for both modified and new files while `git status` can optimize differently).
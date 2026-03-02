# Git Mailing List Digest — 2025/04/14

**The day in brief.** A moderately busy Monday with 89 emails across 21 threads saw continued refinement of several major patch series, including build system alignment, Meson benchmark integration, and MIDX/cruft pack optimizations. Notable developments include Junio's approval of documentation modernization work, progress on the promisor-remote capability extension, and an engaging start to Git's 20th anniversary community interview project.

## Notable threads

### Build system alignment reaches final refinements

The extensive build system alignment series saw multiple technical discussions today as it nears completion. Ramsay Jones and Patrick Steinhardt worked through final platform-specific issues including Cygwin regex support (debating whether to drop compatibility with Cygwin 1.7), memory calculation fixes for `git gc`, and Meson's path handling limitations for DEFAULT_EDITOR/DEFAULT_PAGER values. The thread demonstrated Git's careful consideration of backward compatibility tradeoffs, with consensus forming around modernizing platform support while maintaining robust build configurations.

### Meson benchmark integration approved

Patrick Steinhardt's series integrating Git's performance benchmarks with the Meson build system received final approval after addressing all review feedback. The v2 implementation maintains compatibility with the traditional `t/perf/run` script while adding native Meson support for single-version benchmarking. Key changes included proper Perl path portability using `#!/usr/bin/env perl`, out-of-tree build support, and comprehensive test wiring. The series represents an important step in Git's multi-year Meson adoption effort, with the team now turning attention to adapting `t/perf/run` for Meson builds as future work.

### MIDX/cruft pack optimization series progresses

Taylor Blau posted v2 of his 8-part series optimizing how MIDX interacts with cruft packs during repacking. The refined implementation introduces a configurable `repack.midxMustContainCruft` option (defaulting to true for compatibility) that can exclude cruft packs from MIDX when they're not needed for reachability closure. The series includes preparatory refactoring of pack-objects option handling and introduces a new `--stdin-packs=follow` mode for maintaining reachability guarantees. Junio provided early feedback on code style and initialization patterns, while the core optimization appears technically sound pending real-world testing at GitHub.

### Promisor-remote capability extension raises protocol concerns

A 4-part series extending promisor-remote capabilities to support additional remote configuration fields drew Junio's scrutiny regarding protocol design. While the implementation (adding `promisor.sendExtraFields` and `promisor.checkExtraFields` config options) appeared technically solid, Junio raised important concerns about the open-ended nature of the field mechanism. He argued for precisely defined fields in the protocol specification rather than an unbounded extension system, citing interoperability and security considerations. This feedback may require significant redesign before the feature can proceed.

### Git's 20th anniversary community interview launches

The Git Rev News team initiated a collaborative community interview project to celebrate Git's 20th anniversary, posing 9 reflective questions about Git's evolution and future. Early responses from contributors like Luca Milanesio and Lucas Seiki Oshiro offered personal perspectives ranging from disaster recovery stories to CLI design critiques. The project provides an engaging opportunity for community participation ahead of the anniversary edition of Git Rev News planned for late April.

## In brief

Junio approved Jean-Noël Avila's documentation modernization series converting `git-reset`, `git-rm`, and `git-mv` to modern AsciiDoc format after all v1 feedback was addressed. The perf test fixes series from Philippe Blain was acknowledged as complete, with only a minor documentation typo fix remaining. The reftable API refactoring thread concluded with Justin Tobler's positive review of Patrick Steinhardt's v2 changes. The `git add` wildcard pathspec bugfix saw test refinement discussions focusing on Windows compatibility. A UTF-8-BOM handling issue on Windows was reported, with analysis suggesting interaction between BOM conversion and CRLF line endings. Lucas Seiki Oshiro contributed a simple cleanup removing an unused `log_reencode` field from `struct rev_info`.

## On the radar

The `--no-hooks` global option discussion continues with proposals to document the existing `/dev/null` workaround as an alternative to new code. The patch series tracking mechanism discussion explored Theodore Ts'o's "Patch Set ID" system as a Change-Id alternative, while the GPG signing thread shifted focus to transport-layer authentication approaches like patatt's header-based method. The path-based delta compression series remains in "What's Cooking" awaiting deeper maintainer review despite real-world adoption in Git for Windows. The object-file cleanup and build tweaks series are progressing toward merge pending final review comments.
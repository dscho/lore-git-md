Here's the daily digest for February 19, 2026:

**The day in brief.** A busy day with 87 emails across 31 threads, dominated by several major technical efforts reaching completion. Key highlights include the finalization of the ref backend selection series (enabling zero-downtime migrations between files and reftable backends) and UTF-8 alias support in config subsections. Multiple CI/build system improvements also progressed toward integration.

**Notable threads**

**Ref backend selection finalized**  
Karthik Nayak's series enabling zero-downtime migrations between ref backends (files<->reftable) has reached v7 with maintainer approval. The implementation now provides three control mechanisms: `extensions.refStorage` config, `GIT_REFERENCE_BACKEND` environment variable, and URI-based specification. The final patches focus on stub management centralization and comprehensive test coverage, particularly for environment variable behavior. This completes a multi-version effort primarily serving GitLab's migration needs, with all technical concerns resolved and positive reviews from Patrick Steinhardt and others.

**UTF-8 alias support ready**  
Jonatan Holmgren's series implementing UTF-8 support in config subsections (e.g., `[alias "förgrena"]`) is now merge-ready after seven iterations. The v7 patch reverts the output format of the new `--aliases-for-completion` option to match Git's established convention of using newlines within NUL-terminated records. The solution handles both traditional and subsection alias formats with comprehensive test coverage (including Swedish, CJK, and spaces). Junio has approved the series for inclusion in 'next', marking the successful conclusion of this first-time contributor's effort.

**CI/build system unification**  
Patrick Steinhardt's series standardizing test infrastructure across GitLab and GitHub CI is now complete, with Johannes Schindelin contributing a Windows-specific test-slicing fix. The changes ensure identical behavior for MSVC+Meson builds on both platforms, including proper failure handling and artifact collection. Key improvements include one-based test slicing, consistent Meson test execution paths, and proper shell interpreter selection. Junio has marked the topic for integration, representing the final piece in making Git's test infrastructure robust across platforms.

**In brief**  
**Ref iteration fixes** -- Patrick Steinhardt's series addressing BUG() conditions in bitmap and bisect ref handling is approved, standardizing on ref hierarchy matching with trailing slashes.

**Config-based hooks progress** -- Junio indicates Adrian Ratiu's prerequisite hooks-conversion work is ready for 'next', while reviewing v2 improvements to callback data management.

**Build dependency tracking** -- D. Ben Knoble's solution for `config-list.h` generation now validated across platforms, with comprehensive path escaping and identical Makefile/Meson behavior.

**Repository discovery hardening** -- Tian Yuchen's security-focused series addressing `.git` symlink handling progresses to v6, with Junio providing detailed feedback on error handling contracts.

**Worktree API refinements** -- Phillip Wood's v2 series removing redundant repository parameters advances, while Junio raises deeper questions about worktree/repository relationships in submodule contexts.

**On the radar**  
**`the_repository` removal** -- Elijah Newren's merge-ort series nears completion, with discussions surfacing about historical constraints in prefetch behavior and macro naming consistency.

**Documentation clarifications** -- Minor but important docs improvements progress for `fetch --jobs=0` behavior and `format.noprefix` boolean handling.
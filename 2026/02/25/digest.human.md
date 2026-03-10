Here's the daily digest for 2026/02/25:

---

**The day in brief.** A busy Wednesday with 121 emails across 28 threads, featuring major progress on reference backend selection, Linux fsmonitor support, and geometric repacking. Key developments include the final approval of Karthik Nayak's ref backend series and Taylor Blau's RFC for incremental MIDX repacking.

---

**Notable threads**

**Reference backend selection ready for 'next'**  
Karthik Nayak's series implementing zero-downtime migrations between ref backends (files<->reftable) has reached its ninth iteration and final approval. The implementation provides three configuration mechanisms (`extensions.refStorage` config, `GIT_REFERENCE_BACKEND` environment variable, and URI syntax) with proper precedence rules. Junio Hamano performed final polish on test script style before merging to 'next'. The work primarily serves GitLab's needs for migrating large repositories.

**Linux fsmonitor support stabilized**  
Paul Tarjan's Linux inotify-based fsmonitor implementation (v6) has addressed all major technical concerns, with only minor memory leaks remaining as non-blocking issues. The series now includes meson build support and handles edge cases like container filesystems through timeouts. Patrick Steinhardt will review the final version before submission, marking the conclusion of a six-iteration effort to bring Linux to parity with Windows/macOS fsmonitor backends.

**Geometric repacking RFC**  
Taylor Blau introduced a 14-patch RFC for incremental MIDX/bitmap-based repacking, building on his prior MIDX compaction work. The proposal avoids periodic all-into-one repacks by maintaining geometric progression of MIDX layers, introducing new `--checksum-only` and `--base` MIDX options. The series represents the third major component of large-repository repack improvements and will likely generate significant discussion from pack subsystem experts.

**Configurable hooks moving forward**  
Junio Hamano signaled readiness to proceed with Adrian Ratiu's config-based hooks series (v3) while treating parallel execution as a separate follow-up effort. The core functionality has maintainer approval after addressing Patrick Steinhardt's reviews, with only email delivery issues between contributors noted as a logistical concern.

**Pre-add hook documentation finalized**  
Chandra Kethi-Reddy's `pre-add` hook implementation has settled on backtick formatting (`pre-add`) for documentation, matching existing hook style. The v3 patch addresses all technical feedback and awaits final maintainer confirmation of the documentation presentation before merging.

---

**In brief**  

**Submodule remote handling fix** -- Karthik Nayak's patch fixing submodule fetching's assumption about remote names ("origin") is queued for integration after addressing final test portability issues.

**UTF-8 alias memory leak** -- Junio Hamano identified and fixed memory leaks in the UTF-8 alias subsections implementation now in 'next', where alias values in string list util fields weren't being freed.

**Worktree API refinements** -- Phillip Wood and Junio discussed architectural questions about worktree-repository relationships that emerged from recent API simplifications, particularly around `is_current_worktree()` behavior in multi-repo contexts.

**Build dependency tracking** -- D. Ben Knoble's series ensuring `config-list.h` stays synchronized with documentation files is now complete, handling the critical edge case where removed files would break Make builds.

**Cover letter formatting** -- Mirko Faina's configurable cover letter format series saw extensive review from Jeff King, who proposed extending format strings to customize patch numbering prefixes while questioning if the complexity was justified.

**GSoC proposal refinement** -- Tian Yuchen submitted a properly formatted GSoC proposal for `the_repository` removal after initial submission issues, building on their experience with test modernization and symlink handling patches.

**On the radar**  

**Partial clone disk reclamation** -- Amisha Chhajed's GSoC proposal for improving disk space recovery in partial clones has shifted focus from spatial (cone/non-cone) to temporal (old/new objects) considerations per Derrick Stolee's feedback.

**Log decoration behavior** -- An active discussion continues about making `git log`'s `%d` format specifier respect `--decorate` flags, with proposals ranging from a new 'never' value to modifying `--decorate=no` behavior, highlighting tensions between consistency and backward compatibility.
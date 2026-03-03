Here's the daily digest for May 9, 2025:

**The day in brief.** A busy Friday with 104 emails across 22 threads, dominated by major architectural changes to Git's object database and systematic cleanup of the contrib/ directory. Key highlights include Patrick Steinhardt's finalized ODB refactoring series and ongoing discussions about stash import/export functionality. Several bugfixes and platform-specific improvements also progressed toward merging.

**Notable threads**

**Object database refactoring complete**  
Patrick Steinhardt's 17-patch series to decouple Git's object database from `the_repository` reached its conclusion with maintainer approval. The changes rename core structures (`raw_object_store` to `object_database`, `object_directory` to `odb_alternate`), establish consistent `odb_`-prefixed APIs, and remove global state dependencies across 139 files. Junio Hamano will merge the series as-is, noting only minor naming preferences but no technical objections. This foundational work enables future pluggable ODB backends, with Patrick planning an RFC for mixed backend support in Git 2.51.

**Stash import/export refinement**  
The stash import/export feature series saw extensive review from Junio Hamano, focusing on edge case hardening. Discussions covered reflog handling robustness, commit validation during import, and error reporting consistency. Brian Carlson clarified the feature's motivation - enabling safe stash transfer across development environments like GitHub Codespaces - while addressing questions about its advantages over manual push/apply workflows. The series appears technically sound but awaits final documentation updates before merging.

**Systematic contrib/ cleanup**  
Patrick Steinhardt's 11-patch series removing obsolete contrib/ components concluded with Elijah Newren raising policy questions about using "5 years without updates" as a removal criterion. The series deleted 27 items including git-resurrect.sh (replaced by reflog), mw-to-git (unmaintained MediaWiki helper), and Emacs integration files (superseded by Magit). Discussion now shifts to formalizing contrib/README policies, with consensus that components should "move up, move out, or disappear" rather than linger unmaintained.

**Packed-refs memory optimization finalized**  
The packed-refs memory optimization series wrapped up with detailed discussion about mmap strategy implementation. Reviewers and author shejialuo aligned on using direct mmap/verify/unmap for fsck operations while preserving existing mmap approaches for other cases. The changes reduce memory pressure when verifying large packed-refs files and are ready for integration after finalizing error handling details.

**In brief**  

**Mailinfo header leak** -- Lidong Yan explores refactoring `decode_?_segment()` functions to use caller-allocated strbufs after fixing a memory leak in header parsing.

**Reftable cleanup** -- Lidong Yan removes the unused REFTABLE_REALLOC_ARRAY macro from reftable/basics.h after confirming it wasn't referenced anywhere.

**Commit-graph leak fix** -- Lidong Yan addresses a memory leak in `fill_oids_from_packs()` when pack index opening fails, adding proper cleanup in the error path.

**BSD CPU detection** -- Brad Smith improves OpenBSD/NetBSD CPU core detection by adding HW_NCPUONLINE support when SMT is disabled.

**Parse-options leak discussion** -- Phillip Wood proposes simplifying unknown option handling by modifying argv in-place rather than allocating new strings.

**Build system safety** -- Junio confirms Meson properly handles special characters in path arrays, addressing earlier concerns about build system robustness.

**On the radar**  

**Git log filtering** -- A feature request highlights display issues when excluding bot-generated commits, particularly graph visualization problems that break history flow. This may spark discussion about improving `--exclude-author` and parent rewriting for filtered views.
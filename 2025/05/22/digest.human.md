# Git Mailing List Digest - 2025/05/22

## The day in brief

A busy day with 100 emails across 25 threads, featuring significant progress on several fronts. Key highlights include the completion of the stash import/export feature series, ongoing refinements to the ODB abstraction work, and multiple performance optimizations nearing completion. The day also saw several bugfixes and the introduction of OAuth2.0 support for imap-send.

## Notable threads

### Stash import/export reaches final form

The long-running effort to enable stash portability between repositories has culminated in v6 of the series, now approved for merging. The implementation introduces `git stash export` and `git stash import` commands that store stashes as commit chains under `refs/stash-export/`. The final version includes rigorous validation of imported commits, comprehensive test coverage, and addressed all major review concerns. Junio Hamano provided detailed feedback on the import validation logic and memory handling, ultimately giving the implementation his approval pending Phillip Wood's final sign-off.

### ODB abstraction naming conventions refined

Patrick Steinhardt's ongoing object database abstraction work saw active discussion about naming conventions for core structures. Justin Tobler raised concerns about the proposed `struct odb_alternate` name, suggesting `object_source` might better reflect its role in both primary and alternate object storage. The exchange highlights the careful consideration being given to long-term API design in this foundational refactoring effort. Meanwhile, the first patch of v3 (renaming `struct raw_object_store` to `struct object_database`) received positive review with only minor observations about temporary naming quirks.

### MIDX performance optimizations refined

Multiple threads converged on performance improvements for multi-pack-index handling. Jeff King and Junio Hamano engaged in detailed discussion about edge case safety in the negative lookup cache implementation, ultimately confirming the approach correctly handles pack lookups across MIDX chains. A separate series from Phillip Wood addressing integer overflow issues in MIDX repack functionality reached v2 with improved documentation of the mtime-based tie-breaking behavior. These changes collectively improve handling of large repositories, particularly on 32-bit systems.

### imap-send gains OAuth2.0 support

A two-part series revitalized the neglected `imap-send` command, fixing a configuration parsing bug and adding modern OAuth2.0 authentication support. The initial implementation using OpenSSL for base64 encoding was revised to use curl's native OAuth support after Brian Carlson raised licensing concerns. The series also addressed memory leaks in the CRAM-MD5 authentication path. While the command appears to have limited usage, the improvements modernize its authentication options and fix long-standing issues.

### Packed-refs memory handling finalized

Jeff King confirmed approval of the final iteration in the packed-refs memory optimization series, which standardizes memory handling across runtime and fsck paths. The changes address filesystem safety for empty packed-refs files and refactor buffer allocation into a reusable helper function. With explicit sign-offs from all key reviewers (King, Steinhardt, and Hamano), this work is now cleared for merging after five iterations of refinement.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**Batched ref updates delayed post-2.50** -- Karthik Nayak confirms the performance optimization will get extra bake time on 'next' due to its sensitive nature, despite all technical issues being resolved.

**MPTCP transport support explored** -- Matthieu Baerts outlined integration pathways for Git's various transports (git://, HTTPS, SSH) with the multipath TCP protocol.

**CVS server Perl warning fixed** -- Ondřej Pohořelský addressed a Perl 5.41.4+ compatibility issue in git-cvsserver, switching to the idiomatic `!~` operator.

**Hard link performance regression** -- Justin Tobler identified metadata changes as the cause of slow `git status` with hard-linked large files, suggesting `core.trustCTime=false` as a workaround.

**Documentation extension updates** -- Jouke Witteveen and Junio Hamano discussed challenges in transitioning .txt references to .adoc while maintaining compatibility with Git's documentation toolchain.

**On the radar**

**Smudge/clean filter empty file handling** -- Joey Hess reported and Jeff King analyzed inconsistent behavior when filters process empty files, with Junio Hamano questioning whether the current behavior might be intentional.

**HEAD.lock contention** -- A new bug report describes intermittent failures during checkouts when background maintenance holds HEAD.lock, raising questions about proper lock contention handling.
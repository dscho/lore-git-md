# Git Mailing List Digest - 2025/04/08

**The day in brief.** A busy day with 97 emails across 26 threads, featuring major architectural changes, build system updates, and ongoing discussions about commit metadata. Key highlights include the completion of Patrick Steinhardt's object-file refactoring series, final approvals for batched reference updates, and continued debate about Change-ID standardization.

## Notable threads

### Object storage subsystem refactoring completed

Patrick Steinhardt's 9-part series to split object-file.c into logical components has been merged after extensive review. This major architectural change systematically reorganizes Git's object storage code, moving functionality to more appropriate locations: directory utilities to dir.c, mmap handling to wrapper.c, index operations to read-cache.c, and core object store management to a new object-store.c. The series eliminates global state and establishes clean boundaries between subsystems, paving the way for future pluggable storage backends. Junio noted some "unpleasant interactions" during merging but confirmed the series is now queued for inclusion.

### Batched reference updates ready for merging

Karthik Nayak's series implementing batched reference updates with partial failure support received final approval after six iterations. The changes introduce a `--batch-updates` flag for `git update-ref` that allows transactions to proceed when individual updates fail, while maintaining atomicity for system errors. The implementation spans all ref backends (files, packed, reftable) and includes comprehensive test coverage. A minor email formatting issue sparked discussion about `git send-email` usability improvements, though this didn't affect the technical approval.

### Merge-recursive removal advances

Elijah Newren's series to fully remove the legacy merge-recursive backend in favor of merge-ort reached its third iteration. The changes systematically convert remaining callers (checkout, merge-recursive builtin, sequencer) while updating tests to expect merge-ort's behavior. The series now focuses on documentation aspects, with v3 improving commit messages to better explain merge-ort's advantages (performance, worktree-less operation, bug fixes) and the planned nature of this multi-year transition. This represents the final step in making merge-ort Git's sole merge backend.

### Change-ID debate evolves

The Change-ID standardization discussion took an interesting turn as Junio Hamano proposed an alternative DAG-based model using explicit predecessor-successor relationships recorded in commit trailers. This contrasts with earlier proposals for persistent Change-IDs, addressing concerns about distributed evolution tracking while maintaining Git's native workflow. The thread continues to balance comprehensive solutions against implementation complexity, with Phillip Wood noting similarities to the earlier "git evolve" proposal and Nico Williams advocating for simpler approaches.

## In brief

Build system updates dominated the day's volume work. Emily Shaffer added 'headers-check' functionality to meson builds, matching Makefile's hdr-check target. Junio raised naming concerns (hdr-check vs headers-check) that will need resolution. CI infrastructure changes addressed GitHub's upcoming Ubuntu 20.04 deprecation, with Brian Carlson and Johannes Schindelen coordinating updates to runner images and sparse package handling.

The reflog-expire maintenance task series from Patrick Steinhardt advanced to v2, refactoring expiration code to remove global state while adding the new task to git-maintenance. Karthik Nayak's performance optimization for bundle creation (replacing O(N^2) duplicate checking with O(1) strset lookups) is ready for merging after addressing style feedback.

Test improvements included Anthony Wang's Perforce label import test updates (now using `show-ref --verify` instead of grep patterns) and Lucas Oshiro's modernization of t5601-clone.sh to use test_path_is_file. The Perl removal series was approved for merging after Johannes Schindelen's sign-off, completing a major test dependency cleanup.

## On the radar

The discussion about preserving extra commit headers during rebase operations remains active, now focusing on validation requirements and semantic concerns. Phillip Wood's RFC patch has drawn attention from multiple maintainers, with brian m. carlson emphasizing real-world corruption patterns and Junio questioning how to handle headers with unknown semantics during rewriting operations.
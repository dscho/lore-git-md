# Git Mailing List Digest - 2025/10/14

## The day in brief

A busy day with 97 emails across 28 threads, dominated by major progress on Patrick Steinhardt's `git-history` series and Li Chen's `rebase --trailer` feature. The `git-history` series received comprehensive review and maintainer integration, while the rebase trailer work reached its fourth iteration. Notable discussions also emerged around patch-id hash algorithms and HTTP authentication behavior.

## Notable threads

### git-history command series progresses

Patrick Steinhardt's 17-patch series introducing the new `git history` command for commit rewriting (with `reword` and `split` subcommands) received thorough review from Karthik Nayak and was rebased onto the latest `seen` branch by Junio Hamano. The series has evolved through multiple versions to focus on non-conflicting operations and now builds on shared `replay.c` infrastructure. Key technical points include:

- New `wt_status_collect_changes_trees()` function generalizes status collection
- Continued `the_repository` removal in replay infrastructure
- Robust commit parsing checks added for external callers
- Comprehensive test coverage for history editing operations

The series appears ready for potential inclusion pending resolution of minor style nits and documentation questions. Junio's rebase encountered only trivial conflicts, suggesting good architectural alignment with existing code.

### Rebase trailer support refactoring

Li Chen's 29-patch series adding `--trailer` support to `git rebase` reached its fourth iteration, with significant refactoring to move trailer processing from an external `interpret-trailers` helper into core Git. The series now features:

- New `amend_strbuf_with_trailers()` for in-memory processing
- State persistence across rebase interruptions
- Comprehensive test coverage in t3440
- Support for config-based trailer mappings
- Integration with `fixup -C` operations

Review feedback focused on test organization and documentation formatting rather than core functionality, suggesting the technical approach is sound. The series demonstrates careful attention to edge cases and maintains backward compatibility throughout.

### Patch-id hash algorithm debate

A discussion emerged around standardizing SHA-1 usage for patch IDs, prompted by an Outreachy participant's patch. Key technical points:

- Current behavior respects repository hash algorithm (SHA-256 in SHA-256 repos)
- Junio argues SHA-1 should be used for stable patch comparison
- brian m. carlson cites SHA-1 deprecation requirements
- Proposed compromise: configurable algorithm with SHA-1 fallback

The thread highlights tensions between backward compatibility and cryptographic standards, with no clear resolution yet. The discussion remains technical but collegial, focusing on finding a solution that serves both historical comparison needs and future algorithm flexibility.

## In brief

**SHA-1/SHA-256 interoperability** -- Patrick Steinhardt approved the 9-patch subset moving to 'next' after Junio's stability confirmation and brian m. carlson's readiness declaration.

**Hook parallelization FD handling** -- Adrian Ratiu abandoned switching from 0 to -1 for invalid FDs due to extensive run-command API impacts.

**Git data model documentation** -- Julia Evans' v3 of gitdatamodel.adoc incorporates feedback from 48 testers, explaining core concepts while avoiding implementation details.

**Ref backend naming** -- Patrick Steinhardt elaborated on maintaining separate `struct ref` (transport) and `struct reference` (core) types post-merge.

**Symlink symref deprecation** -- Proposal to remove `core.preferSymlinkRefs` in Git 3.0, with warnings added for current usage.

**HTTP 403 authentication** -- Initial patch to treat 403 like 401 for credential prompts faced pushback over protocol semantics and workflow impacts.

## On the radar

**Test modernization** -- Junio provided detailed feedback on proper use of test_path helpers in worktree prune tests, awaiting corrected patch.

**Outreachy contributions** -- Multiple applicants are engaging with mentors on starter projects including test modernization and `the_repository` removal.

**Maintenance release** -- Git 2.51.1 is planned with documentation updates including `whatchanged` deprecation guidance.
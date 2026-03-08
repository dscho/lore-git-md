# Git Mailing List Digest - 2025/11/03

**The day in brief.** A busy Monday with 71 emails across 20 threads, featuring a significant regression in tag handling during fetch operations, final refinements to the ODB abstraction work, and the first release candidate for Git 2.52.0. The most critical discussion centers around a `git fetch --tags` regression in bare repositories that surfaced after the reference transaction changes in Git 2.51.0.

## Notable threads

### Fetch tag handling regression in bare repositories

A serious regression in `git fetch --tags` behavior in bare repositories has been identified and analyzed. David Bohman reported that since Git 2.51.0, when fetching tags in a bare repository where some tags would be rejected (such as when force-pushing tags upstream), the entire operation fails and no tags are stored - even those that should have been accepted. This represents a change from pre-2.51 behavior where only the conflicting tags would fail while others would succeed.

Karthik Nayak confirmed the issue is reproducible and appears related to the new reference transaction system's atomic behavior. Chris Torek noted this may be an intentional (if undesirable) consequence of the atomic transaction model. The discussion has evolved into whether the old per-tag behavior or new atomic behavior is correct, with David Bohman arguing the current behavior is both a regression and misleading since the output suggests tags were fetched when they weren't actually stored.

### ODB abstraction work finalized

Patrick Steinhardt's 13-patch series to refactor loose object handling in Git's object database (ODB) abstraction reached its final form with v3. The series encapsulates all loose object state into a dedicated `struct odb_source_loose` and standardizes function names with an `odb_source_loose_*` prefix. After resolving earlier naming debates (settling on `odb_source_loose` over `odb_loose_source`), the series received final approval from reviewers. This work is part of the larger effort to enable pluggable object storage backends in Git.

### Version tag sorting regression

A regression in version tag sorting (`--sort=version:tag`) was identified after merging the `ps/ref-peeled-tags` changes. Junio Hamano reported that version tags like v1.0, v2.0, v10.0 now sort incorrectly (ASCII order rather than numeric). Jeff King pinpointed the issue to incorrect caching behavior in `ref-filter.c` where `data->maybe_object` wasn't being reset between tag evaluations. A simple one-line fix was proposed, though its exact placement may need discussion. This affects a core functionality that worked correctly before the ref-filter optimizations.

### Git v2.52.0-rc0 released

Junio Hamano announced the first release candidate for Git 2.52.0, containing 585 non-merge commits from 76 contributors (30 new). Highlights include the new `git repo` subcommand, `git last-modified` path tracking, continued `the_repository` removal work, Rust integration progress, and numerous performance improvements. The release notes provide comprehensive details on all changes, with the final release expected after the RC period.

## In brief

**Atomic ref updates for git replay** -- Siddharth Asthana and Phillip Wood refined reflog message formats for atomic reference updates in `git replay`, settling on including target branch names for clarity.

**Git data model documentation** -- Julia Evans and Patrick Steinhardt made final polishing touches to the new `gitdatamodel.adoc` documentation, resolving terminology questions around "root tree" versus "top-level directory" and improving list formatting consistency.

**Maintenance optimization checks** -- Karthik Nayak's series adding `git maintenance is-needed` subcommand received final review feedback, with only minor documentation and style tweaks remaining before merge.

**Diff algorithm configurability for blame** -- Phillip Wood reviewed the final patches enabling diff algorithm selection in `git blame`, suggesting hiding the legacy `--minimal` flag to avoid confusion with the new multi-algorithm support.

**Merge-ort rename caching fix** -- Elijah Newren submitted a three-patch series fixing an assertion failure in merge-ort's rename detection cache, encountered in production at GitHub during complex replay operations.

## On the radar

**Rust interoperability** -- The SHA-1/SHA-256 interoperability work continues with technical discussions about integer type choices at the Rust FFI boundary, though no new patches were posted today.

**Outreachy proposals** -- Two Outreachy internship proposals for refactoring Git's global state were submitted, with Christian Couder providing deadline reminders and administrative guidance to applicants.
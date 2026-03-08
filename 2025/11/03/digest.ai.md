# Git Mailing List Digest - 2025/11/03

## The day in brief

A busy day with 71 emails across 20 threads, dominated by a regression in `git fetch --tags` behavior in bare repositories and the final polishing of Patrick Steinhardt's ODB abstraction series. Notable developments include a confirmed bugfix for batched tag fetches, a version sorting regression in the first release candidate for Git 2.52.0, and continued refinement of Git's data model documentation.

## Notable threads

### **`git fetch --tags` regression in bare repositories**

David Bohman provided a reproducible test case showing that `git fetch --tags` in bare repositories fails to store new tags when encountering a rejected force-pushed tag, despite reporting them as "[new tag]". Karthik Nayak confirmed the issue, which appears related to the reference transaction updates in Git 2.51.0. The discussion evolved into whether the new atomic failure behavior (where any tag rejection fails the entire operation) is preferable to the old per-tag behavior. Jeff King identified the root cause in `ref-filter.c` where `data->maybe_object` wasn't being reset between tag evaluations.

### **ODB abstraction series reaches v3**

Patrick Steinhardt's 13-patch series to encapsulate loose object handling in Git's object database abstraction reached its final form with v3. The series standardizes on the `odb_source_loose_*` naming pattern and fully encapsulates loose object state. Karthik Nayak provided final approval for the key patch introducing `struct odb_source_loose`. This completes the architectural work to properly separate loose object operations from the generic ODB interface, paving the way for pluggable backends.

### **Version tag sorting regression**

Junio Hamano reported a regression where `git tag --list --sort=version:tag` produces incorrect ASCII sorting rather than proper version sorting after the ps/ref-peeled-tags changes. The issue was bisected to commit 054f5f45 ("ref-filter: parse objects on demand") and affects both loose and packed ref scenarios. Jeff King proposed a fix addressing incorrect caching behavior in `ref-filter.c`.

### **Git v2.52.0-rc0 released**

Junio announced the first release candidate for Git 2.52.0, containing 585 non-merge commits from 76 contributors. Highlights include new `git repo` and `git last-modified` commands, performance improvements, continued `the_repository` removal work, Rust integration progress, and numerous bug fixes. The comprehensive release notes provide detailed categorization of all included changes.

### **Data model documentation final polish**

Julia Evans' `gitdatamodel.adoc` series received final polishing feedback from Patrick Steinhardt and Junio Hamano. The discussion focused on terminology consistency ("root tree" vs "top-level directory"), punctuation in lists, and clarifying tag object descriptions. With these minor editorial tweaks, the documentation appears ready for merging after extensive review involving multiple domain experts.

## In brief

**Atomic ref updates for git replay** -- Siddharth Asthana proposes specific reflog message formats for both `--onto` and `--advance` modes in response to Phillip Wood's feedback.

**Interactive add navigation wording** -- Ulrich Windl reiterated a stylistic preference for the original help text wording in `git add --interactive`, though acknowledges Junio Hamano's technical rationale for the change.

**Diff algorithm configurability for blame** -- Phillip Wood provided final feedback on the implementation, focusing on `--no-minimal` flag behavior and test script optimizations.

**Merge-ort rename caching fix** -- Elijah Newren submitted a three-patch series fixing an assertion failure in merge-ort's rename detection logic, with preparatory cleanups and an extensive test case.

**Outreachy internship proposals** -- Two applicants submitted proposals for refactoring Git's global state, with Christian Couder providing deadline reminders and administrative guidance.

## On the radar

**Rust interoperability** -- The SHA-1/SHA-256 interoperability work saw technical clarification about integer type choices at the Rust/C FFI boundary, though no new patches were posted today.

**Maintenance optimization checks** -- Karthik Nayak's series to add `git maintenance is-needed` received final polish feedback from Patrick Steinhardt, with only minor documentation tweaks remaining.
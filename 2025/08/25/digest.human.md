# Git Mailing List Digest - 2025/08/25

**The day in brief.** A moderately busy Monday with 54 emails across 22 threads, featuring ongoing technical debates about Rust interop and data structures, several performance optimizations nearing completion, and important clarifications about grafts removal impacts. Key highlights include finalization of the bulk-checkin refactoring series and strengthened warnings about SHA-1/SHA-256 interoperability limitations.

## Notable threads

**Rust interop header inclusion debate continues**  
Ezekiel Newren continues the discussion about Rust/C interop implementation details, specifically addressing header file inclusion style for the `ivec` type in the xdiff series. The email provides statistical analysis showing 361 of 362 existing includes of git-compat-util.h use double quotes rather than angle brackets, seeking Junio Hamano's guidance on whether to follow this convention or adopt a different style. Meanwhile, Newren also explains to Ben Knoble why Rust's native `Vec` can't be used directly for FFI due to unstable struct layouts between Rust versions, defending the custom `ivec` implementation as necessary for version stability.

**Packfile store abstraction progresses**  
Taylor Blau provides detailed review feedback on Patrick Steinhardt's packfile store abstraction series (patch 2/16), focusing on memory management and API design. Key concerns include dangling pointers in `packfile_store_free()`, unwieldy access chains like `current->repo->objects->packfiles->packs`, and redundant close operations. While generally positive about the incremental approach, Blau suggests renaming fields for clarity and documenting memory management requirements more explicitly. The review maintains the thread's technical focus on safely abstracting packfile operations.

**Grafts removal and shallow clone concerns clarified**  
Junio Hamano provides important technical clarification that shallow clone operations (`--depth`, `--shallow-since`) use their own mechanism separate from both grafts and git-replace, meaning grafts removal shouldn't impact shallow clone functionality. This responds to Askar Safin's earlier concern about maintaining `git clone --depth=1` behavior. While this narrows the scope of potential impacts, the thread continues to discuss the separate history.git use case that remains unresolved by this clarification.

**Bulk-checkin refactoring finalized**  
The bulk-checkin refactoring series reaches its final form, completing the removal of global transaction state as part of the `the_repository` elimination project. Junio Hamano indicates readiness to merge the changes, which now consistently use `repo_get_object_directory()` and enforce mandatory transaction requirements. While two globals remain marked for future work, the series represents a significant step toward supporting pluggable ODB backends. The implementation maintains identical behavior while enforcing stricter transaction lifecycle management across multiple builtin operations.

**Line-log optimizations receive final polish**  
Derrick Stolee's line-log optimization series concludes with Junio Hamano accepting the final readability-focused patch after positive performance results. The series speeds up merge commit processing in `git log -L` by avoiding unnecessary tree diffs (1.15-1.38x speedups even with Bloom filters) and includes several cleanups to memory management and control flow. The last patch reorganizes a condition check for better readability, which Hamano accepts despite initial skepticism, considering poor readability a form of "brokenness" worth fixing.

## In brief

**`git whatchanged` deprecation discussion** -- Jeff King and Junio Hamano clarify that empty commit handling differs between `git whatchanged` and its `git log` replacement due to `revs.always_show_header`, but consider this philosophically justified rather than a bug.

**Documentation updates** -- Julia Evans submits a 5-patch series improving `git-checkout` man page clarity based on user feedback, while Kristoffer Haugsbakk fixes a mergetool.adoc typo and brian m. carlson strengthens warnings about `extensions.compatobjectformat` limitations.

**Performance experiments** -- Jeff King shares benchmark results comparing khash and commit-slab implementations for topological sorting, showing commit-slabs outperform khash 7x in dense commit scenarios despite khash advantages in sparse cases.

**Progress meter signal handling** -- Johannes Sixt's bugfix for progress delay functionality concludes with agreement on using `int` rather than `bool` for backport compatibility, finalizing the signal-free timer implementation.

**`git pull` short option** -- A new patch adds `-u` as shorthand for `--set-upstream` in `git pull`, matching `git push`'s existing alias.

## On the radar

**SHA-1/SHA-256 interoperability** -- The documentation warnings have been strengthened, but the fundamental technical limitations (no packed object support making push/fetch non-functional) remain unresolved.

**Rust interop design** -- While implementation details like header inclusion are being refined, the broader policy debate about Rust in Git's core continues without clear resolution.
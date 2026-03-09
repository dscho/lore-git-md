# Git Mailing List Digest - 2026/01/13

**The day in brief.** A busy day with 91 emails across 22 threads, featuring significant progress on several fronts. Key highlights include the final approval of Adrian Ratiu's submodule path encoding series after 10 iterations, Patrick Steinhardt's `git-history` command nearing completion with Elijah Newren's sign-off, and multiple bugfixes addressing regressions in hook behavior. Junio's "What's cooking" report provides a comprehensive snapshot of ongoing development efforts.

## Notable threads

### **Submodule path encoding finalized**

Adrian Ratiu's long-running submodule path encoding series has received final approval after 10 iterations. The implementation provides robust filesystem-safe handling of submodule paths through URL encoding and hashing fallbacks, with comprehensive collision resolution. Production readiness was confirmed by Josh Steadman (Google), validating the solution against real-world use cases. Patrick Steinhardt and Junio Hamano have signed off, marking the conclusion of this multi-month effort to handle submodule paths across all filesystems.

### **git-history command approaches completion**

Patrick Steinhardt's `git-history` command series has addressed all technical feedback in its v11 iteration, with Elijah Newren providing final sign-off on the replay infrastructure changes. The implementation introduces in-memory history editing capabilities (starting with `reword` subcommand) while avoiding working tree impact. Key refinements include standardized conflict reporting, detached HEAD support, and improved test coverage. The series builds on Git's replay machinery and represents significant architectural refactoring to support library integration. With all technical concerns resolved, this appears ready for maintainer evaluation.

### **Hook subsystem regression fixes**

Multiple threads addressed regressions in Git's hook subsystem. A pre-push hook output redirection issue (stdout incorrectly sent to stderr) was fixed by making the behavior configurable, maintaining backward compatibility. Separately, a performance regression causing minute-long delays when pre-push hooks read stdin was identified and is being addressed by Adrian Ratiu with test coverage plan. These fixes highlight ongoing refinement of the hook API while preserving compatibility with existing workflows.

### **In brief**

**MIDX compaction naming** -- Taylor Blau and Patrick Steinhardt finalized naming conventions for MIDX functions, settling on `get_midx_hash_hex()` for consistency with existing interfaces.

**Status push tracking** -- Harald Nordgren's push tracking enhancement for `git status` reached v25 with improved advice display logic and test coverage, now merged to `next`.

**the_repository removal** -- Olamide Caleb Bello (Outreachy intern) progressed on moving config values from globals to repository structs, with Junio providing architectural feedback about config scoping.

**HTTP auth fix** -- Bearer token authentication during large clones was fixed to properly include headers in `probe_rpc()` calls, with thorough test coverage added.

**On the radar**

**Rust integration** -- brian m. carlson's SHA-1/SHA-256 interoperability work continues in the background, mentioned in Junio's "What's cooking" as ongoing.

**Windows symlinks** -- Johannes Schindelin's symlink support improvements are progressing through the pipeline, noted as active work in the integration report.
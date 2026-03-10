Here's the daily digest for February 15, 2026:

**The day in brief.** A moderately busy day with 39 emails across 20 threads, featuring progress on several long-running technical efforts. Key developments include final refinements to the ref backend selection work, resolution of a subtree regression, and ongoing discussions about repository discovery hardening and worktree API improvements.

### Notable threads

**Ref backend selection nears completion**  
Karthik Nayak and Jeff King finalized path handling safety in the reference storage backend selection work, which enables zero-downtime migrations between files and reftable backends. The discussion centered on validating path invariants with either comments or assertions rather than runtime checks. This marks the final polishing phase of a series that has already received maintainer approval from Junio Hamano, with all substantive technical concerns resolved.

**Shallow repository fixes reach v5**  
Samo Pogačnik's series addressing memory leaks and relative-depth fetching in shallow repositories progressed to v5 after addressing Patrick Steinhardt's feedback about code organization. The patches now unify the shallow commit calculation logic while maintaining component reusability. The well-tested changes fix edge cases where deepening operations could leave some shallow commits unreachable behind merged branches.

**Subtree regression resolved with performance win**  
Colin Stagner submitted a final fix for a `git subtree` regression introduced in Git 2.52.0, completely removing a problematic optimization that was causing both functional issues and performance bottlenecks. Testing shows the fixed version runs 3x faster than the original working version while correctly handling squashed commits. The solution represents a significant simplification of the subtree split logic.

**Repository discovery hardening refined**  
Tian Yuchen's security-focused series hardening Git's repository discovery by validating `.git` file types saw significant refinement in its error handling taxonomy. Junio Hamano identified several implementation flaws in how fatal vs non-fatal conditions were classified. The v3 patch now properly distinguishes between missing paths, directories, invalid files, and real errors while maintaining symlink support.

**In brief**  
**HTTP 429 retry tests** -- Jeff King suggested using `test-tool date` instead of GNU date commands in Vaidas Pilkauskas's HTTP rate limiting series for better portability.

**German l10n fix** -- Matthias Schwarzott corrected a typo in the German translation of `color-moved-ws` configuration that was causing a frustrating error loop.

**Ref-filter cleanup** -- Jeff King completed a refactoring series that simplifies the refname component stripping logic in ref-filter.c, addressing both const-correctness warnings and memory handling.

**Worktree API discussion** -- Shreyansh Paliwal clarified the distinction between primary and current worktrees in their ongoing refactor to replace NULL parameters with explicit structs.

**On the radar**  
**Rustification effort** -- While not discussed today, Ezekiel Newren's work to introduce Rust code into Git remains a contentious topic that may resurface, particularly regarding platform support concerns raised by Randall S. Becker.
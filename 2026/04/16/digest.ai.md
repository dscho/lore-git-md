Here's the daily digest for April 16, 2026:

### The day in brief
A moderately busy day with 96 emails across 23 threads, featuring significant discussions on MIDX version compatibility for Git 2.54.0-rc2, test infrastructure improvements with `set -e`, and multiple patch series nearing completion. The MIDX version handling reached consensus while the `git subtree` discussion took an unexpected turn toward potential Rust rewrites.

### Notable threads

**MIDX version compatibility resolved for 2.54.0**  
The thread reached consensus on handling MIDX version 2 compatibility issues, with Jeff King (Peff), Taylor Blau, and Junio Hamano agreeing to temporarily revert the default MIDX version from v1 to v2 while keeping v2 available via explicit configuration (`midx.version=2`). This addresses the critical issue where repositories touched by 2.54.0-rc2 became unusable with Git 2.53.0. The solution includes test adjustments and documents a three-phase plan: short-term compatibility (v1 default), medium-term robustness improvements, and long-term v2 transition once ecosystem support exists.

**Test suite reliability with `set -e`**  
Patrick Steinhardt's 12-patch series to enable `set -e` in Git's test suite (Bash 5+ only) progressed through final reviews, with Jeff King raising concerns about long-term maintainability versus the benefits of catching silent test failures. Junio Hamano ultimately accepted the technical approach while acknowledging the tradeoffs. The series includes comprehensive test adaptations and CI configuration to verify the stricter error checking works as intended.

**`git subtree` maintenance crossroads**  
What began as a bugfix thread for `git subtree split` took an architectural turn when Junio Hamano characterized the component as effectively abandonware due to lack of maintainer attention. He suggested a Rust/libgit2 rewrite might be the sustainable path forward if someone steps up to maintain it, prompting discussion about the script's future. Ian Jackson (the original reporter) emerged as a potential champion for such an effort while Colin Stagner's shell-based fixes remain pending.

**Worktree submodule support**  
A new patch series adds `--recurse-submodules` to `git worktree add`, properly initializing submodules in new worktrees. The implementation sparked discussion about object-sharing strategies, with Junio Hamano and Phillip Wood proposing alternative approaches to the initial hardlink-based design. The thread shows careful consideration of platform compatibility and worktree isolation requirements.

**Partial clone performance optimizations**  
Elijah Newren contributed a series extending batch prefetching to `git cherry` and `git grep` in partial clones, addressing performance penalties from individual blob fetches. The well-documented changes include design specifications and comprehensive tests, demonstrating the kind of performance work that has become increasingly important for large repository workflows.

### In brief

**HTTP Kerberos authentication fix** -- Matthew John Cheetham's series fixes `http.emptyAuth=auto` interaction with SPNEGO authentication, with Junio providing positive review feedback.

**`git backfill` improvements** -- Derrick Stolee reviewed Elijah Newren's series adding safety checks and edge blob handling to the new `git backfill` command, confirming it's ready to proceed.

**Documentation polish** -- Elijah Newren sent a cleanup series fixing typos and grammar across multiple documentation files, including release notes and config documentation.

**Subcommand autocorrection finalized** -- Jiamu Sun's subcommand autocorrection series received final approval from Junio after addressing threshold unification feedback.

**gitk race condition fix** -- Paul Mackerras fixed a long-standing race condition in gitk's diff processing that could crash when switching views during background operations.

### On the radar

**Source tree reorganization RFC** -- Patrick Steinhardt proposed moving libgit.a components into a dedicated "lib/" directory, sparking discussion about code discoverability versus disruption.

**`git checkout-index` tree handling** -- Runxi Yu reported unexpected behavior when checking out tree objects directly, raising questions about intentional design versus bug.
Here's the Git mailing list digest for March 23, 2026:

---

### The day in brief
March 23 saw significant activity across multiple fronts in Git development, with 115 emails spanning 24 threads. The day was marked by several feature series reaching completion, including HTTP 429 retry handling, parallel hooks, and commit-list formatting improvements. Architectural discussions around `the_repository` removal and promisor remote security also featured prominently.

### Notable threads

**HTTP 429 rate limit handling finalized**  
Vaidas Pilkauskas's HTTP 429 retry series received maintainer approval and will be merged into 'next'. The implementation adds configurable retry behavior for rate-limited HTTP requests through three new configs (`http.maxRetries`, `http.retryAfter`, `http.maxRetryTime`) with comprehensive test coverage. The v6 iteration addressed all prior feedback around parameter passing and curl version handling, concluding a significant enhancement to Git's HTTP transport capabilities.

**Parallel hooks series concludes**

Adrian Ratiu's parallel hook execution series is now queued for merging after addressing a minor patch numbering inconsistency. The implementation provides comprehensive controls through `hook.jobs` configuration, per-hook `hook.<name>.parallel` flags, CLI `-j/--jobs` override, and per-event master switches. The series has progressed through multiple review rounds with all substantive feedback incorporated.

**Worktree semantics in `the_repository` removal**

Phillip Wood's series removing `the_repository` from worktree code sparked detailed discussion about "current worktree" definitions. The thread revealed undocumented behavior from 2016 and examined edge cases in path handling. While the technical approach remains sound, the conversation highlighted subtle historical assumptions that need documentation as part of this architectural refactoring.

**Promisor remote security hardening**

Christian Couder's URL-based promisor remote whitelisting series received security-focused review from Junio Hamano. Key concerns included tightening glob pattern rules to prevent `*` from matching across `/` boundaries and explicitly documenting the trust model. The discussion will likely lead to follow-up patches implementing safer URL matching defaults while maintaining backward compatibility.

**`git replay` gains `--ref` option**

Toon Claes proposed adding a `--ref` option to `git replay` for precise control over ref updates. The series began with documentation standardization before introducing the new capability. Reviewers identified edge cases around multi-branch handling that will need addressing in future iterations, but the core functionality appears well-motivated.

### In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**Documentation synopsis conversion** -- Jean-Noël Avila continues converting man pages to consistent AsciiDoc synopsis format with minor wording fixes.

**Partial clone disk space analysis** -- GSoC student Yuvraj Singh Chauhan presented a structured evaluation of eviction approaches, favoring explicit `git evict` over automatic solutions.

**MacOS build workaround** -- Johannes Schindelin explained the root cause of Homebrew clang/REG_ENHANCED issues, suggesting the problem may resurface despite current CI fixes.

**`strview` proposal** -- Mateo Patino suggested a read-only `strview` alternative to `STRBUF_INIT_CONST` after previous attempts violated strbuf's writability guarantees.

### On the radar

**ODB abstraction series** -- Patrick Steinhardt's object database refactoring is complete but flagged future work around hash algorithm handling patterns that affect Rust integration.

**`git backfill` argument validation** -- Siddharth Shrimali withdrew his validation patch after Derrick Stolee's upcoming revision argument support will handle errors through standard revision parsing.

**Promisor file consolidation** -- Lorenzo Pegorari's GSoC work on preserving debug info during repacks may need redesign to maintain temporal information across geometric repacks.

---

The day's traffic showed Git's development process in full swing - from major feature completions to nuanced architectural discussions, all while maintaining rigorous attention to security, backward compatibility, and cross-platform support. The mix of deep technical debate and careful documentation polish reflects the project's maturation as it approaches its third decade.
# Git Mailing List Digest - 2025/07/04

**The day in brief.** A moderately active day with 50 emails across 15 threads, featuring significant progress on several fronts. Key developments include the completion of the bloom filter optimization series, a naming debate around the new `for-each-ref` pagination feature, and continued discussion about remote naming ambiguity. Patrick Steinhardt's reftable-as-default series also saw review approval.

## Notable threads

**Bloom filter optimization finalized**  
Lidong Yan's performance optimization series for bloom filters in revision traversal reached its final form with v4 patches addressing all prior feedback. The changes focus on API naming consistency and test helper reorganization while maintaining the core functionality for handling multiple pathspecs. Junio Hamano and Patrick Steinhardt's concerns about function naming conventions were addressed by adopting the `bloom_key_*` prefix pattern. The series introduces a new `bloom_keyvec` structure to manage multiple pathspec keys efficiently while preserving the single-pathspec fast path. Performance testing confirms the optimization only helps exact-case matches, leaving case-sensitivity handling as future work.

**Reftable as default ref backend moves forward**  
Patrick Steinhardt's series to establish reftable as Git 3.0's default ref storage backend received review approval from Karthik Nayak in its v3 iteration. The implementation now includes a dual rollout strategy - announcing the breaking change for Git 3.0 while enabling immediate opt-in via `feature.experimental`. Documentation updates comprehensively list reftable's advantages including case-sensitivity resolution, Unicode handling, and space efficiency via prefix compression. The series maintains focus on the versioned default transition path while preserving all existing compatibility mechanisms.

**Remote naming ambiguity debate continues**  
The discussion about forward slashes in remote names causing refname ambiguity evolved toward technical consensus. Junio Hamano acknowledged Patrick Steinhardt's suggestion to detect naming conflicts (similar to how Git prevents ambiguous refnames) as superior to simply banning slashes. Lidong Yan demonstrated concrete problems where `git branch --set-upstream-to` fails with slash-containing remote names due to refname resolution ambiguity. The thread is now focused on implementing conflict detection that maintains backward compatibility while preventing the problematic cases.

**In brief**  

**`the_repository` removal in prune** -- Ayush Chandekar completes the conversion of `builtin/prune.c` to use repository instances instead of `the_repository`, including proper initialization sequence handling and a new test case.

**String-list modernization approved** -- Patrick Steinhardt gives final approval to shejialuo's series modernizing string-list tests and implementation, marking it ready for merging.

**Diff-filter regression fixed** -- Jeff King's one-line fix resolves a regression where lowercase exclusion filters weren't working with default output formats in `git log --diff-filter`.

**For-each-ref pagination naming debate** -- Active discussion continues about naming the new pagination option (`--skip-until` vs `--start-with` etc.), focusing on how well each communicates the "seek to position" behavior.

**SHA-256 transition clarification** -- brian m. carlson explains the actual hash conversion process for the SHA-256 transition, correcting a misunderstanding about simply re-hashing SHA-1 strings.

**FreeBSD build fix** -- Ramsay Jones addresses a build failure on FreeBSD by reordering system detection checks to prioritize native sysctl() over sysinfo compatibility.

**On the radar**  

**`git init` template behavior** -- Growing consensus to document the long-standing (since 2008) behavior where empty `--template=` skips template copying, with additional context about the newer `--no-template` flag.
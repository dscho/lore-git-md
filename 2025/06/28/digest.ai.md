# Git Mailing List Digest - 2025/06/28

**The day in brief.** A moderately active Saturday with 26 emails across 7 threads, featuring progress on several technical fronts. The most notable developments include a completed series for interactive diff context configuration, a bugfix for `git apply --intent-to-add`, and continued refinement of the pathspec bloom filter optimization. The `the_repository` removal effort also saw review activity after weeks of silence.

## Notable threads

### **Interactive patch commands gain diff context configuration**  
Phillip Wood's v3 series implementing diff context configuration for interactive patch commands (`add -p`, `checkout -p`, etc.) appears ready for merging. The series now includes command-line overrides (`-U` and `--inter-hunk-context`) that respect the same precedence rules as other diff commands, along with comprehensive test coverage and documentation. After addressing feedback about test organization and edge case handling, the implementation looks solid. This completes a long-standing gap where interactive commands didn't properly respect `diff.context` and `diff.interHunkContext` settings.

### **`git apply --intent-to-add` finally working as intended**  
A new series from Johannes Altmanninger fixes the long-broken `--intent-to-add` (`-N`) option in `git apply`, which has been effectively non-functional since its introduction in Git 2.19. The patches address core issues with index handling, ensure intent-to-add markers are only written for new files, add comprehensive test coverage, and clarify documentation. The changes are well-scoped and address a known usability pain point that's persisted for years. The series appears technically sound and likely to progress quickly.

### **Pathspec bloom filters optimized for multiple queries**  
Lidong Yan's v3 series enabling bloom filter acceleration for multiple pathspec queries addresses Junio's review feedback with improved test coverage for wildcard handling and magic bit checking. The implementation introduces `struct bloom_keyvec` to manage multiple pathspec keys while maintaining the single-pathspec fast path. Tests now properly handle both expanded and literal wildcards using `eval`, resolving earlier shell expansion issues. This optimization should significantly improve performance for commands like `log` with combined pathspec queries.

### **`the_repository` removal sees review activity**  
After three weeks without feedback, Ayush Chandekar's `the_repository` removal patches for the precious objects configuration received constructive style suggestions from Jialuo She. The changes involve moving `repository_format_precious_objects` into `struct repository` and updating related commands (`gc`, `prune`, `repack`). While the technical review was light, the exchange demonstrates continued progress on this long-running effort, with Chandekar promptly addressing documentation and attribution improvements.

## In brief

**Git-daemon signal handling redesign** -- Carlo Marcelo Arenas Belón continues refining his pipe-based child tracking approach, addressing Phillip Wood's concerns about FD_CLOEXEC handling and signal handler placement in the ongoing discussion about solving AIX recursion issues.

**Comment character selection in rebase** -- Ayush Chandekar and Phillip Wood discuss edge cases in the interaction between `core.commentChar="auto"` and `core.commentstring`, revealing deeper architectural questions about state management during conflict resolution.

**`git refs list` command proposal** -- The author acknowledges Junio's concerns about feature divergence between the proposed `git refs list` and existing `git for-each-ref`, conceding the current approach might lead to code duplication rather than simplification.

## On the radar

**Reftable compaction** -- While not active today, Patrick Steinhardt's recent reftable work remains important as the new ref backend gains adoption. Watch for follow-ups to his compaction algorithm improvements.

**Rustification effort** -- Ezekiel Newren's ongoing work to introduce Rust code into Git remains contentious, particularly around platform support concerns raised by Randall S. Becker. The discussion may resurface as more Rust patches are proposed.
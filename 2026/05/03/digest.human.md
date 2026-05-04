Here's the daily digest for 2026/05/03 (Sunday) - a moderately busy day with 33 emails across 17 threads, featuring several patch series nearing completion, platform compatibility fixes, and a maintainer's "What's cooking" report.

### The day in brief

A Sunday with steady technical activity across multiple fronts. Junio Hamano's "What's cooking" report provides a snapshot of in-flight work, while several patch series reach maturity - notably the `git url-parse` plumbing command and remote group push support. Platform-specific issues (Windows large objects, RHEL 6 OpenSSL) and philosophical debates about workflow design also feature prominently.

### Notable threads

### `git url-parse` reaches final polish

Matheus Afonso Martins Moreira's series introducing a `git url-parse` plumbing command to expose Git's internal URL parsing logic has reached its third iteration with all technical aspects resolved. Today's discussion focused on final wording tweaks to the cover letter after Junio noted the original justification overstated the shell-out reduction benefits. The implementation handles all Git URL formats (including SCP-style and IPv6) with comprehensive cross-platform testing, particularly for Windows path handling. Torsten Bögershausen raised a question about UNC path support, which the author confirmed works correctly through the existing pipeline. This well-tested series appears ready for merging.

### Remote group push support finalized

Usman Akinyemi's fifth iteration of remote group push support (`git push <group>`) addresses the final type safety concerns flagged by Junio, consistently using `size_t` for array indices in the push CAS (compare-and-swap) option handling. The series now shares group resolution logic between fetch and push commands via refactored helpers in remote.c, with comprehensive tests verifying failure isolation and config handling. The implementation uses child processes for each remote push to ensure failures don't affect other remotes, explicitly rejecting `--atomic` as incompatible with group operations. After five iterations incorporating maintainer feedback, this feature appears merge-ready.

### Windows large object handling progresses

Johannes Schindelin continues advancing his series to fix Windows' 4GB object size limitations, responding to reviews about the incremental patch strategy. He explains the deliberate approach of splitting changes into small, reviewable pieces that follow prior size_t conversion patterns (like f9ba6acaa934), contrasting with the more monolithic GitHub PR approach some contributors found discouraging. The discussion highlights the challenges of upstreaming Windows-specific fixes via Git's mailing-list workflow, where large changes often need decomposition. Additional work remains (notably PR #3533 for pack-objects), but the current patches systematically address index-pack, zlib, and ODB streaming issues.

### Philosophical impasse on tracking branch auto-fetch

Harald Nordgren's `--track=fetch` proposal faces continued rejection from Junio on philosophical grounds despite technical refinements in v5. The maintainer reiterates that considering stale remote-tracking branches "wrong" conflicts with Git's distributed workflow model, where staleness is inherent rather than erroneous. Junio provides an alternative commit message framing the feature purely as a convenience option, but shows no movement on the core objection. The thread appears at an impasse unless the author can reframe the justification in terms compatible with Git's explicit verification principles. Meanwhile, Nordgren pivots to a separate `--autostash` proposal for `git checkout`.

### `fetch.pruneBranches` design debate

Junio responds to Harald Nordgren's `fetch.pruneBranches` proposal with conceptual support but architectural concerns about baking the logic directly into fetch. Instead, he suggests decomposing the functionality into fundamental primitives: a `git branch --forked` to identify branches based on remote refs and `git branch --prune-merged` to clean them up, which fetch could then compose. This reflects Git's Unix philosophy of small, composable tools rather than monolithic features. The response maintains the original safety distinctions (safe/force modes) while surfacing workflow nuances like multi-phase development where branches might be kept despite upstream merging.

### In brief

**Git-gui bare repo handling debate** -- Johannes Sixt advocates preserving backward compatibility for non-bare `.git` directory handling in git-gui, countering Mark Levedahl's more architectural refactoring approach. The discussion awaits maintainer input on whether to prioritize correctness or compatibility.

**Submodule fetchJobs configuration fix** -- Saagar Jha's final v3 patch fixes how `submodule.fetchJobs` is read from `.gitmodules`, correcting a pointer indirection issue after incorporating all review feedback about commit message clarity.

**Windows build configuration update** -- Johannes Schindelin removes the unmaintained nedmalloc allocator from Windows builds to address GCC 16 compatibility, noting Git for Windows already uses mimalloc.

**RHEL 6 build failure** -- A regression report shows Git 2.54 failing to build on RHEL 6 due to OpenSSL 3.x symbol requirements conflicting with the system's OpenSSL 1.0.x libraries, particularly affecting `git-imap-send`.

**GSoC 2026 participants announced** -- Kaartic Sivaraam shares the selected contributors for Google Summer of Code, including projects on `git repo` improvements, remote object info, partial clone space recovery, and global state reduction.

### On the radar

**Rustification effort** -- Ezekiel Newren's work to introduce Rust code into Git continues in the background, with Junio's "What's cooking" noting xdiff cleanup as preparatory steps. The digest shows steady progress across multiple fronts, with several features reaching maturity while platform-specific issues and workflow design debates continue to shape Git's evolution. Junio's "What's cooking" report provides a valuable snapshot of the broader development landscape as the day's discussions unfold.
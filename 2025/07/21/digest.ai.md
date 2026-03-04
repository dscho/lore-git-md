# Git Mailing List Digest - 2025/07/21

**The day in brief.** A moderately busy Monday with 36 emails across 16 threads, dominated by final refinements to the promisor-remote protocol extension series and several performance investigations. Key developments include Christian Couder's v6 of the promisor-remote validation patches nearing merge readiness, Phillip Wood's analysis of xdiff optimization alternatives, and multiple threads reaching maintainer approval.

## Notable threads

**Promisor-remote protocol extensions reach v6** -- Christian Couder's series to enhance promisor-remote capabilities is in its final polishing phase after six iterations. The changes allow servers to advertise additional remote fields (`partialCloneFilter` and `token`) via `promisor.sendFields` while clients can validate these using `promisor.checkFields`. Today's discussion focused on documentation refinements and terminology (whether to use "field" or "configuration variable" language), with Junio Hamano raising concerns about consistency with Git's existing configuration model. The technical implementation appears settled, leaving only naming and documentation questions before potential merge.

**xdiff optimization alternatives analyzed** -- Phillip Wood presented benchmark results showing that a 15% performance improvement in xdiff (previously attributed to Rust optimizations) can be achieved through targeted C changes: an 8% gain from moving whitespace flag checks out of a hot path and 6% from switching to XXH3 hashing. This analysis informs the ongoing debate about whether introducing Rust dependencies is justified when similar gains may be achievable through C optimizations. Junio noted the findings suggest the whitespace optimization would be valuable regardless of language considerations.

**IMAP folder archiving for send-email** -- Aditya Garg's v2 patch adding IMAP folder support to `git send-email` received minor documentation fixes from Eric Sunshine, followed by naming critique from Junio Hamano. The feature addresses email providers that don't save SMTP-sent messages by piping them to `git imap-send`. Junio suggested more specific terms like "sent" or "fcc" (file carbon copy) would better reflect the feature's purpose than the current generic `sendemail.imapfolder` naming. The implementation appears technically sound with only naming semantics remaining as the final pre-merge question.

**MemorySanitizer challenges with crypto** -- Jeff King and Karthik Nayak discussed strategies for handling MemorySanitizer false positives from cryptographic operations, particularly OpenSSL's `csprng_bytes()`. The thread explored three approaches: manual memory unpoisoning, Clang ignorelists, and full library instrumentation. While the immediate pagination feature triggering these warnings is ready, the discussion revealed systemic challenges with cryptographic library interactions that may need broader consideration in Git's sanitizer strategy.

**Test reorganization for help output** -- A 2-patch series implemented Junio Hamano's suggested strategy for testing command help output, automating outside-repository testing via `git --list-cmds` while relocating repository-dependent tests to command-specific files. The changes demonstrate a maintainable approach to comprehensive help testing while keeping test files logically organized. The patches received positive feedback and appear ready for merging.

## In brief

**Diff context configuration approved** -- Junio plans to merge Phillip Wood's series unifying diff context configuration across interactive commands after v4 addressed all review feedback, including test modernization and config handling simplifications.

**Early-output removal queued** -- Junio confirmed Jeff King's assessment that the undocumented `--early-output` feature has no users, approving its removal from the revision machinery.

**Reftable compaction fix** -- A v3 patch fixes a memory leak in reflog expiration found by ASAN, adding proper cleanup for `reflog_expire_entry_option` allocations.

**Bisect interface improvements discussed** -- Junio and Christian Couder explored potential safety checks for `git bisect` after diagnosing a user's confusion from misusing `skip HEAD~1`.

**Git for Windows permissions issue** -- James Gao reported serious file permission errors preventing basic operations in Git 2.50.1 on Windows 11, with no known workarounds yet.

**Rebase documentation confusion** -- Rodney Bates identified contradictory descriptions in `git rebase` help text, with Nico Williams clarifying the precise meaning of "reset" in this context.

## On the radar

**Rust/xdiff optimization debate** -- Phillip Wood's performance analysis has shifted the discussion toward whether C optimizations can achieve similar gains to the proposed Rust implementation, particularly on non-SIMD platforms.
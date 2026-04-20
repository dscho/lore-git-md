Here's the daily digest for April 19, 2026:

**The day in brief.** A moderately active Sunday with 21 emails across 13 threads, featuring significant technical discussions around subtree Rust rewrites, revision walking controls, and partial clone optimizations. Key developments include Ian Jackson's Rust prototype for `git subtree`, final refinements to Derrick Stolee's negotiation controls, and a new `--missing=print-only` mode for rev-list.

**Notable threads**

**Subtree Rust rewrite proposal emerges**  
Ian Jackson escalated the `contrib/subtree` discussion by presenting a working Rust/libgit2 prototype as an alternative to Colin Stagner's shell script fixes. The prototype demonstrates performance improvements but proposes changing commit metadata handling (using dummy committers with original attribution in messages) - a behavior change requiring evaluation. The thread now explicitly weighs three options: Colin's shell fix, Ian's Rust rewrite as a standalone tool, or integration into git.git. Key questions remain about packaging (crates.io vs git.git integration) and Git's `contrib/libgit-rs` maturity.

**Negotiation controls near completion**  
Derrick Stolee responded to Junio Hamano's review comments on the negotiation control series, confirming two final changes: updating the `--negotiate-only` error message to use `%s` placeholders for translation safety, and switching to `OPT_ALIAS()` for the `--negotiation-tip` backward compatibility. The exchange demonstrates the series converging toward production readiness through interface polish, with no substantive changes to the underlying functionality.

**Reverse walking behavior refined**  
The `--reverse=before` revision walk patch saw extensive technical discussion between Mirko Faina and Ben Knoble. Key points included backward compatibility with multiple `--reverse` flags, code clarity improvements via `#define` constants, and detailed analysis of `max_count` handling differences between `before` and `after` modes. Mirko provided thorough explanations of the complex control flow interactions in this two-phase operation (walk phase and output phase), showing progress toward consensus on implementation details.

**In brief**  

**`git stash` push assumption finalized** -- Deveshi Dwivedi's patch to make `git stash` assume "push" when unambiguous flags are present is now merge-ready, having addressed all review comments. The final version unconditionally assumes push for any leading option flag.

**Test suite shell safety** -- SZEDER Gábor's fix for tilde expansion edge cases in object filter tests was confirmed by Elijah Newren, addressing both theoretical username collision risks and real Bash segfaults in versions 3.2-5.0.

**Pseudo-merge bitmap review** -- Elijah Newren identified documentation mismatches in Taylor Blau's series, noting `sampleRate=0` is incorrectly documented as valid when the code now rejects it.

**UTF-8 truncation test question** -- Lorenzo Pegorari questioned whether the control character test case actually demonstrates the out-of-bounds read issue it was designed to catch, suggesting potential test coverage gaps.

**Partial clone rev-list mode** -- A new `--missing=print-only` mode was proposed for `git rev-list` to support GitLab's Gitaly service, emitting bare OIDs without prefixes. Derrick Stolee's review noted several implementation polish opportunities.

**Localization updates** -- Jiang Xin coordinated translation updates for Git 2.54.0, with significant zh_TW changes and continued Bulgarian updates from Alexander Shopov.

**On the radar**  

**Source tree reorganization** -- Patrick Steinhardt's proposal to move libgit.a components into a dedicated "lib/" directory received thoughtful review from Phillip Wood, who questioned whether the change actually improves code discoverability as claimed.
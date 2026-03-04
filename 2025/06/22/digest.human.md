# Git Mailing List Digest — 2025/06/22

## The day in brief

A quiet Sunday with 10 emails across 6 threads, mostly focused on wrapping up existing discussions. The most notable developments include a v2 series for symbolic port handling in git-credential-netrc and procedural decisions about patch organization in the batched reference updates thread. A new bug report about cloning warnings also surfaced.

## Notable threads

**Symbolic port handling in credential-netrc** — Maxim Cournoyer sends a v2 series implementing symbolic port resolution in git-credential-netrc, despite the credential protocol documentation specifying numeric ports only. The three-patch series starts with a portability fix (shebang update), adds warnings for invalid ports, and finally implements `getservbyname` resolution for service names like "smtps". The implementation appears technically sound with good test coverage, though it directly contradicts Brian Carlson's interpretation of the protocol documentation. This keeps alive the design debate about whether symbolic ports should be supported at all.

**Batched reference updates wrap-up** — Junio Hamano and Karthik Nayak finalize the organization of patches from the batched reference updates bugfix series. They agree to keep the D/F conflict handling patch separate from the core bugfix`, characterizing it as a documented stepping stone toward future improvements in transaction processing. This concludes the thread with all immediate issues resolved while preserving context for future work.

**Cloning warnings bug report** — A new thread reports numerous "badFilemode" warnings and one "missingTaggerEntry" warning when cloning git.git from GitHub. The clone succeeds despite the warnings, which reference over 60 object hashes. The report raises questions about repository maintenance practices but provides no analysis of root causes or suggested fixes.

## In brief

**git-gui localization format** — Alexander Shopov proposes removing location comments from future translation submissions to improve diff stability, building on earlier discussions with Johannes Sixt about .po file formats.

**coccicheck build refinement** — Junio Hamano suggests using `grep -q` instead of `wc -c` to check for non-empty Coccinelle patches, offering both stylistic preference and potential performance benefits.

**clang-format workflow** — Junio endorses Jeff King's rebase-based approach for applying clang-format changes, seeing it as a useful middle ground between full automation and manual review.
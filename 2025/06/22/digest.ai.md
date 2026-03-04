# Git Mailing List Digest — 2025/06/22

## The day in brief

A quiet Sunday with 10 emails across 6 threads, mostly focused on wrapping up existing discussions. The most notable developments include a v2 series for symbolic port handling in git-credential-netrc and procedural conclusions about batched reference updates. Build system refinements and localization discussions continue at a measured pace.

## Notable threads

### **Symbolic port support in credential-netrc reaches v2**

Maxim Cournoyer's v2 series for git-credential-netrc implements helper-side resolution of symbolic port names (like "smtps") despite credential protocol documentation specifying numeric ports only. The three-patch series:

1. Improves Perl script portability with `/usr/bin/env` shebang
2. Adds warnings for invalid port specifications
3. Implements full `getservbyname` resolution for symbolic ports

The implementation touches multiple components — the netrc helper, Git.pm, and git-send-email — with thorough test coverage. While technically sound, this approach directly contradicts Brian Carlson's interpretation of the credential protocol as numeric-only, leaving the design debate unresolved. The series demonstrates a working alternative to caller-side resolution but may face protocol-level pushback.

### **Batched reference updates patch disposition finalized**

Junio Hamano and Karthik Nayak concluded discussion about organizing fixes for batched reference updates. They agreed to keep two patches separate: one master-bound bugfix for `git-update-ref`, and another addressing D/F conflicts in `receive-pack` that Junio characterizes as a "band-aid" solution. This preserves context for future improvements to transaction processing while documenting current limitations through a NEEDSWORK comment. The thread wraps up with all immediate issues resolved but clear markers for follow-on work.

## In brief

**Git-gui localization format discussion** — Alexander Shopov proposes removing location comments from translation files to improve diff stability, building on Johannes Sixt's earlier questions about .po file sorting. The suggestion moves the infrastructure discussion forward while keeping the Bulgarian translation work complete.

**`make coccicheck` refinement** — Junio Hamano suggests using `grep -q` instead of `wc -c` to check for non-empty Coccinelle patches, offering both performance benefits and alignment with his longstanding preference for certain shell syntax patterns.

**Clang-format workflow improvements** — Junio responds to Jeff King's rebase-based workflow proposal, endorsing the `-x -p` approach as a middle ground that maintains developer oversight while catching obvious style violations. The discussion shifts from configuration details to integration mechanics.

## On the radar

**Repository clone warnings** — A new bug report details numerous "badFilemode" and one "missingTaggerEntry" warnings during git.git clones from GitHub. The volume suggests systematic object database issues that may warrant investigation, though the clone operation completes successfully despite the warnings.
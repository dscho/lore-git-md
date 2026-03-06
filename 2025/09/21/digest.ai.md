# Git Mailing List Digest - 2025/09/21

## The day in brief

A moderately active day with 28 emails across 6 threads, dominated by technical discussions around Rust integration challenges and xdiff refactoring cleanups. The most notable developments include definitive confirmation of Rust adoption barriers on NonStop platforms and the resolution of several post-merge questions about the xdiff preparatory series. A small but important curl type-safety bugfix series also received maintainer approval.

## Notable threads

### Rust adoption constraints on NonStop platforms

The thread exploring Rust compatibility with NonStop systems reached its conclusion with Randall Becker providing definitive technical constraints. Cross-compilation from Windows or Linux is impossible due to proprietary licensing requirements, security audit needs, and production control limitations. Becker confirmed organizational interest in Rust support exists but the timeline exceeds Git's Rustification schedule. Ezekiel Newren summarized the systemic chicken-and-egg problem where platform constraints prevent Rust adoption while lack of demand prevents vendor investment in Rust tooling. This discussion effectively closes the technical feasibility question, leaving open the policy decision about how Git should proceed with Rust given these confirmed platform limitations.

### xdiff refactoring post-merge review

Several follow-up discussions emerged about the now-merged xdiff preparatory series (patches 1-10). Phillip Wood raised thoughtful questions about implementation consistency across multiple patches, particularly regarding accessor function removal versus addition, local variable elimination, and struct embedding patterns. Ezekiel Newren provided detailed responses clarifying the design rationale, agreeing to improve some commit messages and implementation approaches. The most significant resolution came on the NO/YES/MAYBE macro naming issue - the macros will be moved to xprepare.c and the rchg field type changed to bool to address namespace pollution concerns. These post-merge refinements demonstrate the careful review process even after patches land in 'next'.

### curl type-safety bugfix series

Johannes Schindelin submitted a 3-patch series addressing curl_easy_setopt() type mismatches that caused ARM64 CI failures in Git for Windows. The series introduces a safe casting helper (cast_size_t_to_curl_off_t) and applies it to both HTTP and IMAP subsystems, switching to the more appropriate CURLOPT_INFILESIZE_LARGE API where needed. Junio Hamano reviewed and approved all three patches, requesting only minor documentation clarifications. The changes improve type safety across curl interactions while fixing the immediate build issue, continuing Git's tradition of improving surrounding code when fixing bugs.

## In brief

**git-rebase-clean discussion** -- Antonio Mennillo deferred substantive response to Elijah Newren's technical critique of the semantic conflict resolution tool due to travel, promising a full reply by Wednesday.

**Empty submission** -- An apparently accidental email containing only an iPhone signature line was sent to the list with no content or patches.
# Git Mailing List Digest — 2025/04/26

**The day in brief.** A light Saturday with 7 emails across 4 threads, mostly focused on polishing existing work. The standout is a fix for replace ref decoration in nested paths, while the `git send-email` Outlook integration sees final documentation tweaks and a proposal for runtime Message-ID detection. One new SMTP validation patch rounds out the day.

## Notable threads

### Replace ref decoration fix for nested paths

A bugfix from an unnamed contributor addresses inconsistent handling of replace refs in nested paths (like `refs/replace/2012-migration/abc123`). While Git correctly registered such refs, the decoration logic in `log-tree.c` failed to properly display them due to oversimplistic path parsing. The fix makes decoration use the same "last path component" approach as registration, adding test coverage for both nested replace functionality and log decoration. This low-risk change benefits users who organize replace refs hierarchically, though such usage appears uncommon.

### Outlook SMTP integration nears completion

Aditya Garg's enterprise configurability patch for Outlook Message-ID handling in `git send-email` received final polish from Eric Sunshine, fixing typos and simplifying help text. The core functionality (queued in v6) adds a `--[no-]smtp-outlook-id-tweak` option for cases where Outlook SMTP servers use custom domains. Separately, Yao Zi proposed runtime detection of Message-ID rewriting behavior via cover letter probing, potentially handling non-Outlook providers like Tencent mail. This builds on Greg Kroah-Hartman's earlier enterprise configuration concerns, suggesting a more automated solution than static hostname checks.

## In brief

**SMTP FQDN validation** -- A 3-line patch strengthens `git send-email`'s hostname validation to enforce RFC1035 rules, fixing cases where malformed domains (like "MacBook..local") passed the old check but failed SMTP handshakes.

**CI dependency caching** -- Jialuo responded to Junio Hamano's suggestion about caching external dependencies in CI, committing to investigate platform-specific caching approaches following recent download resiliency improvements.
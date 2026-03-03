# Git Mailing List Digest — 2025/04/26

**The day in brief.** A quiet Saturday with 7 emails across 4 threads, mostly focused on polishing existing work. The notable developments are final refinements to `git send-email`'s Outlook Message-ID handling and a bugfix for replace ref decoration in nested paths.

## Notable threads

**Outlook Message-ID handling reaches final polish** — Aditya Garg's series adding enterprise configurability to `git send-email`'s Outlook Message-ID handling (via `--[no-]smtp-outlook-id-tweak`) received its final documentation polish from Eric Sunshine, catching typos and improving help text clarity. Meanwhile, Yao Zi proposed runtime detection of Message-ID rewriting behavior as a more general solution than hardcoded Outlook domains, suggesting using the cover letter as a probe. This thread has evolved from initial hostname-based detection to a comprehensive solution addressing both standard and enterprise SMTP configurations.

**Replace ref decoration fix for nested paths** — A new patch fixes an inconsistency in how Git handles commit decoration for replace refs organized in nested paths (e.g., `refs/replace/2012-migration/abc123`). While registration worked correctly, the decoration logic was only stripping the base `refs/replace/` prefix, causing warnings and missing decorations. The fix makes both paths use the same "last part of ref" approach and adds test coverage for nested path cases. Though hierarchical replace refs may be uncommon, this aligns behavior consistently.

## In brief

**CI dependency caching investigation** — Jialuo responded to Junio Hamano's suggestion about caching external dependencies in CI pipelines, committing to explore implementation approaches for platform-specific caching to avoid redundant downloads.

**Stricter FQDN validation in send-email** — A small but impactful patch strengthens `git send-email`'s validation of Fully Qualified Domain Names to enforce RFC1035 rules, fixing cases where malformed hostnames (like "MacBook..") previously passed validation but caused SMTP failures.
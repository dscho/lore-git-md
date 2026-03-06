# Git Mailing List Digest — 2025/10/04

## The day in brief

A quiet Saturday with 7 emails across 5 threads, featuring ongoing discussions about Git's Rust integration, documentation fixes, and window management improvements in gitk. The most notable technical contribution is a bugfix for HEAD ref validation in the files backend.

## Notable threads

**Debating Rust vs C tradeoffs** — Eric Wong responds to Ezekiel Newren's comparison of C and Rust features, offering counterpoints about C's capabilities through extensions like `__cleanup__` and `-Wconversion`. While acknowledging Rust's safety benefits, Wong expresses concerns about its async model and build system complexity. This philosophical discussion continues even after technical consensus was reached on Git's Rust integration, highlighting differing views on language evolution approaches.

**gitk window geometry persistence** — Johannes Sixt and Michael Rappazzo collaborate on the final piece of restoring full window management capabilities to gitk, specifically addressing the Tags/Heads window. Sixt proposes a simplified 12-line version of Rappazzo's patch that stores geometry as `geometry(showrefs)` while fixing a subtle scoping issue. This concludes a long-standing effort to modernize gitk's window management now that historical Cygwin Tcl/Tk limitations are no longer relevant.

**HEAD ref validation fix** — A second version of a patch moves validation logic from `fsck_head_link()` to the refs subsystem, preventing creation of invalid HEAD refs during operations like `git reset`. The change adds runtime enforcement in `files-backend.c` and includes a test case in t7102-reset.sh. This focused 10-line change closes a security boundary issue by leveraging existing validation infrastructure.

## In brief

**AsciiDoc rendering fix** — Jean-Noël Avila and Jeff King resolve an edge case in `refStorage` configuration documentation where a warning paragraph's indentation behaved differently between Asciidoc and Asciidoctor processors.

**Test modernization contribution** — Outreachy applicant Vedansh Singh proposes converting `test -f` calls to `test_path_is_file` in t1410-reflog.sh as part of Git's ongoing test helper standardization effort.
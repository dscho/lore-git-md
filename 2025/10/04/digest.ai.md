# Git Mailing List Digest — 2025/10/04

## The day in brief

A quiet Saturday with 7 emails across 5 threads, primarily focused on bugfixes and documentation improvements. The most notable developments include Johannes Sixt's refinement of gitk's window geometry persistence and a security-conscious fix for HEAD ref validation in the files backend. The philosophical debate about Rust vs C continues in the background but appears unlikely to change Git's technical direction.

## Notable threads

**gitk window geometry persistence finalized** — Johannes Sixt builds on Michael Rappazzo's work to complete gitk's Tags/Heads window geometry persistence, reducing the implementation from 39 lines to 12 while fixing a subtle scoping issue. The solution stores window position and size in gitk's config file, marking the culmination of efforts to modernize gitk's window management now that Cygwin Tcl/Tk 8.4.1 limitations are no longer relevant. The thread shows consensus on the approach with only implementation details being refined in this final iteration.

**Files-backend HEAD ref validation** — A security-focused patch moves validation of HEAD ref names from fsck checks to runtime enforcement in the refs subsystem. The change prevents creation of invalid HEAD refs during operations like `git reset` that could place ref files outside the proper directory structure. The v2 revision adds a test case in t7102-reset.sh to verify the new behavior, demonstrating rejection of malformed HEAD references (e.g., `refs/../foo`). The 10-line core change leverages existing validation infrastructure and appears likely to merge without controversy.

## In brief

**C vs Rust philosophical debate** — Eric Wong responds to Ezekiel Newren's comparison of C and Rust features, arguing C can approximate some Rust safety features through extensions while questioning Rust's async/await model and build system complexity. The discussion continues after technical consensus was reached on Rust infrastructure patches.

**AsciiDoc rendering fix** — Jean-Noël Avila and Jeff King resolve an edge case in `refStorage` configuration documentation where a warning paragraph's indentation behaved differently between Asciidoc and Asciidoctor processors, using explicit block markers to maintain consistent rendering.

**Test modernization contribution** — Outreachy applicant Vedansh Singh proposes converting `test -f` calls to `test_path_is_file` in the reflog test script, marking a small but appropriate first contribution aligned with Git's test modernization efforts.
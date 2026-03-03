# Git Mailing List Digest - 2025/05/25

**The day in brief.** A moderately busy day with 48 emails across 14 threads, featuring significant progress on several fronts. Key developments include final iterations of the pack-bitmap memory leak fixes, MIDX API improvements, and a controversial proposal for Microsoft Graph API support in `send-email`. Documentation standardization continues with a 9-patch series converting merge-related manpages.

## Notable threads

**Pack-bitmap memory leaks fixed** -- Taylor Blau and Lidong Yan's v4 series addressing memory leaks in bitmap loading paths has reached its final form, now with comprehensive test coverage. The solution simplifies error handling by centralizing cleanup in `free_bitmap_index()` and introduces a new test helper for structured corruption testing. Junio C Hamano's earlier review feedback has been fully incorporated, making this likely the last iteration before merging.

**MIDX API improvements** -- Taylor Blau continues refining the multi-pack-index subsystem with a series that makes `prepare_midx_pack()` return a `packed_git` pointer directly. This change encapsulates MIDX internals better and reduces error-prone pack ID translation in callers. The series builds on prior performance work while improving API safety - a pattern we've seen consistently in Taylor's MIDX contributions.

**Microsoft Graph API for send-email** -- Aditya Garg's RFC patch proposing Graph API support sparked heated discussion when Brian m. carlson objected on principle, arguing Git should only support standard email protocols. The technical merits (faster sends and Message-ID preservation) are uncontested, but the philosophical debate about vendor-specific integrations may stall progress. This thread is worth watching as it touches core project values around protocol neutrality.

**IMAP-send modernization complete** -- After multiple rounds addressing configuration parsing, memory leaks, and authentication methods, the `imap-send` series has reached v5 with new conveniences like command-line folder specification. Eric Sunshine's request for a `range-diff` is the only remaining item - a strong signal this long-running effort is nearly ready for merging.

**send-email threading fixes** -- Kristoffer Haugsbakk and Aditya Garg worked through edge cases in `send-email`'s message numbering, particularly when editing the first patch in a series. The discussion revealed subtle reproduction requirements but confirmed the core issue exists. The proposed fix (decrementing counters during edits) appears sound though the thread shows how even simple-sounding changes can have complex implications.

## In brief

**Parallel fetch divide-by-zero** -- Junio C Hamano proposes systemic fixes for `max_children` validation, suggesting hardening `online_cpus()` across all platforms rather than just patching fetch.c.

**Notes editor whitespace tests** -- Kristoffer Haugsbakk suggests more comprehensive testing for notes template whitespace cleanup by comparing full output against `git stripspace` results.

**R language diff support** -- Rodrigo Carvalho adds userdiff patterns for R, with Eric Sunshine noting the tests should perhaps cover `=` function assignment alongside the implemented `<-` syntax.

**Maintenance lock conflict** -- A bug report identifies a regression where detached maintenance creates HEAD.lock conflicts during concurrent operations like checkout.

**Documentation standardization** -- A 9-patch series converts merge-related documentation (checkout, switch, merge, mergetool) to the new synopsis style, completing another chunk of this ongoing effort.

## On the radar

**Rustification debate** -- While not active today, Ezekiel Newren's Rust integration work remains a contentious topic, particularly regarding NonStop platform support concerns raised by Randall S. Becker.
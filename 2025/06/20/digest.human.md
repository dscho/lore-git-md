# Git Mailing List Digest - 2025/06/20

## The day in brief

A busy Friday with 79 emails across 15 threads, dominated by major technical discussions around the SHA-256 transition and IMAP improvements. The standout developments include Junio's approval of the long-running `imap-send` feature series and brian m. carlson's SHA-256 default transition patches nearing completion after extensive review.

## Notable threads

### SHA-256 default transition moves forward

brian m. carlson's 10-part series to make SHA-256 the default hash algorithm when built with `WITH_BREAKING_CHANGES` received thorough review from Junio Hamano. The patches introduce new constants (`GIT_HASH_DEFAULT` and `GIT_HASH_ORIGINAL`) to distinguish between legacy SHA-1 requirements and the new default, while maintaining backward compatibility. Key changes include repository initialization updates, test suite adjustments, and the final conditional definition that makes SHA-256 the built-in default when the breaking changes flag is set. 

After resolving naming questions (settling on `GIT_HASH_LEGACY_SHA1` for the constant marking mandatory SHA-1 usage) and verifying comprehensive test coverage for backward compatibility, the series appears ready for merging. This represents a major step toward Git 3.0's planned SHA-256 transition.

### IMAP improvements reach finish line

Aditya Garg's 19-iteration series overhauling `git imap-send` received Junio's approval with all major review feedback addressed. The changes combine critical bugfixes (configuration parsing regressions and memory leaks) with significant new features (OAuth2.0 authentication support and enhanced folder management). The final version improves string handling safety by using `xstrfmt()` and `strbuf` for credential encoding, following Phillip Wood's review suggestions.

This long-running effort makes `imap-send` production-ready while adding modern authentication methods and better UX. Junio's "Looking good. Will replace." indicates the patches meet Git's quality standards after extensive review.

### `repo-info` command design discussions

The new `git repo-info` command series from GSoC student Lucas Seiki Oshiro saw active review from both Karthik Nayak and Junio Hamano. Key discussion points included the command's output model (whether to default to showing everything or require explicit field selection) and documentation gaps. Junio suggested architectural improvements to make the code more maintainable as new fields are added, while Karthik emphasized the need for clearer design rationale in commit messages.

The thread revealed ongoing questions about the `--allow-empty` flag's justification, with Junio proposing an inverted `--all` approach instead. Documentation remains the primary outstanding item for this series as it continues through the review process.

### Style enforcement debate continues

The discussion about Git's `.clang-format` configuration and style enforcement saw continued debate between Junio Hamano and Christian Couder. Junio argued for consistent mechanical improvements to new code, while Christian warned against overly aggressive style checking that could annoy contributors with low-value suggestions. The discussion revealed philosophical differences about whether to prioritize mechanical consistency or human judgment in code formatting, with no clear resolution yet.

## In brief

**Reftable compaction fix** -- Karthik Nayak submits final fixes for batched reference updates, addressing segfaults in the files backend and directory/file conflict handling in receive-pack.

**IMAP error message improvements** -- Joerg Thalheim's v3 series enhances `git imap-send` error messages with actionable advice, now using Git's standard `error()` and `advise()` APIs.

**Symbolic port validation** -- Maxim Cournoyer's patch to support symbolic ports in git-credential-netrc gains RFC6335-based validation rules after review feedback.

**Submodule remote lookup regression** -- Junio reports a segfault in the submodule remote lookup series when `branch->merge_name` was removed, requiring a fix before merging.

**On the radar**

**Rustification effort** -- While not active today, Ezekiel Newren's work to introduce Rust code into Git remains a contentious topic, particularly regarding platform support concerns raised by Randall S. Becker.

**Documentation conversion** -- Jean-Noël Avila's ongoing effort to convert man pages to synopsis-style AsciiDoc continues in the background, with periodic updates expected.
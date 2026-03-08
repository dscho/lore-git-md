# Git Mailing List Digest - 2025/11/24

**The day in brief.** A moderately busy Monday with 43 emails across 13 threads, dominated by a major hook subsystem refactoring series reaching its final form. Notable developments include the completion of the hook API modernization, ongoing discussions about `git clone` config parsing behavior, and a "What's cooking" report from Junio Hamano highlighting several graduated topics.

## Notable threads

### Hook subsystem refactoring completes

Adrian Ratiu and Emily Shaffer's 10-part series converting Git's hook subsystem to a structured API reached its final form today with v3 addressing critical memory leaks identified by Junio Hamano. The series standardizes hook execution across Git, introducing callback-based stdin handling and output capture capabilities while maintaining backward compatibility. Key improvements in this iteration include:

- Fixed memory management in receive-pack hook conversions
- Simplified receive hook stdin callback logic
- Renamed run-command output API to remove hook-specific terminology
- Added all REF_STATUS_REJECT_* to pre-push hook handling
- Improved documentation for hook.h and run-command.h interfaces

The series has received positive reviews from multiple maintainers and appears ready for merging after addressing Junio's concerns about pipe closure timing. This represents a significant infrastructure improvement that enables future features like config-based hooks and parallel execution.

### `git clone` config parsing regression debate

A reported regression in `git clone` config parsing sparked a detailed discussion about historical behavior and documentation expectations. The issue stems from v2.52.0 removing whitespace-trimming around config keys in `-c` arguments, breaking commands like `git clone '-c KEY=VALUE'`. 

Jeff King and Junio Hamano provided historical context tracing this behavior to 2010, with both suggesting the removed behavior may have been unintentional all along. Junio noted the documentation specifies `-c` should receive `<key>=<value>` as a separate argument, implying the quoted form was never officially supported. The discussion is trending toward accepting the change as a correction rather than a regression, though the final decision remains open.

### ASan hardening series refinement

Jeff King's ASan hardening series saw continued discussion around cache-tree parsing safety (Patch 4/9). Junio Hamano proposed a more thorough validation approach and floated the idea of a cache-tree version 2 format to avoid text parsing entirely. The exchange revealed tension between:

- Code simplicity vs thorough validation
- Performance considerations (buffer copying overhead)
- Compatibility with ASan's strict_string_checks feature

While the immediate parsing safety questions are being resolved, Junio's suggestion of a binary format points to potential future architectural changes in how Git handles cache-tree data.

## In brief

**Documentation standardization** -- Jean-Noël Avila completed the standardization of `git fetch`, `git pull`, and `git push` man pages to consistent AsciiDoc synopsis style across 10 files (800+ lines). The v2 series fixed a minor typo and is now ready for merging.

**Reference backend URI design** -- Karthik Nayak refined the GIT_REF_URI implementation based on review feedback, agreeing to clarify documentation and improve input validation while maintaining the temporary override semantics.

**`git config` error message fixes** -- Two separate patches corrected misleading help text in `git config unset` (René Scharfe) and fixed an incorrect suggestion in multi-value config error messages (Paul Wintz).

**`git instaweb` path handling** -- A minimal fix addresses `@` character handling in paths but leaves broader security concerns about Perl script generation unaddressed.

## On the radar

**`git whatchanged` deprecation** -- Kristoffer Haugsbakk defended the forced opt-in approach as intentional, noting the command has been documented as deprecated since 2013. The thread may be winding down unless new arguments emerge.

**Secret management workflows** -- The discussion about clean/smudge filters for secret restoration continues, with Junio Hamano reinforcing architectural constraints while alternative approaches are being explored.
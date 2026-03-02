# Git Mailing List Digest — 2025/03/17

**The day in brief.** A moderately busy Monday with 68 emails across 25 threads saw significant progress on several fronts. Key developments include final resolution of the protocol v2 fetch behavior series, approval of safer assertion macros, and the introduction of new cruft pack handling options. The Git for Windows 2.49.0 release also brought notable deprecation announcements.

## Notable threads

### Protocol v2 fetch behavior finalized

The long-running discussion about HEAD update behavior during fetches reached resolution today with Jeff King and Junio Hamano agreeing on strict rules for when HEAD should be updated. The new behavior will only update HEAD when using configured fetch refspecs, excluding both exact-OID fetches and the intermediate case of fetching specific refs by name without configuration. This represents a user-visible change from current Git but was deemed justified given the feature's newness and the current behavior's lack of clear rationale.

Taylor Blau followed this with a 4-patch series refactoring the refspec API to use boolean flags rather than enums, completing the modernization effort started by the protocol v2 work. The changes make the API more explicit by replacing generic initialization functions with fetch/push-specific variants, though Junio noted the accidental inclusion of `git-diff-pairs` in the submission.

### Safer assertions approved

Elijah Newren's series introducing production-safe assertion macros received final approval after resolving licensing concerns about the CI detection mechanism. The technique for identifying problematic assertions (which transforms `assert(expr)` into `(void)(not_supposed_to_survive || (expr))`) was confirmed to be public domain by its original author. While naming of the new `BUG_IF_NOT()` macro remains under discussion (with Junio preferring alternatives like `ASSERT()`), the technical approach is now cleared for merging.

The series systematically replaces 9 identified `assert()` calls with the new macro across merge, object storage, and parallel checkout code. Taylor Blau suggested documenting the new assertion restrictions in CodingGuidelines but acknowledged this might open a larger discussion about assertion usage patterns.

### New cruft pack handling options

Taylor Blau proposed a new `--combine-cruft-below-size` option for `git repack` to replace the problematic `--max-cruft-size` behavior. The 5-patch series first reorganizes tests before implementing the change, which makes `--max-cruft-size` act as a cruft-specific override for `--max-pack-size` while introducing the new flag to control when small cruft packs should be combined. The change allows combined packs to exceed the threshold size when aggregating small packs, avoiding the previous behavior of repeatedly repacking the same objects near the size limit.

## In brief

Documentation updates: Junio confirmed Johannes Schindelin's review of a `git restore --patch` pathspec documentation fix, removing outdated Perl-era limitations. Zheng Yuting's SMTP error handling refactor received feedback on commit message formatting. Oswald Buddenhagen's `core.commentString=auto` documentation patch prompted discussion about intentionally vague wording.

Build system: The unreachable code detection series concluded with a generalized `NOT_CONSTANT` macro solution. Patrick Steinhardt agreed with Eli Schwartz's patch making Perl mandatory for documentation builds while noting it's the last remaining Perl build dependency.

Windows issues: A thread about file handling fixes revealed fundamental Windows limitations in distinguishing between locked files and permission errors, with no clear solution yet. The Git for Windows 2.49.0 release announced deprecation of git-svn support and 32-bit installers (except MinGit until 2029).

Test suite: Ramsay Jones proposed removing Cygwin 1.7 compatibility code including its NO_REGEX setting, while Junio questioned standardizing NUL byte matching behavior in regex tests.

## On the radar

The `git switch --force` vs `--discard-changes` behavior discrepancy with untracked files sparked discussion about whether to document or align the differences. Junio traced the divergence to implementation details in 2019 but noted the behavior may now be too established to change.

Performance reports highlighted a significant gap between `git add .` and `git status` speeds in large repositories, with brian m. carlson explaining the inherent optimization limitations when scanning for both modified and untracked files.
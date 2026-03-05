# Git Mailing List Digest — 2025/08/17

**The day in brief.** A moderately active Sunday with 21 emails across 15 threads, featuring several notable developments: Junio signs off on the long-running `git repo` command series, localization updates land for Git 2.51.0, and a new proposal emerges for JSON output in `git log` that quickly sparks technical debate about character encoding challenges.

---

## Notable threads

### `git repo` command series approved

Junio C Hamano gave final approval to Lucas Seiki Oshiro's `git repo` command series (v11) after confirming the documentation formatting follows Asciidoctor best practices. This marks the successful completion of a Google Summer of Code project to introduce a new command hierarchy for repository management. The series has undergone extensive review over multiple iterations, with today's discussion focusing narrowly on documentation syntax (`:::` vs `;;` for description lists). With all architectural questions settled, the changes will now move to the 'next' branch for integration.

### Localization updates for 2.51.0

Jiang Xin coordinated a substantial localization update pull request containing translations for 10 languages, which Junio promptly merged. The update includes over 6,000 lines of changes across Bulgarian, Catalan, French, Indonesian, Swedish, Turkish, Ukrainian, Vietnamese, and both Simplified and Traditional Chinese. Catalan saw the most significant refresh (3,000+ lines) while Vietnamese and Swedish each received over 1,000 updates. These changes, submitted after 2.51-rc2, will appear in the final 2.51.0 release.

### JSON output proposal for `git log`

Ron Ziroby Romero proposed adding `--pretty=json` output to `git log`, sparking immediate technical discussion about implementation challenges. The initial suggestion to reuse Git's existing `json-writer.h` infrastructure was quickly followed by deeper concerns from D. Ben Knoble and brian m. carlson about handling non-UTF-8 data in commit messages and paths. Junio later weighed in with design questions about scope—whether to include diff outputs and how to handle GPG signatures—suggesting the feature may need tighter initial boundaries. The thread represents a classic Git design challenge: balancing machine-readability with the system's flexible handling of metadata.

### Documentation rendering issues surface

SZEDER Gábor reported Asciidoctor rendering issues in the recently merged `pretty-formats.adoc` conversion, including unwanted `+` characters and Unicode arrow substitutions. While the problems don't block the changes (already approved), they highlight ongoing toolchain compatibility challenges in the documentation standardization effort. The issues specifically affect the `%(decorate)` section's formatting when built with Asciidoctor 2.0.16, contrasting with the expected output from asciidoc.py.

### Windows support policy clarified

In the Rust dependency series thread, Junio acknowledged Git for Windows' documented support for Windows 8.1+ while emphasizing the importance of conscious version support decisions. The exchange with Matthias Aßhauer clarified that core Git follows Microsoft's EOL dates while Git for Windows maintains extended support, a divergence that needs explicit documentation in technical choices like CARGO_BUILD_TARGET selection. The discussion revealed underlying tensions between security best practices and backward compatibility requirements.

---

## In brief

**Submodule gitdir encoding CI fixes** — Adrian Ratiu reports test failures in the GitHub CI pipeline for his completed 9-patch series on submodule path encoding, with v2 forthcoming to address Windows and macOS-specific issues.

**Stash workflow improvements debated** — Phillip Wood's `--unstaged` option for `git stash pop` drew skepticism from Junio, who questioned whether the behavior should be default rather than opt-in, highlighting differing philosophies about stash's purpose.

**Index file checksum inspection** — Jeff King provided shell commands to manually verify `.git/index` checksums after confirming Git lacks built-in tooling for this low-level debugging need.

**Describe commit safety review** — René Scharfe analyzed Jeff King's `describe_commit()` interface redesign, confirming its general safety while identifying one subtle edge case around callback sequencing in abnormal repository states.

**CMake test discovery fix** — Junio acknowledged Johannes Schindelin's CMake fix for reftable test migration, a straightforward build system parity adjustment missed during the 2.51.0 release cycle.

**Gitweb configuration question** — A user sought help configuring Gitweb to access repositories in user directories (`~/public_git`), though the request lacked technical details needed for specific troubleshooting.

**On the radar.** The `git-add` documentation discussion continues between Jean-Noël Avila and Junio, now focused on how to introduce `git commit` relationships without overwhelming beginners—a debate that may influence broader documentation style guidelines.
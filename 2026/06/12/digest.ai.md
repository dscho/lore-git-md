Here's the Git mailing list digest for June 12, 2026:

### The day in brief
A busy Friday with 107 emails across 29 threads, featuring significant progress on MIDX optimizations, ref backend modernization, and documentation improvements. Key highlights include Taylor Blau's incremental MIDX repacking series reaching completion and Patrick Steinhardt's refactoring to eliminate global state in setup.c being approved for merging.

### Notable threads

**MIDX repacking reaches maturity**  
Taylor Blau's incremental MIDX repacking series (16 patches over 4 versions) has concluded with post-merge test infrastructure improvements. The work enables flexible MIDX/bitmap-based repacking with both append-only and geometric compaction strategies. SZEDER Gábor identified a missing test helper that led to proper test infrastructure generalization rather than a quick fix. This represents a major advancement in Git's large repository maintenance capabilities.

**Global state removal in setup.c**  
Patrick Steinhardt's 7-patch series to eliminate global variables from setup.c has received final approval after thorough review from Justin Tobler and Toon Claes. The changes migrate `protect_hfs`, `protect_ntfs`, and related configurations to repository-specific storage as part of the ongoing `the_repository` removal effort. The series demonstrates Git's careful approach to architectural changes, with all edge cases around initialization sequencing carefully validated.

**Documentation trailer conventions**  
Kristoffer Haugsbakk's 6-patch series to document Git's commit trailer conventions in SubmittingPatches has progressed through review rounds. Junio Hamano provided substantive feedback on the `Based-on-patch-by` trailer documentation, emphasizing the importance of DCO sign-off requirements. Patrick Steinhardt suggested including concrete examples of non-identity trailers to make the guidelines more actionable for newcomers.

**Reachability optimization proposal**  
Kristofer Karlsson proposed an optimization for `paint_down_to_common` that could dramatically speed up merge-base calculations (300x-1000x improvements in some cases). After substantive discussion with Derrick Stolee, they refined the termination conditions to handle complex DAG topologies correctly. The "release branch" scenario was identified as a key use case where the optimization would provide significant benefits.

**AI-assisted code review policy**  
Following Christian Couder's proposal to formalize AI-assisted code review, Junio Hamano established the project's position: while AI tools are encouraged as developer aids, they won't be mandated as submission requirements. The policy draws parallels with GitHub CI's optional-but-valuable status, maintaining Git's tradition of valuing human judgment over process automation.

### In brief

**Reftable backend modernization** -- Patrick Steinhardt's 9-part series modernizing ref backend infrastructure has been fully reviewed, with the final patch transitioning to absolute paths receiving approval from Karthik Nayak.

**`git log --graph` indentation** -- Pablo Sabater's v4 series improves visualization of commits with excluded parents using cascading indentation, now awaiting final documentation polish.

**`git ls-files` optimization** -- Tamir Duberstein's patch to optimize `--deleted` and `--modified` with pathspecs was approved after demonstrating 65.8s to 5.0s improvements in test cases.

**French translation update** -- Jean-Noël Avila submitted routine updates to the French `.po` file, keeping it synchronized with current source strings.

**Test infrastructure gap** -- Miklos Vajna's `git log --follow` improvements for non-linear history are ready for merging pending only a test numbering conflict resolution.

### On the radar

**Rust compatibility concerns** -- Randall S. Becker continues to highlight NonStop build failures due to default-enabled Rust components in Git 2.55.0-rc0, with Junio noting the `NO_RUST` workaround while maintaining Git 3.0's planned Rust requirement.

**Notes handling bug** -- A new test demonstrates incorrect note persistence during rebase when commits disappear due to content being already present, suggesting deeper issues in notes.c and rebase.c interaction.
Here's the daily digest for June 27, 2025:

---

### The day in brief
A moderately busy day with 45 emails across 15 threads, featuring significant progress on performance optimizations, several Windows-specific bug reports, and Junio's "What's cooking" status update. The bloom filter pathspec optimization series nears completion while new issues surface around Windows filesystem interactions.

---

### Notable threads

**Windows regression investigations deepen**  
Two separate Windows-related threads saw progress today. Jörg Hohwiller reported that Git 2.46.0 works where newer versions fail with lock file and ref update errors, narrowing the regression window. Meanwhile, Erwan Leroy identified a "Permission denied" issue with DFS-mounted network drives in Git 2.36+, with Brian m. carlson pointing to a related Git for Windows issue (#3825). Both threads highlight ongoing filesystem interaction challenges on Windows platforms.

**Bloom filter optimizations near completion**  
Lidong Yan's series optimizing bloom filters for multiple pathspecs received thorough review, with Junio Hamano identifying a final correctness issue around magic bit checking. The patches, which address performance gaps when using commands like `git log -- file1 file2`, are in final polishing with test refinements being the primary remaining work. Benchmark results show the optimized version now matches single-pathspec performance.

**Git daemon signal handling refinements**  
The long-running discussion around git-daemon's signal handling saw parallel approaches emerge. Carlo Marcelo Arenas Belón's EINTR handling patch reached v3 with stylistic refinements around variable declarations, while Phillip Wood reviewed a pipe-based child tracking alternative. Both solutions aim to address race conditions in child process reaping, with the thread now focused on implementation details rather than architectural questions.

**In brief**  
- `git refs list` proposal -- A GSoC participant introduces a new subcommand mirroring `for-each-ref`, inviting discussion about ref-related command consolidation. Junio questions the justification for a new command versus evolving the existing one.  
- Interactive rebase worktree bug -- Phillip Wood investigates false "local changes" errors during interactive rebase in secondary worktrees, seeking more details about the triggering conditions.  
- Comment character selection fix -- Ayush Chandekar's patch fixing comment character selection during rebase conflicts nears merge after addressing final review feedback about test robustness.  
- Memory leak in send-pack -- A straightforward fix plugs a 2160-byte leak in the extra_have oid array during push operations, missed by existing tests.  

**On the radar**  
- SHA-256 default preparation -- Junio's "What's cooking" notes the `bc/use-sha256-by-default-in-3.0` series continues cooking, though not generating traffic today.  
- Reftable compaction -- Patrick Steinhardt's reftable work was recently mentioned in "What's cooking" as needing review attention.  

--- 

The day's traffic shows steady progress on technical improvements alongside persistent platform-specific challenges, particularly on Windows. Several threads appear close to resolution while new proposals begin their review cycles.
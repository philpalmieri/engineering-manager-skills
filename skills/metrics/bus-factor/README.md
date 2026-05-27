# Bus Factor Analysis Skill

**Purpose:** Realistic risk assessment for team coverage gaps and single points of failure

**Key Features:**
- Requires 3+ months of data by default (prevents misleading single-month snapshots)
- Distinguishes between "concentration" (primary owner with backups) vs "single point of failure" (no backups)
- Calculates meaningful contributor threshold: 2+ PRs OR 5+ reviews
- Provides actionable recommendations without shaming focused contributors

**Output:**
- Executive summary with risk breakdown (🔴🟡🟢)
- Per-repo coverage table with bus factor scores
- Team breadth analysis showing cross-functional vs specialized patterns
- Specific recommendations for building backup coverage

**Usage:**
- `run bus factor analysis` — analyze last 3 months
- `bus factor for Q1` — specific quarter
- `who can work on {repo}` — specific repo breakdown

**Integration:**
- Uses data from `team-data-sync` skill
- Part of `team-health-report` quarterly review
- Complements `review-network` and `review-cards`

**Key Insight:**
This skill emphasizes using 3+ months of data for realistic risk assessment. Single-month snapshots can be skewed by PTO, sprint focus, or work variation, leading to false conclusions about knowledge silos.

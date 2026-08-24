# OpenJUMP — Roadmap & Contribution Tracker

**Repo**: github.com/openjump-gis/openjump (mirrored: gitlab.com/openjump-gis; upstream listing
sourceforge.net/projects/jump-pilot/ — downloads/tickets only, dev happens on GitHub)
**Fork**: chrisdebian/openjump — remote `origin`; upstream is `upstream`
**Licence**: GPLv2 | **Build**: Maven | **Language**: Java (Swing UI) — no TypeScript/JS
migration exists or is planned (confirmed 2026-08-24)
**Onboarded**: 2026-08-24, via a self-sent "Claude Code" handoff email (see
`project_openjump_status` memory)
**Surveyed at**: commit `a385da2` (2026-08-10)

## Ground rules (from the onboarding email, Chris's own standing rules for this repo)

- British English throughout — code comments, commit messages, any docs.
- No AI/LLM signal in commits or PR descriptions (external project).
- No PR (or issue comment) submitted without building and running it locally first
  (`mvn -B package -P snapshot` at minimum).
- **Issue creation is restricted on this repo** — maintainers accept PRs, not new issues. Do not
  open new GitHub issues.
- No AI-drafted community-facing proposals (the "bitmagnet #468 lesson" — see
  `project_bitmagnet_status`). Every output here is a code change or a factual reproduction
  comment, not a proposal thread.
- One PR per checkbox item. Do not bundle unrelated changes.

## Fact-check (2026-08-24, fresh clone + live GitHub checks)

Extremely accurate — 9 of 10 checks confirmed exactly as claimed:

- Commit activity: real and active, most recent commit 2026-08-10 (14 days before this check).
- File counts: 1,548 `src` Java files, 48 `test` Java files — exact match.
- TODO/FIXME/HACK counts: 334/10/6 — exact match.
- `@Deprecated` usage: 35 files — exact match.
- CI: single Ubuntu job ("Java CI with Maven"), JDK 25 — exact match.
- log4j 1 still present in `pom.xml` (the `log4j`/`slf4j-log4j12` bridge), issue #49 still open —
  confirmed.
- No `CONTRIBUTING.md`/`CODE_OF_CONDUCT.md`/issue templates — confirmed, `.github/` only has
  `dependabot.yml` and `workflows/`.
- No TypeScript/JS migration plan anywhere in README/TODO.txt/Changes.txt — confirmed.
- All 6 target issues (#27, #47, #49, #61, #71, #115) confirmed **OPEN**; **zero open PRs** on the
  repo at present — confirmed.

**One correction**: issue #54 was listed among the open issues without a "verify" caveat (unlike
#106, which correctly had one) — #54 is actually **closed**, since 2022. #106 is also closed, as
the caveat anticipated. Current full open-issue count is **11**, not the ~13 loosely implied:
#27, #41, #43, #47, #49, #61, #65, #71, #115, #145, #147.

**Not yet checked**: whether `pom.xml`'s Java 8 `<source>`/`<target>` compiler settings (found at
survey time) are worth reconciling with the JDK 25 CI toolchain — not a contradiction (compiling
an old bytecode target with a newer JDK is normal), just noted for when Phase 2 (CI hardening)
touches this area.

## Priority order

Per the email's own phasing (Phase 0 fact-check is done, this is the order for what's left):

1. **Phase 1a — Issue triage** (reproduce-and-comment only, no fix required): #41, #43, #65,
   #145, #147 have not been assessed in any prior session. Cheap, useful to a low-bandwidth
   maintainer team, and a good way to build familiarity with the codebase before Phase 4's fixes.
2. **Phase 1 — Low-risk groundwork**: `CONTRIBUTING.md` (linking the OJ wiki setup page,
   confirmed live 2026-08-24), a minimal PR template, and a test-coverage-by-module map before
   Phase 5 invests in new tests.
3. **Phase 4 — Targeted bug fixes**, one PR each, only for the 4 issues already confirmed open
   and in scope: #71 (InfoFrame), #115 (selection tool — check relation to #71 first, same
   reporter/subsystem), #47 (GeoPackage extent), #61 (datastore layer not editable after detach).
4. **Phase 3 — Dependency/security debt**: log4j 1 → modern SLF4J binding (#49) — check extension
   repos for direct log4j 1 API usage first, not just the bridge, before removing it. Then the
   `@Deprecated` review, prioritising APIs actually removed upstream over merely-flagged ones.
5. **Phase 2 — CI hardening**: build matrix (windows-latest/macos-latest/ubuntu-latest), separate
   nightly workflow.
6. **Phase 5 — Test coverage**: core datastore/driver layer (#27/#47/#61), once Phase 1's module
   map confirms what's actually core vs. an abandoned extension.

## Phase 0 — Fact-check (done, 2026-08-24)

- [x] All 10 checks run against a fresh clone — see Fact-check section above. One correction
  found (#54), everything else confirmed accurate.

## Phase 1a — Issue triage (help without necessarily fixing)

- [x] #41 — Error Saving project (2022-01-23) — **commented 2026-08-24, root-caused**:
  `DataSourceFileLayerSaver.write()` (used by "Save dataset as (testing)" specifically, not the
  regular save path) stores the raw `java.net.URI` object under `DataSource.URI_KEY` in the
  layer's DataSource parameters (`DataSourceFileLayerSaver.java:51`); no `java2xml` binding exists
  anywhere for `java.net.URI`, so a subsequent project save throws exactly the reported exception
  when serialising that parameter. Confirmed the regular (non-testing) save path doesn't store a
  `URI_KEY` value this way, matching the report's own observation. No fix attempted — source-level
  root cause only, no Maven/display available to build and verify a fix end-to-end.
- [x] #43 — Modified Features (2022-01-29) — **commented 2026-08-24**: confirmed still present,
  verified via exhaustive source search — `BasicFeature.setModified(false)` is never called
  anywhere in the codebase (only `setModified(true)`, in `FeatureUtil`). Found and noted a
  related-but-distinct save-time reset (`AbstractSaveDatasetAsPlugIn` calls
  `layer.setFeatureCollectionModified(false)`) that clears a different, layer-level flag, not the
  per-feature one — likely why this could look partially fixed at a glance. Source-level check
  only, no Maven/display available to confirm via the actual UI.
- [ ] #65 — ECW support on OpenJUMP (2022-10-19) — not yet assessed
- [x] #145 — copy-paste style also copies SRID (2025-07-12) — **commented 2026-08-24**: verified
  the SRID-exclusion fix mukoki referenced (r5312) is genuinely present in current `main`
  (`PasteStylesPlugIn.java`, explicit `if (style instanceof SRIDStyle) continue;`). No Maven/
  display available in this environment, so this is a source-code verification, not a live UI
  reproduction — said so plainly in the comment. Mukoki's second, still-open question (why
  setting a new SRID detaches the datastore source) is unaddressed, flagged as such.
- [ ] #147 — Is it possible to increase WMTS rendering (2025-09-13) — not yet assessed

For each: reproduce against current `main`, comment confirming still-reproducible / no-longer-
reproducible / needs-more-info. Factual and minimal only — no restating the issue, no unsolicited
scope creep. Every "still reproducible" claim must be backed by an actual local run.

## Phase 1 — Low-risk groundwork

- [ ] `CONTRIBUTING.md` summarising the Maven/Eclipse/IntelliJ setup already documented at
  https://ojwiki.soldin.de/index.php?title=Eclipse:_Set_up_project_and_example_extension_from_git_sources
  (confirmed live 2026-08-24) — link, don't duplicate at length
- [ ] Minimal `.github/PULL_REQUEST_TEMPLATE.md` noting issue creation is restricted and PRs
  should reference existing tracked issues where relevant
- [ ] Map test coverage by module (core vs. plugins/extensions) before Phase 5 — check the
  README's extension migration status table for "Abandoned"/"Unmaintained" entries first, don't
  write tests for dead extensions

## Phase 2 — CI hardening

- [ ] Extend the Maven GitHub Actions workflow to a build matrix: windows-latest, macos-latest,
  ubuntu-latest. Re-confirm the JDK 25 CI pin still matches `pom.xml`'s compiler settings first.
- [ ] Separate scheduled/nightly workflow so longer-running checks don't block PR builds

## Phase 3 — Dependency/security debt

- [ ] Scoped PR replacing the log4j 1 → `slf4j-log4j12` bridge with a modern SLF4J binding
  (logback or log4j2-slf4j-impl). Grep all extension repos under the `openjump-gis` org for
  direct log4j 1 API usage (not just SLF4J) before removing the bridge outright — a removal that
  breaks extensions is worse than leaving it. Reference issue #49.
- [ ] Review the 35-file `@Deprecated` usage list — prioritise anything tied to APIs actually
  removed upstream (JTS namespace, ImageIO SPI changes) over ones merely flagged for future
  removal

## Phase 4 — Targeted bug fixes

Only for issues confirmed open in the fact-check above. Each its own PR. Reproduce locally
against current `main` before writing a fix — these are from 2021–2024 and may be stale or
partially addressed already.

- [ ] #71 — InfoFrame inconsistent behaviour
- [ ] #115 — Selection tool unexpected behaviour (check relation to #71 first — same reporter,
  same UI subsystem; confirm before treating as one fix)
- [ ] #47 — `DataStoreDataSource` not reading GeoPackage table extent
- [ ] #61 — Datastore layer not editable after source detached
- [ ] #27 — Improve core database driver, get rid of db extensions (larger, likely feeds Phase 5)

## Phase 5 — Test coverage

- [ ] Unit tests around the core datastore/driver layer exercised by #27/#47/#61, using Phase 1's
  module map to confirm this is core (not an abandoned extension) before investing time

## Technical Debt

Per project policy, only markers with a concrete contribution in mind are tracked in detail here
— external project, not batch-filing the maintainer's own legacy debt. Full counts at survey time
(2026-08-24): 334 TODO, 10 FIXME, 6 HACK across `src` (non-test), 35 files using `@Deprecated`.
See Phase 3 above for the prioritisation approach once picked up.

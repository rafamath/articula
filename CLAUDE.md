# Articula: rules for every session

Full background: `docs/kickoff.md`. Decisions: `docs/adr/`.

## Product

Free, open-source (AGPL-3.0) Expo app for iOS and Android. People with autoimmune and inflammatory
joint conditions log which joints hurt, how much, for how long and how they look, then generate a
PDF report for their doctor. Rafael (product owner) is the first user and uses it daily from M1:
**real data must survive every update.**

- **Log, don't interpret.** No suggestions, scores, diagnoses or "insights" anywhere. Facts derived
  directly from logs (e.g. "both knees affected in the same period") are allowed.
- **Usable in pain.** A log takes under 30 seconds: large targets, minimal typing.
- **Accessibility from day one.**
- **Local-first.** No account, no server, no analytics, no network calls in v1.
- **Free forever.** No ads, paywalls or in-app purchases.
- Out of scope for v1: voice, server, accounts, doctor portal, triggers, medication, weather, AI summaries.

## Architecture

- **Repository layer.** UI and domain code never touch SQLite. All storage goes through repository
  interfaces.
- **Sync-ready records.** UUID primary keys; timestamps stored in UTC, with the user's time zone
  stored where display needs it; `createdAt` / `updatedAt` on every record; soft delete via `deletedAt`.
- **Versioned migrations.** Never break existing data. Every migration has a test that runs it on a
  database at the previous version.
- **Commands.** Every user action is a plain domain command (`startFlare`, `updateFlare`,
  `endFlare`, `logMorningStiffness`, `attachPhoto`, ...). UI calls commands; so will voice in phase 2.
- **Joint catalog as data.** One typed catalog (id, display name key, region, side, view, parent
  region) shared by body map, database and report. IDs look like `knee.left`, `hand.right.pip3`,
  `spine.lumbar`. IDs are stable forever once data uses them.
- Prefer Expo-maintained libraries. Justify every library choice in an ADR.
- All user-facing strings go through i18n. No hard-coded copy.

## Accessibility (blocking, not optional)

- Tap targets at least 44x44 pt (iOS) / 48x48 dp (Android), larger where possible.
- Screen reader label on every interactive element, including every joint.
- Layouts work at the largest system text size.
- WCAG AA contrast.
- Never rely on color alone (heatmaps also show numbers or patterns).
- A milestone closes only after each feature is checked on a real device with VoiceOver / TalkBack
  and the largest text size.

## Working agreement

- Rafael reviews and merges every change. **Never push to main, never merge your own PR, never
  bypass hooks or branch protection.**
- One task = one issue = one branch = one PR. PR body: `Closes #<issue>`, how it was tested,
  screenshots for UI changes.
- **Acceptance tests first:** write tests from the issue's acceptance criteria before implementing.
- Re-verify any claim about the code against the code itself, not comments, docs or memory.
- Keep tasks under about half a day. If one grows, stop and propose a split.
- Ambiguous? Ask. Record the decision in the issue or an ADR.
- Don't polish the body map (M2) or the report (M4) before their milestones.
- Before accepting any external PR, raise the AGPL / App Store / CLA question.

## Tooling and GitHub

- Install Expo packages with `npx expo install`, never plain `npm install`.
- Before any GitHub write, check that `gh api user --jq .login` returns `rafamath`.
- Commits use the repo-local GitHub noreply identity (`git config user.email`), never a personal email.
- EAS projects and builds go under the Expo team account, never a personal one.
- GitHub settings changes, deletions or bulk creation: list the steps as a, b, c and wait for an
  explicit "yes a, b, c".
- When a command fails, capture and show the error before retrying, and mention it in the summary.
- Board: when a PR opens, set its issue to In review. After Rafael merges, ask him for Actual (h),
  Review rounds and First-pass accepted, then fill them in.

## Measurement

The project also evaluates the AI-assisted workflow. The GitHub Project fields are the single
source of truth (no separate log). Per task record: Estimate (h), Actual (h), Review rounds,
first-pass acceptance, Escaped bugs (linked to the original issue), spec drift, AI cost ($).
Notes go in the PR description. Lead time, change failure rate and rework are derived from GitHub
data with `gh` (script added in M1).

**Spec drift:** if a change does anything the issue didn't ask for, say so in the PR.

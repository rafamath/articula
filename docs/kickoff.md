# Articula: Kickoff Brief

> Original brief for the first Claude Code session, kept as the project's reference.

## 1. What we're building

Articula is a free, open-source mobile app (iOS and Android) for people with autoimmune and inflammatory joint conditions (rheumatoid arthritis, psoriatic arthritis, lupus, ankylosing spondylitis and others). Users log which joints hurt, how much, for how long, and what the joint looks like, then generate a report to show their doctor.

The product owner (Rafael) lives with joint pain and is the first user. The app will be used daily from Milestone 1 onward, so real data must survive every update.

### Principles

- **Log, don't interpret.** No suggestions, scores, diagnoses or "insights". Interpretation is the doctor's job. Facts derived directly from the logs (for example "both knees affected in the same period") are allowed.
- **Logging must work for someone in pain.** Under 30 seconds per log, large tap targets, minimal typing.
- **Accessibility from day one**, not bolted on later.
- **Local-first.** All data stays on the phone. No account, no server, no analytics in v1.
- **Free forever.** No ads, no paywalls, no in-app purchases.

## 2. Product decisions (v1)

### Body map

- Front and back views of a simple, stylized figure (not realistic anatomy).
- Tapping a hand or foot zooms into a detailed view with every small joint: MCP (metacarpophalangeal), PIP (proximal interphalangeal), DIP (distal interphalangeal), thumb IP (interphalangeal), wrist, and in the foot MTP (metatarsophalangeal) joints, toes, ankle.
- Back view includes spine regions (cervical, thoracic, lumbar) and sacroiliac joints, plus shoulders and hips.
- A legend makes left and right unambiguous ("your left / your right").
- Every joint region is an SVG (Scalable Vector Graphics) shape with a stable ID and a screen reader label.
- The same SVGs are reused to draw the report heatmap.

### Onboarding

- Ask for the condition(s), allowing multiple and including "Not diagnosed yet / Not sure". It can be changed in settings.
- The condition only changes emphasis (which regions are expanded or shown first). It never hides joints.

### Flares (core flow)

- Select joint(s), tap Start, set pain level. Allow backdating the start ("started yesterday evening").
- Pain uses the 0 to 10 NRS (Numeric Rating Scale) with labels at key points (0 none, 5 moderate, 10 worst imaginable).
- Optional characteristics per joint: redness, tenderness, swelling, warmth.
- While a flare is ongoing, the user gets reminders (configurable) answered with one tap: Better / Same / Worse / Ended. Updates can also adjust the pain level.
- Optional notes and photos attached to a flare update. Photos are stored inside the app (not the camera roll) and compressed, and each one has an "include in report" toggle.

### Morning stiffness

A daily entry in minutes.

### Report (PDF, generated on the phone)

- Date range picker (default: since the last report).
- Body heatmap of the most affected joints.
- A written table per joint: dates, duration, peak and typical pain, characteristics.
- Flare timeline and morning stiffness trend.
- Symmetry shown as a derived fact.
- Selected photos with dates.
- No interpretation anywhere.

### Trust and safety

Optional app lock (Face ID / fingerprint / passcode), encrypted backup export and import to a file the user controls (iCloud Drive, Google Drive), delete all data, and a "not a medical device, does not diagnose" disclaimer.

### Out of scope for v1

Voice input (phase 2), any server, accounts, doctor portal, population reports (phase 3), triggers, medication tracking, weather, AI summaries.

## 3. Tech stack

- Expo (React Native) + TypeScript (strict mode), iOS and Android from one codebase.
- expo-sqlite for storage, expo-notifications for local reminders, expo-print for PDF, expo-image-picker for photos, expo-local-authentication for the app lock, expo-file-system for backup files.
- react-native-svg for the body map.
- i18n (internationalization) set up from M0. English first, Portuguese and Spanish later.
- Testing: Jest + React Native Testing Library for unit and component tests. Choose an end-to-end tool (Maestro or Detox) in an ADR (architecture decision record) during M0.
- Pick and justify libraries in ADRs. Prefer the Expo-maintained option when one exists.
- Expo Go vs development build: some features (for example Face ID via expo-local-authentication on iOS, some notification behavior) may not fully work in Expo Go. Decide in an M0 ADR when to move to a development build, so it doesn't surprise us in M3 or M5.
- Build costs: EAS (Expo Application Services) cloud builds have a limited free tier. CI runs lint, type checks and tests on every PR. Cloud builds run only when needed (for example releases or device testing), not on every PR.

## 4. Architecture rules

- **Repository layer.** UI and domain code never touch SQLite directly. All storage goes through repository interfaces, so a sync backend can be added later without rewriting features.
- **Sync-ready data**, even though there is no sync yet: UUID (universally unique identifier) primary keys, all timestamps in UTC (Coordinated Universal Time) with the user's time zone stored where needed for display, createdAt / updatedAt on every record, and soft delete (deletedAt).
- **Versioned migrations from M1.** Never break existing data. Each migration gets a test that runs it on a database from the previous version.
- **Command-style actions.** Every user action is a plain domain command (startFlare, updateFlare, endFlare, logMorningStiffness, attachPhoto...) so voice input can call the same commands in phase 2.
- **Joint catalog as data.** One typed catalog (ID, display name key, region, side, view, parent region) shared by the body map, the database and the report. Example IDs: `knee.left`, `hand.right.pip3`, `spine.lumbar`.

Draft data model (refine in an ADR):

- Flare: id, startedAt, startedAtEstimated, endedAt, notes, timestamps.
- FlareJoint: flareId, jointId.
- FlareUpdate: id, flareId, recordedAt, status (start / better / same / worse / ended), notes.
- JointObservation: id, updateId, jointId, pain (0 to 10), redness, tenderness, swelling, warmth.
- Photo: id, updateId, jointId (optional), fileUri, includeInReport.
- MorningStiffness: id, date, minutes.
- Settings: conditions, reminder preferences, lock enabled.

## 5. Accessibility requirements

- Minimum tap target 44x44 pt (iOS) / 48x48 dp (Android), larger where possible.
- A screen reader label on every interactive element, including every joint.
- Support the largest system text sizes without breaking layouts.
- Meet WCAG (Web Content Accessibility Guidelines) AA contrast.
- Never rely on color alone (the heatmap also shows numbers or patterns).
- Every feature is checked on a real device with VoiceOver / TalkBack and the largest text size before its milestone closes.

## 6. Milestones

Each milestone ends with something working on a real phone.

- **M0: Foundations.** Repo, license, Expo + TypeScript, lint, type check, tests, CI on GitHub Actions, i18n scaffold, ADRs (including Expo Go vs development build), Project board with metric fields. Done when: the app runs on iOS and Android and CI is green.
- **M1: Walking skeleton.** A crude front body map, start / list / end a flare with a pain level, SQLite behind the repository layer, migrations, a bare-bones PDF. Done when: Rafael logs a real flare and generates a report. Daily use starts here.
- **M2: Body map.** Front and back views, hand and foot zoom, full joint catalog, left/right legend, accessibility labels, onboarding with condition emphasis.
- **M3: Rich logging.** Characteristics, backdating, one-tap updates, reminders, morning stiffness, notes, photos.
- **M4: Doctor report v1.** Date range, heatmap, written per-joint table, trends, derived symmetry, photo selection. Done when: ready to show Rafael's doctor (feedback checkpoint).
- **M5: Trust and safety.** App lock, encrypted backup export and import, delete all data, disclaimer. Required before anyone else uses the app.
- **M6: Release.** Accessibility audit, TestFlight and Google Play internal testing, store listings and privacy disclosures, optional translations.

Don't polish the body map or the report in M1. M2 and M4 own that work.

## 7. GitHub setup

Account: rafamath.

- Check `gh auth status`. Creating a GitHub Project needs the `project` scope: if it's missing, run `gh auth refresh -s project`.
- Create a public repo rafamath/articula with an AGPL-3.0 license, a README (purpose, principles, status, "not a medical device"), and .gitignore for Expo.
- Create milestones M0 to M6 with the descriptions and exit criteria above.
- Create one epic issue per feature area per milestone as placeholders. Only M0 and M1 epics get broken into task sub-issues (GitHub sub-issues) now. Later milestones are groomed when they come up, to avoid speculative issues. Labels: epic, task, bug, adr, a11y, plus area labels (area:body-map, area:logging, area:report, area:storage, area:infra).
- Add an issue template for tasks with these sections: Context, Acceptance criteria (testable), Out of scope, Estimate (h).
- Create a GitHub Project (board) for the repo with status columns Backlog / Ready / In progress / In review / Done and custom fields: Estimate (h), Actual (h), Review rounds, Escaped bugs, AI cost ($).
- Turn on branch protection for main: PRs required, CI must pass, no direct pushes.

## 8. Working agreement

- Rafael reviews and merges every change. Never push to main, never merge your own PR, never bypass hooks or branch protection.
- One branch and one PR per task. The PR body says `Closes #<issue>` and includes how it was tested and screenshots for UI changes.
- Acceptance tests first: write tests from the issue's acceptance criteria before the implementation, so "done" is decided by tests, not by judgment.
- Before claiming something about the code (from comments, docs or memory), re-verify it against the actual code.
- Keep tasks small (ideally under half a day). If a task grows, stop and propose a split.
- When something is ambiguous, ask instead of guessing, and record the decision in the issue or an ADR.

## 9. Workflow measurement

The project is also an evaluation of the AI-assisted workflow. It follows the DORA (DevOps Research and Assessment) and DX AI Measurement Framework ideas, scaled down to one person. Feelings about speed are not evidence, so measure from recorded data.

The Project fields are the single source of truth for these numbers (no separate log file to keep in sync). Notes and context go in the PR description. Per task, record:

- estimate vs actual time
- review rounds until accepted
- first-pass acceptance (yes/no)
- escaped bugs (found after merge, linked to the original issue)
- spec drift (did the change do anything the issue didn't ask for?)
- AI cost

Derived from GitHub data later with `gh`: lead time (issue opened to PR merged), change failure rate, and rework (lines changed again within 14 days). Add a small script for this in M1.

## 10. First session

- Do the GitHub setup in section 7.
- Add CLAUDE.md to the repo summarizing sections 1, 4, 5, 8 and 9, and add this brief as docs/kickoff.md.
- Groom M0 (and a first pass of M1) into tasks (issues with acceptance criteria and estimates) and propose them. Stop and wait for Rafael's approval before implementing anything.

## 11. Open questions to raise when relevant

- **AGPL and the App Store:** Apple's terms are known to conflict with GPL-family licenses for code from outside contributors. As the sole copyright holder Rafael can still publish his own app, but accepting outside contributions may need a CLA (contributor license agreement). Flag this before accepting any external PR.
- The end-to-end test tool (Maestro vs Detox).
- Encryption approach for backup files.
- Name availability: check "Articula" on the App Store, Google Play and in a trademark search before M6.

## Notes for setup

- When you scaffold the Expo project, merge Expo's generated CLAUDE.md with ours instead of overwriting either one.
- When linking the app to EAS, create the project under the Expo team account, not the personal account.

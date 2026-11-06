# Roadmap

A proposal, not a commitment, and not yet agreed with the maintainer. The order
is deliberate: **make shipping safe, then earn trust, then add reach.** New
features reach few people while installing shows warnings and existing users
cannot learn about updates.

Effort is a rough guess: S is under a day, M is a few days, L is a week or more.

## Now: make shipping safe (0.2)

Cheap work that prevents expensive mistakes.

| Item | Effort | Why |
|---|---|---|
| Run `npm run lint` in CI | S | CONTRIBUTING asks for it, and nothing enforces it |
| Require passing tests before `release.yml` builds | S | `v0.1.0` went out ahead of green CI |
| Enable private vulnerability reporting | S | The app parses untrusted files, and there is no private channel |
| Add issue and pull request templates | S | Asks for the system and `.wajha` file up front; templates get a gallery checklist |
| Verify the store sizes against Apple and Google docs | S | "Exact sizes" is the core promise ([open question 3](MEMORY.md#open-questions)) |
| Remove dead code and assets | S | See the [known gaps](MEMORY.md#known-gaps-verified-2026-11-06) |
| Script the version bump across all three files | S | Avoids mismatched versions |
| Fail CI when bundle size exceeds its budget | S | The budget exists on paper only |
| Release 0.2.0 using [docs/RELEASING.md](docs/RELEASING.md) | S | Delivers the fixes and the full-size template preview |

## Next: earn trust and keep users current (0.3)

| Item | Effort | Why |
|---|---|---|
| macOS signing and notarization | M + cost | Removes the Gatekeeper warning |
| Windows code signing | M + cost | Reduces SmartScreen warnings |
| **Update notification, on by default with a clear off switch and disclosure** | M | Ends the old-client problem at its source |
| Version the gallery path and define format rules | M | Lets formats change without breaking old clients ([SCALING §1](docs/SCALING.md#1-old-clients-never-go-away)) |
| Run a real format migration in a test | S | The migration path has never been exercised |
| Scheduled store rule review | S | A calendar issue each quarter and before Apple's autumn releases |

**Recommendation on updates:** start with a notify-only check of a static file, not
silent auto-install. It is simple and keeps the user in control, and it needs no
updater signing key on day one. It does add a network request, so the README must
say so. This needs the maintainer's agreement because it touches a stated promise.

## Then: reach and leverage (0.4 and on)

Ordered by value for effort.

1. **Headless CLI (L): the biggest untapped opportunity.** The engine is already
   pure, and `scripts/gallery-previews.test.ts` renders real templates in Node. A
   `wajha export project.wajha --out dir` command would put screenshots in CI
   pipelines next to fastlane, which no GUI can do. Ship it as a separate package,
   because `@napi-rs/canvas` is native and must not enter the app. The risk is that
   Node and WebKit render text differently, so compare pixels before promising
   identical output.
2. **Faster, resumable large exports (M).** See
   [SCALING §3](docs/SCALING.md#3-export-time-grows-multiplicatively).
3. **More interface languages (M).** `UiLanguage` is hard-coded to `en | ar`.
   Generalize it and add a translator guide, starting with languages where
   contributors appear.
4. **More gallery templates (ongoing).** Cheapest way to attract contributors.
   Add a "good first template" call and credit authors visibly.
5. **Refresh a project from new captures (M).** Folder sorting already exists, so
   re-importing fresh captures into an existing project is a short step, and it
   makes Wajha part of each release instead of a one-off.
6. **Discovery (ongoing).** A project site with example output, posts in Arabic and
   English developer communities, and a "made with Wajha" gallery.

## Needs a decision first

- Optional bring-your-own-key AI translation ([open question 4](MEMORY.md#open-questions)).
- Hosted web version ([open question 5](MEMORY.md#open-questions)).
- Direct upload to App Store Connect and Google Play. Useful, but it means handling
  store credentials, and the fastlane layout already covers much of the need.

## Not planned

Accounts, cloud sync, telemetry, Electron, and general-purpose design features. See
[PROJECT.md](PROJECT.md#non-goals).

## How to know it is working

There is no telemetry and there should not be, so use public signals only: downloads
per release (GitHub API), share of downloads on the latest version, outside pull
requests, gallery templates added, issues opened, and stars. Asking users directly
will tell you more than any of these.

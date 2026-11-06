# Project memory

What a new contributor or AI assistant would otherwise have to rediscover: why
things are the way they are, traps, and open decisions.

**Keep this useful.** Add what the code and git history cannot tell you. Date
each entry. Delete what stops being true. Do not copy in architecture details;
link to them instead.

## Decisions and why

Mostly inherited from the original project. Its reasoning is inferred from the code
and docs, and nobody has confirmed it with the original author.

| Decision | Why | Where |
|---|---|---|
| Tauri, not Electron | Small download and low memory are features | ARCHITECTURE §2 |
| Canvas 2D with no rendering library | The same engine runs in the app and in Node tests, with no dependency | `src/engine/` |
| Layouts in fractions, never pixels | One design serves every export size | ARCHITECTURE §5 |
| Non-primary layouts are derived, unless edited | Edits to the primary family flow to the others until a family is customized | ARCHITECTURE §6.1 |
| Layouts are never stored mirrored | RTL is computed at render time, and edits map back through `mirrorPatch` | `src/model/mirror.ts` |
| One export job at a time | A full-size render plus PNG is tens of MB, so memory stays bounded | `src/export/runner.ts` |
| Hand-written validators, no schema library | No dependency; every external file is validated | `src/model/validate.ts` |
| Gallery on jsDelivr from the repo | No hosting cost, and contributions are pull requests | `src/config.ts` |
| Actions pinned to commit SHAs | A moved tag cannot change what runs with write access | `.github/workflows/` |
| npm and Cargo updates by hand | Dependabot covers only GitHub Actions; deliberate given the dependency policy | `.github/dependabot.yml` |
| English text is the translation key | Simple; `i18n.test.ts` fails if an Arabic entry is missing | `src/i18n/` |
| AI tool files are git-ignored | The original project keeps `CLAUDE.md`, `AGENTS.md` and similar out of the repo | `.gitignore` |
| Run as an independent fork (2026-11-06) | Rang0x chose to own this version rather than contribute upstream | `PROJECT.md` |

## Traps

- **Old clients stay in use.** There is no updater, and the gallery URL, gallery
  format and template format are effectively a public API. Read
  [docs/SCALING.md](docs/SCALING.md#1-old-clients-never-go-away) before touching
  them.
- **Validators drop unknown keys.** A new field survives only in versions that know
  about it. If an older version opens and saves the file, the field is gone with
  no error.
- **A new enum value is a breaking change for old clients.** `frame`, for example,
  goes through `enumField` against a fixed list.
- **No `ctx.filter`.** The WebKit targets do not support it. Glows use `shadowBlur`.
- **WebKit is the test target, on purpose.** It is what macOS and Linux run. A
  change that works in Chrome has not been tested.
- **Node rendering is not WebKit rendering.** Tests draw with `@napi-rs/canvas`, so
  they do not prove WebKit pixels match. `e2e/text-baseline.spec.ts` exists because
  WebKit shifted mixed-direction text.
- **No regex lookbehind.** Regexes avoid lookbehind because WKWebView before
  Safari 16.4 cannot parse it (`src/model/rules.ts`).
- **`README.md` promises one network request.** Anything new that calls out, such
  as an updater, makes that claim false. Change the claim and get agreement first.
- **The version lives in three files:** `package.json`, `src-tauri/tauri.conf.json`
  and `src-tauri/Cargo.toml`.

## Known gaps (verified 2026-11-06)

- `npm run lint` is not run in CI, although CONTRIBUTING asks for it.
- `scripts/desktop-smoke.sh` is not run in CI; only its build is tested.
- The size and speed budgets (ARCHITECTURE §8) are not enforced anywhere.
- `release.yml` does not require CI to pass. `v0.1.0` was tagged at `eb7575a`; CI
  on `main` then failed twice, and the next commits were CI fixes.
- No code signing or notarization, no updater, no crash reporting.
- GitHub private vulnerability reporting is off.
- No issue or pull request templates and no code of conduct.
- `MIGRATIONS` is empty, so the upgrade path has never run on a real format change.
- Probably dead: `src/model/stub.ts` (`notImplemented` is defined but not called),
  and `src/assets/hero.png` and `vite.svg` (not referenced from `src/` or
  `index.html`). The rule about `notImplemented` in ARCHITECTURE §4 goes with it.
- Largest non-test modules, to split when next changed: `src/store/editor.ts`
  (1,520 lines), `src/ui/kit/FillEditor.tsx` (1,095), `src/model/adapt.ts` (889).

## Open questions

Each needs a decision from Rang0x.

0. **Name and app identifier: settle before the first release.** The build still uses
   the name Wajha and the identifier `studio.wajha.app`, inherited from the original.
   With the same identifier, this build and the original share a data folder on one
   machine, and the operating system may treat them as the same app. Changing it after
   release would strand users' projects unless the app migrates them. Keeping the name
   may also confuse people about which version they have.
1. **Updater:** add an auto-updater, a notify-only check, or neither? It conflicts
   with the README's network promise, so it should be opt-in or disclosed.
   Recommendation in [ROADMAP.md](ROADMAP.md).
2. **Signing budget:** who pays for an Apple Developer account and Windows signing,
   and whose identity signs? Sponsorship could fund it.
3. **Store rule accuracy:** ARCHITECTURE §6.5 says Apple requires both 6.9″ and
   6.3″ iPhone sizes, and `catalog.ts` marks both `required`. Not checked against
   Apple's current documentation. Verify it, because it decides the default export.
4. **AI translation:** would an optional bring-your-own-key feature fit "offline
   first"? It is a large time saver, but also the first feature to send user text
   to a third party.
5. **Hosted web version:** the browser build already works. Hosting it could be a
   strong discovery channel, but it adds a service to run, and browser storage is
   more fragile than files.
6. **Governance:** who else can review and merge, and who holds release rights?

## Log

- **2026-11-06:** Became an independent fork under `Rang0x/Wajha`. `REPO_OWNER` and
  the package metadata now point here, so the gallery and in-app links use this repo.
  GitHub Issues were turned on. `origin` is the fork and `upstream` is the original.

- **2026-11-06:** Cloned the repo, reviewed the code, docs, CI and GitHub state, and
  added `PROJECT.md`, `MEMORY.md`, `ROADMAP.md`, `docs/SCALING.md`,
  `docs/RELEASING.md`, `SECURITY.md` and `CHANGELOG.md`. Listed the `watch`, `tv`
  and `vision` frames in the existing docs. No app code changed. Tests were not run
  locally; "CI passing" comes from GitHub's run history.

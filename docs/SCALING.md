# Scaling Wajha

Wajha has no servers, so "scale" does not mean traffic. It means four other things:
**old versions in the wild, the gallery, project size, and the number of people
working on the code.** This document lists what breaks first and what to do about
it.

Unless marked as measured, numbers are the project's stated targets or arithmetic.
No profiling was done for this document, so measure before optimizing.

## 1. Old clients never go away

**This is the most important constraint in the project.** With no updater, every
released version may run indefinitely, and it fetches this repository's `main`
branch:

```
https://cdn.jsdelivr.net/gh/M-Alkassem/Wajha@main/gallery/
```

So these are public APIs, even though they look like internal files:

| Surface | What breaks it |
|---|---|
| `gallery/` path, on `main` | Renaming the folder, the repo or the owner stops the gallery in every shipped version |
| `gallery/index.json` format | Bumping `version`: `migrate()` rejects newer versions, so old apps lose the whole gallery, not just new entries |
| `template.json` format | A template using a new `frame` or other enum value fails validation on old apps |
| `.wajha` project format | Version 2 files are refused by version 1 apps, with a message telling the user to update |

Also, validators **drop unknown keys**. If a new field is added without a version
bump, a project saved by an older app silently loses it.

### Rules for format changes

1. **Additive and safe to lose:** an optional field with a default. No version bump
   needed. Confirm that losing it on an older app is acceptable.
2. **Anything else:** bump `version` and add a step to `MIGRATIONS` in
   `src/model/validate.ts`, with a test that reads a real file in the old format.
3. **Keep old fixtures.** Save a sample `.wajha` and `.wajha-template` per format
   version as test inputs, and never edit them.
4. **Changing the gallery format:** do not change the existing one in place. Publish
   the new one beside it (`gallery/v2/`) and keep generating the old index for old
   clients, limited to templates they can read.
5. **Never move or rename** the repo or `gallery/` without a redirect plan.

### What to build

- The update notification ([ROADMAP](../ROADMAP.md)). It does not remove this
  problem, but it shrinks the population of old clients over time.
- Consider pinning new versions to a tag or versioned path rather than `@main`. Then
  a bad merge cannot reach every user at once, and changes can be staged.

## 2. The gallery

Today: 4 templates and a 2.4 KB index. Roughly 0.6 KB per entry, so the app's 8 MB
index limit (`MAX_INDEX_BYTES`) is thousands of templates away. **The index size is
not the first thing to break.** These are, in likely order:

| Pressure | When | Response |
|---|---|---|
| Review effort | First | CI already validates; add a PR template, and automate preview rendering so contributors cannot forget it |
| Merge conflicts | At several open PRs | `index.json` and `.hashes.json` are generated files every PR edits. Generate them on merge with a bot instead |
| Browsing | Dozens of templates | The picker needs search and category filters that work well, and sorted or featured entries |
| Repo size | Hundreds of templates | Each preview may be up to 400 KB and stays in git history forever. Lower the cap, or move previews elsewhere |
| CDN caching | Any time | jsDelivr caches branch URLs, so changes do not appear immediately. Check its current limits on repo and file size before growth depends on it |

The trust model should be reviewed too: gallery files are validated, hashed and
size-limited, but nothing signs them, so write access to `main` equals the ability
to publish to all users. Require reviews on `main` and consider signed releases of
the index.

## 3. Export time grows multiplicatively

`jobs = slides x languages x sizes`, run one at a time on purpose
(`src/export/runner.ts`). At the 1.5 s per image target:

| Project | Jobs | Time at target |
|---|---|---|
| 5 slides, 3 languages, 3 sizes | 45 | about 1 minute |
| 8 slides, 10 languages, 4 sizes | 320 | about 8 minutes |
| 8 slides, 30 languages, 4 sizes | 960 | about 24 minutes |
| 10 slides, 51 languages, 6 sizes | 3,060 | about 77 minutes |

In order of value for effort:

1. **Measure first.** Add a benchmark on real hardware and track it. The target has
   not been verified here.
2. **Incremental export.** Hash each job's inputs and skip images that have not
   changed. Most re-exports change little, so this probably beats parallelism and
   uses no more memory.
3. **Resume after failure or cancel.** Falls out naturally from the hashing above.
4. **Encode in parallel.** PNG encoding already runs in a worker, so two or three
   encoders with a bound on bytes in flight may help. Keep the memory guarantee,
   because it is why the runner is sequential.

## 4. Project size in the editor

The target is smooth editing at 20 slides x 3 languages, with a 192 MB budget for
decoded previews. Risks as projects grow:

- The stage draws every slide of the active family side by side. Check that offscreen
  slides are skipped.
- Undo keeps up to 200 steps. Immer shares structure, so this should be cheap, but
  verify with a large project and a heap snapshot.
- `project.json` is capped at 16 MB and `.wajha` at 500 MB. Many high-resolution
  screenshots per language could reach the second limit. Check the failure message
  is clear.

**Enforce the budgets.** ARCHITECTURE §8 lists them, but nothing checks them, so they
will erode unnoticed. Start with bundle size in CI, then installer size, then a
render benchmark that only reports.

## 5. Store rules change

`src/model/catalog.ts` is the app's most time-sensitive file. New devices and rule
changes can make exports wrong or rejected, and old clients keep the old rules.

- Review on a schedule, and every time Apple announces new devices.
- Record the source and review date next to the sizes.
- Consider a small sizes file served like the gallery, so old clients can pick up
  new sizes. It adds complexity, so weigh it after the updater exists.

## 6. More contributors

Docs and tests are strong for a project this young. The gaps are around people.

- **One maintainer** is the largest single risk. Add a second person with merge and
  release rights, and protect `main`.
- **Lower the barrier.** Label good first issues; template and translation work needs
  no Rust knowledge.
- **Split large modules when next changing them, not as a project of its own:**
  `src/store/editor.ts` (1,520 lines) by domain (slides, elements, languages,
  history), `src/ui/kit/FillEditor.tsx` per fill type, `src/model/validate.ts` per
  document kind.
- **Interface translation:** `src/i18n` handles exactly `en | ar`, using literal checks
  such as `current === 'ar'`. Before adding a third language, make the catalog
  generic, pick the RTL flag from language data, and let `i18n.test.ts` report
  coverage without failing on partial languages.
- **Hard-coded repository:** `REPO_OWNER` in `src/config.ts` means a fork's builds
  still read the upstream gallery unless `VITE_GALLERY_URL` is set. Document this for
  forks.

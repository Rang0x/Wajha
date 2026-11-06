# Wajha: project brief

The one page to read first. For how the code works, read
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md), which stays the contract between
modules. This file covers what the project is and where it is going.

| Doc | Answers |
|---|---|
| `PROJECT.md` | What is this, who is it for, what will it never be? |
| [`MEMORY.md`](MEMORY.md) | What has been decided, what was learned, what is still open? |
| [`ROADMAP.md`](ROADMAP.md) | What comes next, and why in that order? |
| [`docs/SCALING.md`](docs/SCALING.md) | What breaks as users, templates and project sizes grow? |
| [`docs/RELEASING.md`](docs/RELEASING.md) | How do we ship a version? |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | How does the code fit together? |
| [`docs/TEMPLATES.md`](docs/TEMPLATES.md) | How do templates and the gallery work? |
| [`SECURITY.md`](SECURITY.md) | What do we defend against, and how do you report a problem? |
| [`CHANGELOG.md`](CHANGELOG.md) | What changed in each version? |

## Ownership

This is an independent version maintained by [Rang0x](https://github.com/Rang0x),
forked on 2026-11-06 from [M-Alkassem/Wajha](https://github.com/M-Alkassem/Wajha).
The original work is MIT licensed, and its license and copyright notice must stay in
this repo. To bring in changes from the original: `git fetch upstream`, then
`git merge upstream/main`.

## The idea

Store screenshots are what people see before they install an app, and making
them is repetitive: every store wants its own sizes and every language needs
its own copy. Multiply slides by sizes by languages and a small app quickly
needs hundreds of images.

**Wajha** (واجهة, "façade" or "storefront") is a free, open-source desktop app
that reduces this to one design. Pick a template, drop in screenshots, write
headlines, and export every size for both stores in every language in one click.

The key design choice is that **a project is a design, not a set of images**.
Layouts are stored in fractions of the canvas, text is stored per language, and
screenshots per device family and language with fallback to the base language.
The pixels are only produced at export. Almost every feature follows from this.

## Who it is for

- **Indie developers and small teams** who ship on iOS or Android and would
  rather not pay for, or sign up to, a design tool for a task they do a few times
  a year.
- **Teams localizing into right-to-left languages**, where a mirrored layout and
  correct shaping are hard to get from generic tools.
- **Teams already using fastlane**, who can use the `deliver` and `supply` export
  layout directly.

## Why it could win

1. **RTL is built in, not added on.** Arabic, Hebrew, Persian and Urdu get proper
   shaping, and slides are mirrored because the stores list those languages from
   the right. The Arabic name signals who it was built by and for. This is the
   most distinctive thing about the project.
2. **Free, offline and private.** No account and no telemetry. A real position,
   but also a constraint on the roadmap (see below).
3. **Small.** Release installers are 2 to 5 MB, except the 78 MB AppImage.
4. **Open templates.** The gallery lets the community add designs through pull
   requests, at no hosting cost.

## What could hold it back

1. **Trust at install.** Builds are unsigned, so macOS and Windows both warn. This
   is probably the largest drop-off, though nothing measures it.
2. **No update path.** The app cannot tell users a new version exists, so old
   versions stay in use, and they constrain every format change
   ([docs/SCALING.md](docs/SCALING.md#1-old-clients-never-go-away)).
3. **Store rules change.** The promise of "exact store sizes" lasts only while
   someone keeps `src/model/catalog.ts` current, and no process does that yet.
4. **One maintainer.** This version depends on one person, and it must also earn its
   own audience next to the original project.

No competitor or market research has been done for this brief. "Why it could
win" is reasoning from the code and README, not validated demand.

## Principles (do not trade these away casually)

From [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md#2-principles):

- **Offline first.** Only gallery browsing touches the network.
- **No accounts, telemetry or analytics.**
- **Light.** Tauri, not Electron. Adding a dependency needs a conversation.
- **Beautiful by default.** Every template looks finished before the user adds
  anything.
- **RTL is first class.**
- **Accessible.** Keyboard reachable, labelled, at least 4.5:1 contrast.

## Non-goals

- Accounts, cloud sync or a hosted backend. Projects are plain folders, so users
  can sync them with whatever they already use.
- A general design tool. If a feature is not about store listings, it does not
  belong here.
- Bundled AI or anything else that requires a server. Optional bring-your-own-key
  features are an open question ([MEMORY.md](MEMORY.md#open-questions)).

## Status (checked 2026-11-06)

| | |
|---|---|
| Version | `0.1.0` in the files; this fork has published no release yet |
| Traction | None yet for this fork. The original had 52 stars, 6 forks and 55 release asset downloads at the fork date |
| Maintainers | Rang0x |
| CI | Passing on the original's `main` at the fork point; not yet confirmed on this fork |
| Size | 548 tracked files |
| Source lines, tests included | `src/ui` 37.7k, `src/engine` 13.9k, `src/model` 6.4k, `src-tauri` 4.7k |
| Tests | 132 unit test files, 21 end-to-end specs, about 109 Rust tests |
| Content | 20 built-in templates, 4 gallery templates, 51 slide languages, 2 interface languages |

These numbers go stale quickly, so re-check them before relying on them.

## Stack

Tauri 2 (Rust), React 19, TypeScript 7 (strict), Vite 8, Tailwind 4, zustand with
immer, Radix UI. Rendering is plain Canvas 2D with no rendering library. Tests use
Vitest, Playwright (WebKit) and `cargo test`.

## Working on it

```bash
npm install
npm run dev          # browser UI at http://localhost:1420 (?demo, ?kit)
npm run desktop      # the desktop app
npm run typecheck && npm run lint && npm test && npm run e2e
cd src-tauri && cargo test
```

Check interface changes in both dark and light themes and in Arabic.
[CONTRIBUTING.md](CONTRIBUTING.md) has the full checklist.

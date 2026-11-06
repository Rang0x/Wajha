# Security

## Reporting a problem

Please do not put vulnerability details in a public issue.

GitHub's private vulnerability reporting is not enabled on this repository yet. Until
it is, open an issue that only asks a maintainer for a private contact, without
describing the problem.

## What Wajha defends against

Wajha is offline with no accounts, so its main risk is **untrusted files**. People
open `.wajha` and `.wajha-template` files from others, install gallery templates, and
add fonts and images.

| Input | Defence |
|---|---|
| Project and template files | Hand-written validators drop unknown keys, clamp numbers and limit string lengths |
| Zip archives | Size, inflated size and entry count are capped from the zip directory before inflating |
| Images | Magic bytes and header dimensions are checked before decoding; assets are re-hashed against their names |
| Fonts | Checked by signature and hash on import |
| CSV import and export | Cells a spreadsheet would run as a formula are escaped, and round trips stay exact |
| Object keys | `__proto__`, `toString` and similar are refused as text keys and slots |
| File paths | Validated in Rust: no absolute paths, `..`, backslashes, empty segments or NUL |
| Gallery downloads | Size-limited and validated like any file; previews are shown from downloaded bytes, never from CDN URLs |

Limits are in `src/platform/limits.ts`.

## The app shell

- Library file access is limited to the app's data directory, and export writes to
  folders the user picked.
- The frontend has no filesystem, dialog or opener permissions. It uses Wajha's own
  Rust commands, and Rust tests check that the rest of the core API is refused and
  that remote pages get nothing.
- The Content Security Policy allows scripts only from the app, and network access
  only to `cdn.jsdelivr.net` for the gallery.
- `open_url` accepts `https` only.
- The `self-test` cargo feature is never enabled in shipped builds.

## Supply chain

- GitHub Actions are pinned to commit SHAs, and Dependabot updates them monthly.
- Release builds use no caches.
- npm and Cargo dependencies are updated by hand, and adding a dependency needs
  discussion.

## Known limitations

- **Release builds are not code-signed**, so users cannot verify who built them.
- **Gallery content is not signed.** Anyone with write access to `main` can publish
  templates to all users. Templates are data and pass validation, which limits the
  impact, but the content itself is not authenticated.
- **Parsing still happens.** Image and font decoding rely on the system web view, so
  web view bugs apply. On Linux, the web view's patch level depends on the
  distribution.
- **No routine dependency audit** runs in CI (`npm audit`, `cargo audit`).

These are tracked in [ROADMAP.md](ROADMAP.md).

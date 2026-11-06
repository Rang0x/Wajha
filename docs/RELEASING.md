# Releasing Wajha

Pushing a `v*` tag runs `.github/workflows/release.yml`, which builds installers for
macOS (Apple silicon and Intel), Windows and Linux and attaches them to a **draft**
GitHub release. Nothing is public until someone publishes the draft.

> [!IMPORTANT]
> The workflow does not check that tests pass, so step 2 is the only gate today.

## Steps

1. **Bump the version** to the same value in all three files, then refresh the
   lockfiles:
   - `package.json`
   - `src-tauri/tauri.conf.json`
   - `src-tauri/Cargo.toml`

   ```bash
   npm install --package-lock-only
   (cd src-tauri && cargo check)
   ```

2. **Confirm CI passed** on the commit you will tag, and run what CI does not:

   ```bash
   npm run lint
   bash scripts/desktop-smoke.sh
   ```

3. **Update `CHANGELOG.md`:** move `Unreleased` into a dated section.

4. **Review store rules** if Apple or Google changed anything since the last release
   (`src/model/catalog.ts`).

5. **Check compatibility.** If a project, template or gallery format changed, follow
   the rules in [SCALING.md](SCALING.md#rules-for-format-changes).

6. **Commit, tag and push:**

   ```bash
   git commit -am "Wajha x.y.z"
   git tag vx.y.z
   git push origin main vx.y.z
   ```

7. **Test the draft** on each system you can reach: install it, create a project from
   a template, add a screenshot, add Arabic, export, and quit and reopen. Upgrade over
   the previous version, and confirm existing projects still open.

8. **Publish the draft.** Replace the default release text with the changelog section.

## Versions

Semantic versioning, loosely, before 1.0: bump the minor version for features and
format changes, and the patch version for fixes only. Say clearly in the changelog
when a release writes files older versions cannot open.

## If a release is bad

Do not delete a published release that people have downloaded. Mark it as a
pre-release so it stops being "latest", say why in its notes, and ship a fixed patch
version. Do not move an existing tag.

If a bad **gallery** change caused it, revert on `main`. Every version reads it, and
the CDN cache means the fix may take a while to reach everyone
([SCALING.md](SCALING.md#2-the-gallery)).

## Still missing

- A test gate in `release.yml`.
- Signing and notarization, which the README warns users about.
- A way to tell users about a new version.
- Checksums published with each release.
- A decision on the extra artifacts. Releases also contain `.msi` and `.app.tar.gz`
  files that the README's download table does not list.

Progress is tracked in [ROADMAP.md](../ROADMAP.md).

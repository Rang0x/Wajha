# Changelog

Notable changes to Wajha. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Entries before this file
was added are reconstructed from commit titles.

## Unreleased

### Added

- Full-size preview of template slides (#4).

### Fixed

- Fixes so CI passes on Linux WebKit and Windows, including the Windows Rust tests and
  the Linux text baseline check.

### Documentation

- Added a project brief, project memory, roadmap, scaling and release guides, a
  security policy and this changelog.
- Listed the Apple Watch, Apple TV and Vision Pro frames in the template and
  architecture docs.

## 0.1.0 (2026-11-03)

First public release.

- Editor with templates, device frames, translations, CSV round trip and one-click
  export to store and fastlane layouts.
- Apple Watch, Apple TV, Vision Pro, and landscape phones and tablets.
- Each device frame keeps its own shape on every slide.
- Dropped screenshot folders are sorted by device and language.
- Custom fonts and a brand kit.
- Arabic interface, mirrored right to left, with a language setting.
- Slides mirror for right-to-left languages.
- Per-language font and line spacing.

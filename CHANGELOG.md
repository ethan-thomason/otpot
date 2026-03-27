# Changelog

All notable changes from upstream mushorg/conpot are documented here.

## Unreleased

### Fixed

- HTTP template rendering now correctly substitutes `<condata>` tags inside
  `<TITLE>` and other raw text HTML elements. The original HTMLParser-based
  approach silently failed for tags inside raw text containers, leaking
  internal template syntax into HTTP responses and allowing trivial honeypot
  fingerprinting. Replaced with regex substitution. (Fixes #1)

- Removed `--force` requirement when running with default filesystem
  configuration. Conpot now gracefully falls back to default paths with
  a warning instead of exiting with an error.

- Removed undocumented `-f` flag requirement for binding to non-local
  interfaces. Replaced hard exit with an informational warning.

- Fixed misleading "Can't find temp directory" log message. Now clearly
  indicates that the default path will be used.

### Known Issues

- Python 3.11+: `cpppo` library breaks EtherNet/IP support. (#2)
- HTTP template warning fires once per protocol on startup — cosmetic.

# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- (placeholder for upcoming changes)

## [1.0.0] - 2026-06-05

### Added
- Initial release of `wrsync`, a Bash wrapper around `rsync` for sending files
  and directories to an SSH server using named configuration profiles.
- Named config profiles loaded from
  `${XDG_CONFIG_HOME:-$HOME/.config}/wrsync/<remote>.conf`.
- Safe config parsing with a strict key whitelist (the config file is never
  `source`d), supporting keys: `HOST`, `USER`, `PORT`, `IDENTITY_FILE`,
  `RSYNC_OPTS`, `SSH_OPTS`.
- Sensible rsync defaults: `--archive`, `--compress`, `--human-readable`,
  `--partial`, `--progress`.
- Strict validation of source existence, target format, required keys, port
  range, and SSH key readability.
- `-n`/`--dry-run` mode to preview transfers.
- `-h`/`--help` usage output.
- Colorized log output on interactive terminals only.
- Runtime dependency checks for `rsync` and `ssh`.
- Permission warning when a config file is readable by group/others.
- Example config file (`monvps.conf.example`) and full English README.

[Unreleased]: https://github.com/mokira3d48/wrsync/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/mokira3d48/wrsync/releases/tag/v1.0.0

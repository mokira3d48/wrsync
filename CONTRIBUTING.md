# Contributing to wrsync

Thanks for your interest in improving `wrsync`! This document explains how to
report issues, propose changes, and the conventions the project follows.

## Table of contents

- [Code of conduct](#code-of-conduct)
- [Reporting bugs](#reporting-bugs)
- [Requesting features](#requesting-features)
- [Development setup](#development-setup)
- [Coding guidelines](#coding-guidelines)
- [Testing your changes](#testing-your-changes)
- [Commit messages](#commit-messages)
- [Pull request process](#pull-request-process)
- [License](#license)

## Code of conduct

Be respectful and constructive. Assume good faith, keep discussions focused on
the technical merits, and help newcomers feel welcome.

## Reporting bugs

Before opening an issue, please:

1. Search existing issues to avoid duplicates.
2. Confirm you are running a supported environment (Bash 4.0+, `rsync`, `ssh`).

When filing a bug, include:

- The exact command you ran (redact any secrets).
- The full output, ideally with `--dry-run` added.
- Your OS, Bash version (`bash --version`), and `rsync --version`.
- A redacted copy of the relevant `*.conf` profile (remove hosts/keys).

## Requesting features

Open an issue describing the use case and the problem it solves, not just the
proposed solution. Small, focused enhancements that stay true to the tool's
purpose (a thin, safe rsync wrapper) are most likely to be accepted.

## Development setup

```bash
git clone https://github.com/mokira3d48/wrsync.git
cd wrsync
chmod +x wrsync

# Run against a throwaway profile while developing:
mkdir -p ~/.config/wrsync
cp monvps.conf.example ~/.config/wrsync/dev.conf
$EDITOR ~/.config/wrsync/dev.conf

./wrsync --dry-run ./README.md dev:~/tmp
```

## Coding guidelines

This project targets **production-grade Bash**. Please keep the existing style:

- Start scripts with `#!/usr/bin/env bash` and
  `set -o errexit -o nounset -o pipefail`.
- Quote all expansions (`"$var"`, `"${array[@]}"`).
- Build external commands with arrays, not string concatenation.
- Never `source` untrusted input; keep the config parser's strict key whitelist.
- Use `local` for function variables; prefer small, single-purpose functions.
- Write user-facing messages and code comments in **English**.
- Send errors/warnings to `stderr` (use the existing `err`/`warn`/`info`/`die`
  helpers).
- Prefer Bash builtins over external processes where reasonable.

### ShellCheck

All code must pass [ShellCheck](https://www.shellcheck.net/) with no warnings:

```bash
shellcheck wrsync
```

If you must suppress a check, add a narrowly-scoped `# shellcheck disable=SCxxxx`
comment with a short justification, as done in the existing code.

## Testing your changes

There is no heavy test harness; please verify manually before submitting:

```bash
bash -n wrsync          # syntax check
shellcheck wrsync       # lint
./wrsync --help         # usage output
./wrsync --dry-run ./some_dir dev:~/tmp   # end-to-end dry run
```

Test both the happy path and error paths (missing config, bad port, missing
source, malformed target).

## Commit messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) style:

```
<type>(optional scope): <short summary>

<optional body explaining what and why>
```

Common types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`.

Examples:

```
feat: add --delete flag to mirror the source
fix: reject target without a colon separator
docs: clarify remote path resolution in README
```

## Pull request process

1. Fork the repo and create a topic branch (`feat/my-change`).
2. Keep the change focused; one logical change per PR.
3. Update the docs (`README.md`) and `CHANGELOG.md` under `[Unreleased]` when
   behavior changes.
4. Ensure `bash -n` and `shellcheck` pass.
5. Open the PR with a clear description and link any related issue.

Maintainers may request changes; please be patient and responsive.

## License

By contributing, you agree that your contributions will be licensed under the
project's [MIT License](LICENSE).

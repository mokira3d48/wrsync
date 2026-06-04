```text
                                     ____
 _      ______________  ______  _____\   \
| | /| / / ___/ ___/ / / / __ \/ ___/_   _|   >_ rsync over SSH,
| |/ |/ / /  (__  ) /_/ / / / / /__  | |          minus the boilerplate.
|__/|__/_/  /____/\__, /_/ /_/\___/  |_|
                 /____/                            ~$ wrsync ./code box:~/app
```

<div align="center">

# `wrsync`

**Push files & folders to any SSH box using named config profiles.**
_No more typing host, port and key paths by hand. Set it once, ship forever._

[![Shell](https://img.shields.io/badge/built%20with-Bash-1f425f?logo=gnubash&logoColor=white)](https://www.gnu.org/software/bash/)
[![ShellCheck](https://img.shields.io/badge/lint-ShellCheck-brightgreen?logo=gnu&logoColor=white)](https://www.shellcheck.net/)
[![Powered by rsync](https://img.shields.io/badge/powered%20by-rsync-blue)](https://rsync.samba.org/)
[![SSH](https://img.shields.io/badge/transport-OpenSSH-black?logo=openssh&logoColor=white)](https://www.openssh.com/)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)
[![Made with ♥](https://img.shields.io/badge/made%20with-%E2%99%A5%20%26%20%24%28%29-red)](#)

</div>


A small, production-grade Bash wrapper around [`rsync`](https://rsync.samba.org/)
that lets you push files and folders to an SSH server using **named
configuration profiles**.

Instead of typing long `rsync`/`ssh` command lines with hosts, ports and key
paths every time, you store each server's settings once in a config file and
refer to it by name:

```bash
wrsync ./my_folder_or_file myvps:~/my_project_dir
```

---

## Table of contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
  - [Config file location](#config-file-location)
  - [Supported keys](#supported-keys)
  - [Example config](#example-config)
- [Usage](#usage)
  - [Synopsis](#synopsis)
  - [Options](#options)
  - [How the destination is resolved](#how-the-destination-is-resolved)
  - [Examples](#examples)
- [How it works](#how-it-works)
- [Security notes](#security-notes)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Features

- **Named profiles** — one config file per server, referenced by name.
- **Safe config parsing** — the config file is read line by line with a strict
  key whitelist. It is **never** `source`d, so it cannot execute arbitrary code.
- **Sensible rsync defaults** — archive mode (`-a`), compression, resumable
  transfers (`--partial`), and progress reporting.
- **Strict validation** — checks that the source exists, the target format is
  correct, required keys are present, the port is a valid number, and the SSH
  key is readable.
- **Dry-run mode** — preview what would be transferred without copying anything.
- **Helpful, colorized output** (only when writing to an interactive terminal).
- Follows the [XDG Base Directory](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)
  spec for config location.

---

## Requirements

- `bash` 4.0 or newer
- `rsync`
- `ssh` (OpenSSH client)

On Debian/Ubuntu/Kali:

```bash
sudo apt-get install rsync openssh-client
```

`wrsync` checks for these dependencies at runtime and exits with a clear error
if one is missing.

---

## Installation

1. Make the script executable and place it on your `PATH`:

   ```bash
   chmod +x wrsync
   sudo install -m 755 wrsync /usr/local/bin/wrsync
   ```

   Prefer a per-user install (no `sudo`)? Use a directory that is on your
   `PATH`, for example `~/.local/bin`:

   ```bash
   install -m 755 wrsync ~/.local/bin/wrsync
   ```

2. Verify it is reachable:

   ```bash
   wrsync --help
   ```

---

## Configuration

### Config file location

`wrsync myvps:...` looks for a profile named `myvps` at:

```
${XDG_CONFIG_HOME:-$HOME/.config}/wrsync/myvps.conf
```

In practice, on most systems that is:

```
~/.config/wrsync/myvps.conf
```

Create the directory and your first profile:

```bash
mkdir -p ~/.config/wrsync
cp monvps.conf.example ~/.config/wrsync/myvps.conf
chmod 600 ~/.config/wrsync/myvps.conf   # recommended: keep it private
```

Then edit `~/.config/wrsync/myvps.conf` with your server details.

### Supported keys

The config file uses a simple `KEY=value` format. Lines starting with `#` and
blank lines are ignored. Inline comments (`KEY=value  # comment`) are supported.

| Key             | Required | Default | Description                                              |
|-----------------|----------|---------|----------------------------------------------------------|
| `HOST`          | **Yes**  | —       | IP address or hostname of the remote machine.            |
| `USER`          | **Yes**  | —       | SSH username.                                            |
| `PORT`          | No       | `22`    | SSH port (must be `1`–`65535`).                          |
| `IDENTITY_FILE` | No       | —       | Path to the SSH private key. `~` is expanded to `$HOME`. |
| `RSYNC_OPTS`    | No       | —       | Extra options passed verbatim to `rsync`.                |
| `SSH_OPTS`      | No       | —       | Extra options passed verbatim to `ssh`.                  |

Unknown keys are ignored with a warning.

### Example config

`~/.config/wrsync/myvps.conf`:

```ini
# --- Required ---
HOST=192.168.1.42          # IP or hostname of the remote machine
USER=arnold                # SSH username

# --- Optional ---
PORT=22                    # SSH port (default: 22)
IDENTITY_FILE=~/.ssh/id_ed25519   # SSH private key to use

# Extra options forwarded to rsync (e.g. exclude paths).
RSYNC_OPTS=--exclude=.git --exclude=node_modules

# Extra options forwarded to ssh.
SSH_OPTS=-o StrictHostKeyChecking=accept-new
```

---

## Usage

### Synopsis

```text
wrsync <source> <remote>:<remote_path>
```

- `<source>` — a local file or directory to send.
- `<remote>` — the name of a config profile in `~/.config/wrsync/`.
- `<remote_path>` — the destination path on the remote machine.

### Options

| Option            | Description                                       |
|-------------------|---------------------------------------------------|
| `-n`, `--dry-run` | Simulate the transfer without copying anything.   |
| `-h`, `--help`    | Show usage help and exit.                          |

### How the destination is resolved

The `<remote>:<remote_path>` argument is split on the first colon:

- The part **before** the colon is the **profile name**. `wrsync` loads
  `~/.config/wrsync/<remote>.conf` and reads `HOST`, `USER`, `PORT`, etc.
- The part **after** the colon is the **path on the remote machine**, passed
  straight through to `rsync`. It is interpreted by the remote shell:
  - `~/my_project_dir` resolves to `/home/<USER>/my_project_dir`.
  - `/opt/my_project_dir` is an absolute system path.

When `<source>` is a directory, a trailing slash is removed so that the
directory **itself** (not just its contents) is copied into the destination.

### Examples

Send a folder into your remote home directory:

```bash
wrsync ./my_folder_or_file myvps:~/my_project_dir
```

> `my_folder_or_file` is copied to `/home/<USER>/my_project_dir/` on `myvps`.

Send the same item into a system directory:

```bash
wrsync ./my_folder_or_file myvps:/opt/my_project_dir
```

Preview a transfer without copying (dry run):

```bash
wrsync --dry-run ./my_folder myvps:~/backup
```

Send a single file:

```bash
wrsync ./report.pdf myvps:~/documents
```

---

## How it works

Under the hood, `wrsync` builds and runs a command equivalent to:

```bash
rsync --archive --compress --human-readable --partial --progress \
      -e "ssh -p <PORT> -i <IDENTITY_FILE> <SSH_OPTS>" \
      -- <source> <USER>@<HOST>:<remote_path>
```

The default `rsync` flags are:

| Flag               | Purpose                                                        |
|--------------------|----------------------------------------------------------------|
| `--archive` (`-a`) | Recursive **and** preserves permissions, timestamps, symlinks. |
| `--compress` (`-z`)| Compresses data during transfer.                               |
| `--human-readable` | Prints sizes in a readable format.                             |
| `--partial`        | Keeps partially transferred files so transfers can resume.     |
| `--progress`       | Shows live progress per file.                                  |

> **Note about `-r`:** you don't need to pass `-r` for directories. `--archive`
> already implies recursion (`-r`) plus metadata preservation, which is the
> recommended default for transfers.

The SSH transport (host, port, key, extra options) is assembled from your
config profile and passed to `rsync` via the `-e` option. Both the rsync and
ssh command lines are built using Bash arrays, so paths and options are quoted
safely.

---

## Security notes

- The config file may contain sensitive details. Keep it private:

  ```bash
  chmod 600 ~/.config/wrsync/myvps.conf
  ```

  `wrsync` warns you if the file is readable by group or others.

- The config is parsed with a **strict key whitelist** and is never executed as
  a shell script, so a malicious or malformed config cannot run arbitrary code.

- Prefer key-based SSH authentication (`IDENTITY_FILE`) over passwords.

- `StrictHostKeyChecking=accept-new` (used in the example) trusts a host's key
  on first contact. For higher security, pre-populate `~/.ssh/known_hosts` and
  drop that option.

---

## Troubleshooting

| Message                                          | Cause / fix                                                              |
|--------------------------------------------------|--------------------------------------------------------------------------|
| `Missing dependency: 'rsync'…`                   | Install `rsync` / `openssh-client`.                                      |
| `Config file not found: …`                       | The profile `<remote>.conf` does not exist in `~/.config/wrsync/`.       |
| `'HOST' is missing` / `'USER' is missing`        | Add the required keys to your config file.                               |
| `Invalid PORT…`                                  | `PORT` must be an integer between 1 and 65535.                           |
| `SSH key not found or not readable`              | Check the `IDENTITY_FILE` path and its read permissions.                 |
| `Invalid target … expected format`               | The target must be `<remote>:<remote_path>` (a colon is required).       |
| `Source not found`                               | The local `<source>` path does not exist.                               |

For more detail on any transfer, run with `--dry-run` first to see exactly what
`rsync` would do.

---

## License

Use, modify and distribute freely. No warranty.

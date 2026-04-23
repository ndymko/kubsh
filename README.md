# KubSH

KubSH is a small Linux shell written in C with an additional FUSE-based virtual filesystem.
The shell combines a readline-powered REPL, a few built-in commands, external command
execution through `execvp`, command history persistence, and a virtual view of system users.

## Features

- Interactive shell prompt powered by `readline`
- Persistent command history stored in `~/.kubsh_history`
- Built-in commands:
  - `\q` to exit
  - `echo ...`
  - `cd <path>`
  - `\e $VAR` to print a colon-separated environment variable line by line
  - `\l <disk>` to inspect partition information for `/dev/<disk>`
- External command execution with `execvp`
- Tilde expansion for paths
- `SIGHUP` handler that keeps the shell alive and prints a reload message
- FUSE virtual filesystem that exposes login-capable users as directories

## Virtual Filesystem

KubSH starts a FUSE filesystem in a background thread on launch.

By default, the mountpoint is:

```text
$HOME/users
```

You can override it with:

```bash
export KUBSH_MOUNTPOINT=/opt/users
```

The mounted filesystem exposes:

- one directory per login-capable user
- `id` file with the user's UID
- `home` file with the user's home directory
- `shell` file with the user's login shell

It also supports:

- `mkdir <mountpoint>/<username>` to create a system user
- `rmdir <mountpoint>/<username>` to remove a system user and home directory

Important: this VFS is not read-only. Creating or removing directories in the mountpoint
can create or delete real system users through `adduser` and `userdel`.

## Requirements

- Linux
- `cmake`
- C compiler with C11 support
- `make`
- `libreadline`
- `libfuse3`
- `dpkg-deb` for packaging

For Debian or Ubuntu:

```bash
sudo apt update
sudo apt install -y cmake build-essential dpkg-dev libfuse3-dev libreadline-dev
```

## Build

Build the executable:

```bash
make build
```

Build and run:

```bash
make run
```

Build a Debian package:

```bash
make package
```

Clean generated artifacts:

```bash
make clean
```

## Project Structure

```text
src/main.c                      Entry point and main REPL loop
src/command_reader.c            Readline-based command input
src/command_parser.c            Simple space-delimited parser
src/command_executor.c          External command execution
src/external_command_executor.c Built-in shell commands
src/history.c                   Persistent command history
src/vfs.c                       FUSE filesystem operations
src/vfs_manager.c               Mountpoint setup and FUSE thread startup
src/disk_parser.c               MBR and GPT partition inspection
```

## Notes and Limitations

- Command parsing is intentionally simple and splits only by spaces.
- The VFS relies on system user information from `/etc/passwd` and `/etc/shells`.
- Some VFS actions may require elevated privileges, depending on how KubSH is started.
- The project currently contains tracked binary artifacts such as `kubsh.deb`.

## Example Session

```text
KubSH> echo hello
hello

KubSH> \e $PATH
/usr/local/bin
/usr/bin
/bin

KubSH> \l sda
...

KubSH> cd /tmp
KubSH> ls
...

KubSH> \q
```

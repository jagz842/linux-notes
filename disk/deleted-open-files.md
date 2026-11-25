# Finding deleted open files with `lsof`: the disk was full, but the big file was gone

Follow a system administrator using `lsof` to find a deleted log file that is still consuming disk space. The names, sizes, and command output below are examples.

## The alert

It was a busy Monday morning when Sara, a system administrator, received an alert: the root filesystem on an application server was full. Soon afterward, the application started reporting errors when it tried to write new data.

Sara checked the server:

```bash
df -h /
```

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       100G   96G     0 100% /
```

A teammate mentioned that an unusually large application log had been deleted a few minutes earlier. Sara expected the deletion to have recovered space, but the filesystem was still full.

## The numbers don't add up

Sara checked how much space the visible files on the root filesystem were using:

```bash
sudo du -xsh /
```

```text
40G    /
```

That was much less than the usage reported by `df`. She also checked the inodes:

```bash
df -i /
```

Inode usage was healthy, so the immediate problem was not inode exhaustion. The mismatch was a clue, though not proof of a deleted file: hidden files beneath mount points, filesystem metadata, reserved blocks, snapshots, and inaccessible directories can also contribute to differences.

Sara had used `sudo` so `du` could inspect protected directories, and `-x` to keep it on the same filesystem. Without those options, the comparison could be misleading.

## The missing log

Sara suspected that the deleted log was still open. She asked `lsof` to list open files with fewer than one link:

```bash
sudo lsof +L1
```

Among the results she found an entry like this:

```text
COMMAND   PID USER   FD   TYPE DEVICE    SIZE/OFF NLINK NODE NAME
myapp    2481 app    4w   REG    8,1 60129542144     0 1234 /var/log/myapp/app.log (deleted)
```

There was the missing log: roughly 56 GiB, marked `(deleted)`, and still open by process `2481`. The `4w` in the `FD` column meant file descriptor **4**, open for writing.

| Field | Example | What it tells Sara |
| --- | --- | --- |
| `COMMAND` | `myapp` | The process holding the file open. |
| `PID` | `2481` | The process ID. |
| `FD` | `4w` | Descriptor 4 is open for writing. |
| `SIZE/OFF` | `60129542144` | The logical file size in bytes. |
| `NLINK` | `0` | The file has no remaining directory links. |
| `NAME` | `app.log (deleted)` | The file was deleted but remains open. |

The size shown by `lsof` is the file's logical size, which can differ from its allocated disk space. Multiple entries may also refer to the same file, so their sizes should not automatically be added together.

## Why deleting it hadn't helped

When Sara's teammate removed the log, `rm` removed its directory entry. The application, however, still had the file open. The kernel therefore kept the file and its allocated blocks available to the process.

`du` could no longer find the file by walking the directory tree, but `df` still counted its blocks as used. The space would become available after the remaining references to the unlinked file were released.

## Getting the space back

Sara checked the application's documentation and confirmed that restarting the service would close its log. After coordinating a short interruption, she ran:

```bash
sudo systemctl restart myapp.service
df -h /
```

The application closed its old descriptor, and the free space returned. If another process had still held the file open, Sara would have needed to investigate that process too. Running `sudo lsof +L1` again is a useful check when space does not return.

### If a restart must wait

For an identified regular log file, truncating it through the open descriptor can be an emergency measure:

```bash
sudo truncate -s 0 /proc/<PID>/fd/<FD-number>
```

Verify the process and descriptor immediately before using the command. Truncation permanently discards the file's contents and can disrupt the application. Continued writes may grow it again; a process that does not use append mode may write at its old offset and create a sparse file. Do not use this on databases or other files without understanding the application's behavior. Check `df` afterward.

## Preventing a repeat

After the incident, Sara configured log rotation using the application's documented reopen mechanism. With `logrotate`, a `postrotate` script can signal an application that supports reopening its logs; the correct signal depends on the application.

Another option is `copytruncate`, which copies the log and truncates the original while the application keeps it open. It can lose entries written between the copy and truncate steps, so a supported reopen mechanism is preferable when available.

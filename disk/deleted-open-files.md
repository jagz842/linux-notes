# Disk space troubleshooting: deleted open files

## Symptom

`df` reports filesystem usage, while `du` totals allocated space for files it can see in a directory tree. If `df` shows a full filesystem but `du` reports much less usage, deleted files still held open by processes are one possible cause.

Compare the same filesystem, replacing `/` with its mount point:

```bash
df -h /
df -i /
sudo du -xsh /
```

`-x` keeps `du` on one filesystem, and `sudo` lets it inspect directories inaccessible to an ordinary user. Healthy inode usage rules out inode exhaustion at that moment, but does not establish the cause of a space mismatch. Other causes include files hidden beneath mount points, filesystem metadata, reserved blocks, snapshots, and inaccessible directories.

## Why it happens

Removing a file with `rm` unlinks a directory entry. If its last link is removed while a process still has the file open, the kernel retains the file and its allocated blocks until the remaining references are released. `du` cannot find the unlinked file by walking the directory tree, but `df` still accounts for its blocks.

## Find the files

```bash
sudo lsof +L1
```

Look for a large file on the affected filesystem, and note its process ID (`PID`), file descriptor (`FD`), and `SIZE/OFF`. For example, an FD shown as `4w` is descriptor **4**; `w` indicates write access. The displayed size is the logical size and may differ from the allocated blocks. Multiple entries can refer to the same file, so do not automatically add their sizes together.

## Reclaim the space

Prefer the application's documented log-reopen mechanism or a service restart, after considering the impact on users:

```bash
sudo systemctl restart <service-name>
df -h /
```

Space is reclaimed once all references to the unlinked file are released; another process may still hold it open. Check `lsof +L1` again if space does not return.

If a restart or reopen is not immediately possible, truncating an identified regular log file through its descriptor can be an emergency measure:

```bash
sudo truncate -s 0 /proc/<PID>/fd/<FD-number>
```

Verify the process and descriptor immediately before running this command. It permanently discards the file's contents and can disrupt the application; continued writes may also grow it again. A process that does not use append mode may continue writing at its old offset, creating a sparse file. Do not use this on databases or other files without understanding the application's behavior. Recheck `df` afterward.

## Prevention

Use the application's supported log rotation or configure `logrotate`. A `postrotate` script can ask an application that supports it to close and reopen its log after rotation; the appropriate signal depends on the application. `copytruncate` retains the same open file, but log entries written between the copy and truncate steps can be lost. Prefer a supported reopen mechanism when available.

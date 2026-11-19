# Maintaining Linux notes

[Home](README.md) · [Topics](docs/topics.md) · [Reading queue](LATEST.md)

## Add a note

1. Choose the closest folder in the [topic index](docs/topics.md) or capture an unfinished idea in the [inbox](inbox/README.md).
2. Keep one concept, task, or troubleshooting case per note. Record the Linux distribution and version, the commands or configuration, expected results, what you tested, and sources.
3. Link the note from its topic and add it to [LATEST.md](LATEST.md). The helper does both for you.
4. After reviewing a note, tick its checkbox in the reading queue and change its status to `reviewed` if it has one. Keep reviewed entries as a learning history.

From the repository root, run with Python 3.9 or later:

```bash
python3 scripts/new_note.py systemd "Systemd timers"
python3 scripts/new_note.py networking/ip-suite "Policy routing"
python3 scripts/new_note.py inbox "Read about cgroups"
```

The helper uses today's local date and refuses to overwrite an existing note. Destination folders listed in the topic index support the helper. A nested path files the note there and links it from the top-level topic index.

## Add or update a note manually

Use the [note template](templates/note.md) for a concept or the [lab template](templates/lab.md) for a directory with runnable examples. Add a relative link to the topic index and insert an entry immediately below `<!-- entries -->` in `LATEST.md`:

```markdown
- [ ] YYYY-MM-DD — [Note title](topic/note-title.md) — One sentence about what you added.
```

For a substantial revision, prefix the link with **Updated** and use the revision date. Spelling fixes and file moves do not need a queue entry.

## Keep the collection navigable

- File new notes with lowercase, hyphenated names. Keep existing paths stable so older links and bookmarks still work.
- Add a topic to [docs/topics.md](docs/topics.md) when it needs a starting page. To enable the helper, give its top-level folder a `README.md` with a `<!-- new-notes -->` marker.
- Keep lab instructions and sample files together. Link screenshots from a nearby `assets/` folder.
- Use placeholders for credentials and machine-specific values. Check paths, permissions, and commands that change system state in a disposable VM before recording a procedure as verified.
- Before committing, run `git diff --check`, open the new links, and check the reading queue.

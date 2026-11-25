# Linux System Administration Notes

Study Linux fundamentals, look up commands, and review troubleshooting procedures for common administration tasks.

[Latest learning and read later](docs/LATEST.md) · [Browse by topic](docs/topics.md) · [Learning inbox](notes/inbox/README.md) · [How to add notes](docs/CONTRIBUTING.md)

## Daily learning

1. File a new note under its topic or capture an unfinished idea in the inbox.
2. Add a dated entry to the [reading queue](docs/LATEST.md) for each new note or substantial update.
3. Tick the checkbox after reviewing an entry. Keep completed entries as a record of what you learned.

From the repository root, the Python 3.9+ helper creates a note and adds it to the reading queue. For a topic note, it also links the note from the topic index.

```bash
python3 scripts/new_note.py systemd "Systemd timers"
python3 scripts/new_note.py inbox "Read about cgroups"
```

## Study and reference

- [Follow a learning path](docs/reference.md#-learning-path-recommendations) for beginner through advanced material, or browse the [Red Hat certification tracks](docs/reference.md#-red-hat-certification-tracks).
- [Search the detailed reference](docs/reference.md) for command guides, administration procedures, and troubleshooting notes.

## Repository layout

| Directory | Contents |
| --- | --- |
| [`notes/`](notes/) | Study notes grouped by Linux topic |
| [`labs/`](labs/README.md) | Projects with example code, scripts, and related notes |
| [`docs/`](docs/) | Topic map, detailed reference, reading queue, and templates |
| [`scripts/`](scripts/) | Repository tools, including the note helper |

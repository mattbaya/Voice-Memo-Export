# AGENTS.md

Notes for AI coding agents working in this repository.

## What this is

A single Bash script, `export-voice-memos`, that converts Apple Voice Memos recordings to MP3 using `sqlite3` and `ffmpeg`. There is no build step and no test suite.

## How it works

1. Recordings and the index DB live in `~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/` (overridable with `VM_DIR`).
2. The script snapshots `CloudRecordings.db` with `sqlite3 .backup` to avoid lock contention with the Voice Memos app.
3. It queries `ZCLOUDRECORDING`:
   - `ZPATH` is the audio file path, usually relative to `VM_DIR`. It can be `.m4a` or `.qta`.
   - The title is `ZENCRYPTEDTITLE` (plain text despite the name) with `ZCUSTOMLABEL` as fallback.
   - `ZDATE` is a Core Data timestamp (seconds since 2001-01-01). Add `978307200` to get Unix time.
   - A non-null `ZEVICTIONDATE` means the memo is in Recently Deleted.
4. Column availability varies by macOS version. The script checks `pragma_table_info` before using optional columns. Keep that pattern when adding new ones.
5. Each recording is transcoded with `libmp3lame` VBR, tagged, and `touch`ed to the recording date. Existing outputs are skipped (idempotent).

## Memos missing or stored only in iCloud

The script can't download from iCloud. It only reports memos whose `ZPATH` file is absent as `MISSING`. When a user says memos are missing:

1. **Check what the Mac knows about.** The count next to "All Recordings" in the Voice Memos sidebar is the whole set this Mac has synced. If it's lower than the user expects, the memos aren't syncing: point the user to iCloud → Saved to iCloud → Voice Memos on both the iPhone and the Mac. Don't change these settings yourself; they belong to the user.
2. **Force downloads by selecting memos.** With the user's approval to control Voice Memos through computer use, click the first row in the list, then send ↓ one row at a time. Give each memo a moment to load its waveform, since keys pressed too quickly get dropped. Scroll to the bottom to confirm you've reached the end of the list. Voice Memos isn't AppleScript-scriptable, so this can't be done with `osascript`.
3. **Ask the user to re-run the script.** It's idempotent and exports only the newly downloaded memos.

## Constraints and conventions

- **TCC / Full Disk Access:** agents usually can't read the Voice Memos container from a sandboxed or unprivileged session. Don't try to bypass or modify macOS privacy settings. Ask the user to run the script from a terminal that has Full Disk Access and share the output.
- **Never commit audio.** `.gitignore` excludes `*.mp3`, `*.m4a` and `*.qta`. Recordings are personal data.
- Configuration uses env vars with in-script defaults: `VAR="${OVERRIDE:-default}"`.
- Keep the script portable to macOS's stock Bash 3.2 and BSD userland (`date -r`, BSD `touch -t`). Avoid GNU-only flags.
- Keep `set -euo pipefail`. Run `bash -n export-voice-memos` (and `shellcheck` if available) after editing.
- Update `README.md` when behavior or options change.

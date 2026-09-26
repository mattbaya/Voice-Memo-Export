# Voice Memo Export

Export every recording from Apple's **Voice Memos** app on macOS to MP3 files, keeping the titles and recording dates.

## Requirements

- macOS with Voice Memos
- [ffmpeg](https://ffmpeg.org/) with `libmp3lame` (`brew install ffmpeg`)
- `sqlite3` (ships with macOS)
- **Full Disk Access** for the terminal app that runs the script (System Settings → Privacy & Security → Full Disk Access). Voice Memos data lives in a protected group container, so without this the script can't read it.

## Usage

```bash
./export-voice-memos              # export into the script's own directory
./export-voice-memos ~/Music/Memos  # export into a different directory
```

Files are named `YYYY-MM-DD HHMMSS - <Title>.mp3`.

### Options (environment variables)

| Variable | Default | Meaning |
| --- | --- | --- |
| `VM_DIR` | `~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings` | Where Voice Memos stores recordings |
| `MP3_QUALITY` | `2` | LAME VBR quality passed to `ffmpeg -q:a` (0 = best, 9 = smallest) |

Example: `MP3_QUALITY=0 ./export-voice-memos`

## Behavior

- Reads titles and dates from a copy of `CloudRecordings.db`, so it doesn't interfere with the running app.
- Sets ID3 `title`, `date` and `album` ("Voice Memos") tags, and sets each file's modification time to the recording date.
- Skips memos in **Recently Deleted**.
- Skips files that already exist, so re-running only exports new memos.
- Reports memos that exist only in iCloud and haven't been downloaded yet. Play them once in Voice Memos, then re-run.

## Privacy

`.gitignore` excludes `*.mp3`, `*.m4a` and `*.qta` so exported recordings aren't committed by accident.

## License

MIT

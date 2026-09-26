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

## Memos missing or stored only in iCloud

The script only exports audio that's already on this Mac. It can't download anything from iCloud.

1. **Check that the Mac has every memo.** Compare the count next to "All Recordings" in the Voice Memos sidebar with what's on your iPhone. If the Mac shows fewer, it isn't syncing them. Make sure **Voice Memos** is turned on under iCloud → Saved to iCloud, on both the iPhone (Settings → [your name] → iCloud) and the Mac (System Settings → [your name] → iCloud), with the same Apple Account. Then leave both apps open on Wi-Fi until the lists match.
2. **Download any memos that are only in iCloud.** The script lists these as `MISSING`. Voice Memos has no "download all" option, but selecting a memo makes the app fetch its audio. Click the first recording, then press ↓ to step through the list, pausing briefly on each one until its waveform appears.
3. **Run the script again.** Files already exported are skipped, so only the new memos get converted.

If the older memos live only on an iPhone that you'd rather not sync, export them there instead: select them, then Share → Save to Files.

## Privacy

`.gitignore` excludes `*.mp3`, `*.m4a` and `*.qta` so exported recordings aren't committed by accident.

## License

MIT

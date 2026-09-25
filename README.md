# Nautilus Media Columns

Adds media metadata columns to Nautilus (GNOME Files) list view for **images, videos, and audio files**.

Supported metadata includes **Dimensions, Duration, FPS (Framerate), Title, Artist, Album, Album Artist, Track, and Genre**. Metadata is read using GNOME-native libraries, GStreamer, and Mutagen, and cached for performance.

<p align="center">
    <img src="assets/media-columns.png" width="80%" alt="Media Columns Example">
</p>

***

## Features

- **Dimensions**
  - Images: fast header-only probe (no full decode)
  - Videos: via GStreamer Discoverer
- **Duration / Length**
  - Videos: via GStreamer
  - Audio: via Mutagen
- **FPS (Framerate)**
  - Videos only
  - Rounded to whole numbers
- **Audio metadata**
  - Title
  - Artist
  - Album
  - Album Artist
  - Track number
  - Genre
- **Persistent cache**
  - SQLite (WAL mode)
  - Indexed by file path
  - Automatically invalidated when file mtime or size changes
  - Automatic pruning:
    - Time-based TTL: 90 days since last access
    - Size-based cap: 50,000 rows
    - Least-recently-used entries are removed first
- **No ffmpeg dependency**
  - Video metadata uses GNOME-provided GStreamer
  - Audio metadata uses Mutagen
- **Image metadata**
  - Prefers GExiv2
  - Falls back to GdkPixbuf if GExiv2 is unavailable

---

## Requirements

- Python 3
- Nautilus with **nautilus-python (API 4.0)**
- GStreamer
- GstPbutils GI typelib
- **Mutagen**
- GExiv2 GI bindings (recommended for richer image metadata)

### Ubuntu / Debian

Install the required packages:

```bash
sudo apt update
sudo apt install \
  python3-nautilus \
  python3-mutagen \
  gir1.2-gst-plugins-base-1.0
```

GStreamer plugins may also be required for additional video codec support.

For richer image metadata, install the GExiv2 GI bindings if available for your distribution.

### Arch / CachyOS

The required components include:

- `nautilus`
- `nautilus-python`
- `python-mutagen`
- GStreamer
- GStreamer plugin packages
- GExiv2 GI bindings (optional)

Install Mutagen with:

```bash
sudo pacman -S python-mutagen
```

---

## Installation

### Per-user installation

Create the Nautilus Python extension directory:

```bash
mkdir -p ~/.local/share/nautilus-python/extensions
```

Download the extension:

```bash
wget https://raw.githubusercontent.com/angelside/nautilus-media-columns-extension/main/nautilus-media-columns.py \
    -O ~/.local/share/nautilus-python/extensions/nautilus-media-columns.py
```

Restart Nautilus:

```bash
nautilus -q
```

Then:

1. Open Nautilus.
2. Switch to **List View**.
3. Right-click the column header.
4. Open **Visible Columns**.
5. Enable the columns you want.

Available columns include:

- Dimensions
- Duration
- FPS
- Title
- Artist
- Album
- Album Artist
- Track
- Genre

> **Note:** If Mutagen is not installed, the extension may fail to load and the media columns will not appear.

---

## Cache

Cache database location:

```bash
~/.cache/nautilus-media-columns/media.sqlite3
```

The cache is safe to delete at any time. It will be recreated automatically.

### Cache behavior

- Entries are refreshed when file **mtime or size** changes.
- Entries are automatically pruned after 90 days without access.
- The cache is limited to 50,000 rows.
- Least-recently-used entries are removed when the size limit is reached.
- Database writes are batched to minimize disk I/O.
- SQLite uses WAL mode.

---

## Supported Formats

### Images

- PNG
- JPG / JPEG
- WebP
- BMP
- TIFF

### Videos

- MP4
- MKV
- MOV
- AVI
- WebM
- M4V

### Audio

- MP3
- M4A
- WAV
- OGG
- FLAC

> Video codec support depends on the installed GStreamer plugins. Audio metadata support is provided by Mutagen.

---

## Audio Metadata

Audio files can expose the following metadata columns:

| Column | Description |
|---|---|
| **Title** | Track title |
| **Artist** | Track artist |
| **Album** | Album name |
| **Album Artist** | Album artist |
| **Track** | Track number |
| **Genre** | Genre |
| **Duration / Length** | Track duration |

Metadata availability depends on the tags stored in the audio file.

For example, an MP3 without an embedded artist tag will have an empty **Artist** column.

Audio duration is obtained from the file's audio information and does not require GStreamer.

---

## Performance Notes

- The first visit to a folder may probe uncached files.
- Subsequent visits use the SQLite cache and should be significantly faster.
- Metadata probing is performed on a best-effort basis.
- Large folders containing many uncached media files may take longer during the first scan.
- Cache entries are reused until the file's mtime or size changes.

---

## Troubleshooting

### Columns are not showing up

First restart Nautilus:

```bash
nautilus -q
```

Make sure you are using **List View** and check **Visible Columns**.

If the columns still do not appear, check the user journal:

```bash
journalctl --user -f -t nautilus-media-columns
```

### Mutagen is missing

If you see:

```
    ModuleNotFoundError: No module named 'mutagen'
```

install the Mutagen package for your distribution.

Ubuntu/Debian:

```bash
sudo apt install python3-mutagen
```

Arch/CachyOS:

```bash
sudo pacman -S python-mutagen
```

Then restart Nautilus:

```bash
nautilus -q
```

### GStreamer / GstPbutils is missing

If you see:

```
ValueError: Namespace GstPbutils not available
```

install the GStreamer GObject Introspection typelib.

Ubuntu/Debian:

```bash
sudo apt install gir1.2-gst-plugins-base-1.0
```

Then restart Nautilus.

### No duration or FPS for some videos

Video probing depends on the installed GStreamer plugins.

You may need additional GStreamer packages, for example:

```bash
sudo apt install \
    gstreamer1.0-libav \
    gstreamer1.0-plugins-ugly
```

Package names vary by distribution.

### Audio metadata is missing

Not every audio file contains complete metadata.

For example:

- A file may have a duration but no artist.
- A file may have an artist but no album.
- Some formats store metadata differently.
- Missing tags are displayed as empty columns.

The extension does not invent metadata that is not present in the file.

### Images show no dimensions or limited metadata

The extension prefers **GExiv2** for image metadata.

If GExiv2 is unavailable, it falls back to **GdkPixbuf** with reduced metadata support.

Install the GExiv2 GI bindings if they are available for your distribution.

---

## Debug Logging

### View normal logs

```bash
journalctl --user -f -t nautilus-media-columns
```

Shows:

- INFO
- WARNING
- ERROR

### Enable debug logging

```bash
G_MESSAGES_DEBUG=nautilus-media-columns nautilus
```

Or:

```bash
journalctl --user -f -t nautilus-media-columns -p debug
```

### View debug output directly in the terminal

```bash
G_MESSAGES_DEBUG=nautilus-media-columns nautilus 2>&1 | grep -i nautilus-media-columns
```

### Debug logging includes

- Cache activity
- Media probing
- Audio metadata extraction
- Database lifecycle events
- GStreamer diagnostics
- Metadata and cache errors

Debug logging is disabled by default.

---

## Tested On

- Nautilus 50.3.1 / Arch-based system (without GNOME desktop environment)

The extension targets **nautilus-python API 4.0** and is expected to work across distributions providing compatible Nautilus, GObject Introspection, GStreamer, and Python dependencies.

---

## License

MIT

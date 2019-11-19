# Massaging videos from the command line

## Adding titles to MOV files

Luckily, this works pretty much the same as for images:

```bash
exiftool -overwrite_original -Title="My exciting video" input.mov
```

Took me a lot of trial and error to realize [using `ffmpeg` for this](https://stackoverflow.com/a/11479066) isn't wise (it loses a lot of metadata).

## Losslessly concatenating mp4 files

```bash
(ls input*.mp4 | sed 's/^/file /') \
  | ffmpeg -protocol_whitelist file,pipe -f concat -safe 0 -i pipe: -vcodec copy -acodec copy output.mp4
```

## Losslessly extracting a range from an mp4 file

```bash
ffmpeg -ss 00:00:00 -t 00:50:00 -i input.mp4 -acodec copy -vcodec copy output.mp4
```

Note that the second timestamp is the **length** of the resulting video, **not the end point** of the extraction.

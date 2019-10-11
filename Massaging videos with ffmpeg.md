# Massaging videos with ffmpeg

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

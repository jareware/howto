# Massaging videos with ffmpeg

## Adding titles to MOV files

```bash
ffmpeg -loglevel quiet -i input.mov -codec copy -metadata title="description" output.mov
```

Note that while `title` is a supported metadata field in MOV files, [arbitrary fields aren't](https://stackoverflow.com/a/11479066).

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

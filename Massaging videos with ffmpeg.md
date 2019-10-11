# Massaging videos with ffmpeg

## Losslessly concatenating mp4 files

```bash
(ls input*.mp4 | sed 's/^/file /') \
  | ffmpeg -protocol_whitelist file,pipe -f concat -safe 0 -i pipe: -vcodec copy -acodec copy output.mp4
```

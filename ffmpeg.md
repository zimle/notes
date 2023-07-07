# ffmpeg

[ffmpeg](https://ffmpeg.org/) is a cli tool to convert video and / or audio data.

Here are some examples:

```bash
# take video, cut out from minute 5:13 to 1:12:31,
# convert it with libx265 slowly (higher compression)
# with conversion rate 28 (higher means more compression, less quality, see https://trac.ffmpeg.org/wiki/Encode/H.265#crf
ffmpeg -i "my-source-video.mkv" --ss 00:05:13 -c:v libx265 -preset slow -cr 28 -to 01:12:31 output-video.mp4
# just cut out from minute 5:13 to 1:12:31
ffmpeg -i "my-source-video.mp4" --ss 00:05:13 -c: copy -to 01:12:31 output-video.mp4
# crop image to specified resolution, copy audio
# see also https://ffmpeg.org/ffmpeg-filters.html#cropdetect
# or get preview via `ffplay -i my-source-video.mp4 -t 0.1 -vf cropdetect`
ffmpeg -i "my-source-video.mp4" -vf crop=1824:1024:90:2 -c:a copy output-video.mp4
```

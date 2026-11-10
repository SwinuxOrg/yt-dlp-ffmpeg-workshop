This document contains content that was discarded either because it was inaccurate or rewritten.

# Workshop

## FFmpeg

### Rate Control

Let's try transcoding with a constrained bitrate target of 1500 kBit/s, using `-b 1500k`. We can set a rate control buffer size use `-bufsize` to control how much the bitrate is allowed to vary. Smaller buffer sizes lead to less variation in bitrate. A value of 2 times the max rate is usually recommended (run command, don't play file).

### Subtitles - Separate Track

If multiple codec options for the same stream type are specified, the last one will get the final say. For example, if you specified `-c copy -c:v libx265`, the audio stream will be directly copied while the video stream will be re-encoded to H.265. FFmpeg will warn you of this.

### Rate Control

You can use 2-pass VP9 with constant quality instead of just an average bitrate by setting `-b:v` to 0 and specifying `-crf`. For VP9, the CRF ranges from 0-63 and the recommended value depends on the output resolution. For our 1280x720 video, a CRF of 32 is recommended and will be automatically used by FFmpeg:
```sh
ffmpeg -i sample_video.webm -t 5 -c:v libvpx-vp9 -row-mt 1 -b:v 0 -crf 32 -pass 1 -an -f null <null>
ffmpeg -i sample_video.webm -t 5 -c:v libvpx-vp9 -row-mt 1 -b:v 0 -crf 32 -pass 2 -c:a copy 2pcq.webm
```

> **In the script:**<br>
> You can use 2-pass VP9 with constant quality instead of just an average bitrate by setting `-b:v` to 0 and specifying `-crf`. For VP9, the CRF ranges from 0-63 and the recommended value depends on the output resolution. For our 1280x720 video, a CRF of 32 is recommended.

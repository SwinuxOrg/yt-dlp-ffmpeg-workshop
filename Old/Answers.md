# yt-dlp Exercise

```sh
ytdlp -f "bv[width<=1280][height<=720]+ba" -S vcodec:vp9.2,vcodec:vp9,vcodec:h265,vcodec:h264,vcodec:vp8,vcodec:h263,vcodec:theora,vcodec:other -o "%(title)s [%(id)s] (format limited).%(ext)s" https://youtu.be/Kn56bMZ9OE8
```

# FFmpeg Exercise

```sh
ffmpeg -ss 2:00 -i sample_video.mov -i subs2.vtt -t 1:00 -s 960x540 -r 15 -c:v libx264 -c:a libvorbis -c:s copy -metadata:s:s language=eng -metadata:s:s title="UK English" reduced.mkv
```

# Homework

```sh
ytdlp -f 398+ba --embed-subs --sub-lang en-GB -o vid.webm https://youtu.be/Kn56bMZ9OE8

ytdlp --skip-download --write-auto-subs --sub-lang es-en --sub-format vtt --sleep-subtitles 60 -o auto_subs https://youtu.be/Kn56bMZ9OE8

ffmpeg -i vid.webm -i auto_subs.es-en.vtt -map 0 -map 1 -metadata:s:s:0 language=eng -metadata:s:s:0 title="English (UK)" -metadata:s:s:1 language=spa -metadata:s:s:1 title="Spanish from English (auto)" -c copy dual_subbed.webm
```
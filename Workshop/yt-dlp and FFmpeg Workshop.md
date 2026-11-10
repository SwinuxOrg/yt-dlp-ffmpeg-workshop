# yt-dlp and FFmpeg Workshop

Welcome to the **Swinux yt-dlp and FFmpeg workshop**. Here's a breakdown of what these tools are.

* yt-dlp is a powerful command-line tool for downloading videos from YouTube. It's a continuation of the now-stagnant youtube-dl. While it's primarily used for downloading YouTube videos, it can be used to download media from other video sites and also target specific media streams on YouTube instead of downloading both video and audio.
* FFmpeg is an open-source project for processing multimedia. While it exists as a coding framework, we'll be mainly looking at it's command-line tools for re-encoding and repackaging media files.

Open a terminal/command prompt and change into your directory/folder for this workshop.

## Command List

Since selecting text to copy from a PDF document is difficult, the commands we'll be running will also be provided in `Commands.html`.

Each area in it will be referred to as a command set and is numbered. It uses read-only text boxes to make selection as easy as clicking on them and pressing Ctrl+A.

## yt-dlp

Let's start with yt-dlp. As established in the pre-workshop, it'll be called `ytdlp` instead. Download the video at <https://youtu.be/bCF2bcqCkYs>. To do that, run (command set 1):
```sh
ytdlp https://youtu.be/bCF2bcqCkYs
```

Since FFmpeg is installed, yt-dlp will download the best video and best audio formats and merge them into one file. If yt-dlp couldn't detect FFmpeg, it would just download the best format that contained video and audio, which usually isn't the best quality.

yt-dlp accepts other forms of YouTube URLs, even just the video ID itself. `--force-overwrites` will overwrite existing files, which is useful for testing or learning yt-dlp, especially in this workshop. Be mindful of when you use it (command set 2):
```sh
ytdlp --force-overwrites https://youtube.com/watch?v=bCF2bcqCkYs
ytdlp --force-overwrites bCF2bcqCkYs
```

<div class="page"/>

### Updates

YouTube is constantly trying to break 3rd-party clients, and yt-dlp is no exception. So if yt-dlp is suddenly no longer able to download anything, consider updating it. While you can manually re-download yt-dlp, it's often better to use its built-in updating mechanism (command set 3):
```sh
ytdlp -U
```

Don't use this for the version of yt-dlp you installed via a package manager (apt, pacman, chocolatey, etc.).

### Output Template

We can control the output filename with `-o`, which is used to specify an output template. For example, this command will mimic yt-dlp's default output template (command set 4):
```sh
ytdlp -o "%(title)s [%(id)s].%(ext)s" --force-overwrites https://youtu.be/bCF2bcqCkYs
# You don't even need a space after '-o', so this works too:
ytdlp -o"%(title)s [%(id)s].%(ext)s" --force-overwrites https://youtu.be/bCF2bcqCkYs
```

Explanation of each field:
* `%(title)s` - The title of the video on YouTube.
* `%(id)s` - The ID of the video in the URL.
* `%(ext)s` - The file extension recommended by YouTube.

You can find more output template fields in `man yt-dlp` (search for "Output Template") or the README on the repo: <https://github.com/yt-dlp/yt-dlp?tab=readme-ov-file#output-template>.

<div class="page"/>

### Format Selection

We can download a specific format for a video with the `-f` argument. To list available formats for a video, use `-F` (command set 5):
```
$ ytdlp -F https://youtu.be/bCF2bcqCkYs
...
[info] Available formats for bCF2bcqCkYs:
ID  EXT   RESOLUTION FPS CH │   FILESIZE   TBR PROTO │ VCODEC         VBR ACODEC      ABR ASR MORE INFO
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
sb3 mhtml 48x27        0    │                  mhtml │ images                                 storyboard
sb2 mhtml 80x45        0    │                  mhtml │ images                                 storyboard
...
139 m4a   audio only      2 │    5.70MiB   49k https │ audio only         mp4a.40.5   49k 22k [en] low, m4a_dash
249 webm  audio only      2 │    5.91MiB   51k https │ audio only         opus        51k 48k [en] low, webm_dash
...
248 webm  1920x1080   30    │   67.30MiB  576k https │ vp9           576k video only          1080p, webm_dash
399 mp4   1920x1080   30    │   37.18MiB  318k https │ av01.0.08M.08 318k video only          1080p, mp4_dash
```

Let's say we wanted to download the best AAC (MP4 Audio) format. That would be format 140 (command set 6):
```sh
ytdlp -f 140 --force-overwrites https://youtu.be/bCF2bcqCkYs
```

We can download just the best audio or video stream on it's own with `-f ba` or `-f bv` respectively. The former is useful if we just want to listen to something that was on YouTube (command set 7):

```sh
ytdlp -f ba --force-overwrites https://youtu.be/bCF2bcqCkYs
ytdlp -f bv --force-overwrites https://youtu.be/bCF2bcqCkYs
```

<div class="page"/>

### Filtering Formats

We can add filters to video and audio formats. For example, to download the best video format with a height no greater than 720, run this. The quotes are important because `<` has special meaning in the command-line (command set 8):
```sh
ytdlp -f "bv[height<=720]" --force-overwrites https://youtu.be/bCF2bcqCkYs
```

The above will only download the video stream. To include the audio, add `+ba` (command set 9):
```sh
ytdlp -f "bv[height<=720]+ba" --force-overwrites https://youtu.be/bCF2bcqCkYs
```

We can combine filters. To limit the video to fit within 1280x720, run (command set 10):
```sh
ytdlp -f "bv[width<=1280][height<=720]+ba" --force-overwrites https://youtu.be/bCF2bcqCkYs
```

Of course, it doesn't matter in this case because the video was already within 1280x720.

To learn more about this feature, check `man yt-dlp` (search for "Filtering Formats") or the relevant README section: <https://github.com/yt-dlp/yt-dlp?tab=readme-ov-file#filtering-formats>.

<div class="page"/>

### Sorting Formats

We can define what format is considered the best using `-S`.

A good use case for this is to prioritise certain codecs over others. This is because older devices won't have hardware acceleration for decoding certain video codecs, making playback of them choppy or power-hungry.

For example, many older devices will only natively support up to H.264/AVC. This command will try to download the video in H.264, falling back to another codec if an H.264 format isn't available (command set 11):
```sh
ytdlp -S vcodec:h264,vcodec:vp8,vcodec:h263,vcodec:theora,vcodec:other --force-overwrites https://youtu.be/bCF2bcqCkYs
```

The list goes from highest to lowest priority. We can even omit codecs that we don't want to be chosen as a fallback.

According to `man yt-dlp` and the README, the default video codec priority used by yt-dlp is: `av01 > vp9.2 > vp9 > h265 > h264 > vp8 > h263 > theora > other`

To learn more about this feature, search for "Sorting Formats" in `man yt-dlp` or visit the relevant README section: <https://github.com/yt-dlp/yt-dlp?tab=readme-ov-file#sorting-formats>.

<div class="page"/>

### Subtitles

By default, yt-dlp doesn't download subtitles, we'll have to manually specify which subtitle tracks to download. For this section, we'll be using a different video that's larger but has subtitles: <https://youtu.be/Kn56bMZ9OE8>

To list available subtitle tracks, use `--list-subs` (command set 12):
```
$ ytdlp --list-subs https://youtu.be/Kn56bMZ9OE8
...
[info] Available automatic captions for Kn56bMZ9OE8:
Language      Name                                                                   Formats
en            English, English, English, English, English, English, English, unknown vtt, srt, ttml, srv3, srv2, srv1, json3, vtt
en-GB                                                                                vtt
ab-en         Abkhazian from English                                                 vtt, srt, ttml, srv3, srv2, srv1, json3
aa-en         Afar from English                                                      vtt, srt, ttml, srv3, srv2, srv1, json3
...
[info] Available subtitles for Kn56bMZ9OE8:
Language Name                     Formats
en       English                  vtt, srt, ttml, srv3, srv2, srv1, json3
en-GB    English (United Kingdom) vtt, srt, ttml, srv3, srv2, srv1, json3
```

The section at the end called "Available subtitles", refers to non-automatic subtitles. The default subtitle language used by yt-dlp is `en`. To download the video with it, use `--embed-subs`. We'll use format `396` with no audio to reduce download time (command set 13):
```sh
ytdlp -f 396 --embed-subs --force-overwrites https://youtu.be/Kn56bMZ9OE8
```

We can choose a different language, such as `en-GB` with `--sub-lang`. We can specify multiple languages, separated with commas, or choose them all with `all` (command set 14):
```sh
ytdlp -f 396 --embed-subs --sub-lang en,en-GB --force-overwrites https://youtu.be/Kn56bMZ9OE8
ytdlp -f 396 --embed-subs --sub-lang all --force-overwrites https://youtu.be/Kn56bMZ9OE8
```

<div class="page"/>

Note that if we're specifying an output template while embedding subtitles, we'll have to make sure to specify the original file extension (such as with `%(ext)s`). Otherwise, we'll get the following error (command set 15):
```
$ ytdlp -f 396 --embed-subs -o my_template.webm https://youtu.be/Kn56bMZ9OE8
# OR
$ ytdlp -f 396 --embed-subs -o my_template https://youtu.be/Kn56bMZ9OE8
...
ERROR: Postprocessing: Error opening output files: Invalid argument
```

Command Set 16:
```
# Works
ytdlp -f 396 --embed-subs -o my_template.mp4 https://youtu.be/Kn56bMZ9OE8
```

To download subtitles into their own file instead of embedding them into the video file, use `--write-subs` instead of `--embed-subs`. To skip downloading video or audio, use `--skip-download` (command set 17):
```sh
$ ytdlp --skip-download --write-subs --sub-lang all --force-overwrites https://youtu.be/Kn56bMZ9OE8
...
[info] Writing video subtitles to: This bus transforms into a train [Kn56bMZ9OE8].en.vtt
[download] Destination: This bus transforms into a train [Kn56bMZ9OE8].en.vtt
...
[info] Writing video subtitles to: This bus transforms into a train [Kn56bMZ9OE8].en-GB.vtt
[download] Destination: This bus transforms into a train [Kn56bMZ9OE8].en-GB.vtt
...
```

<div class="page"/>

You may notice that subtitles downloaded separately with an output template (`-o`) will still have a file extension (command set 18):
```
$ ytdlp --skip-download --write-subs --sub-lang all -o foo --force-overwrites https://youtu.be/Kn56bMZ9OE8
...
[info] Writing video subtitles to: foo.en.vtt
[download] Destination: foo.en.vtt
...
[info] Writing video subtitles to: foo.en-GB.vtt
[download] Destination: foo.en-GB.vtt
...
```

Beware that, unlike Celluloid, VLC will show subtitle files with the same name as the video file as subtitle tracks that appear to be part of the file itself. For this reason, you may want to delete standalone subtitle files after downloading them.

Let's try to download automatic subtitles using `--write-auto-subs` (command set 19):
```
$ ytdlp -f 396 --embed-subs --write-auto-subs --force-overwrites https://youtu.be/Kn56bMZ9OE8
...
[info] Writing video subtitles to: This bus transforms into a train [Kn56bMZ9OE8].en.vtt
ERROR: Unable to download video subtitles for 'en': HTTP Error 429: Too Many Requests
```

This failed because it tried to download an auto-translated subtitle track, which requires PO (proof-of-origin) tokens or a 60-second delay. YouTube chose Japanese instead of English for this video's untranslated automatic subtitles, likely due to the high amount of Japanese speech present in its audio. yt-dlp chooses English subtitles by default, which would be an auto-translation in this case. PO tokens are a way for a YouTube client to prove that they're legitimate.

The untranslated Japanese track is called `ja-orig`, but we can use regex to avoid having to explicitly write the language code for this track by writing `.*-orig` (command set 20):
```
ytdlp -f 396 --embed-subs --write-auto-subs --sub-lang .*-orig --force-overwrites https://youtu.be/Kn56bMZ9OE8
```

Explanation:
* `.` means "any character".
* `*` means "none or any amount of these characters".

<div class="page"/>

If we wanted to download auto-translated subtitles, we would have to use one of the following:
* A PO Token Provider Plugin - Out of scope, but if you're interested: <https://github.com/yt-dlp/yt-dlp/wiki/PO-Token-Guide#po-token-provider-plugins>
* `--sleep-subtitles 60` - Number of seconds to sleep before each subtitle download.

We will use `--sleep-subtitles 60` to download automatic Spanish and English subtitles. This'll take more than 2 minutes, so enjoy the break I guess (command set 21):
```sh
ytdlp -f 396 --embed-subs --write-auto-subs --sub-lang en,es --sleep-subtitles 60 --force-overwrites https://youtu.be/Kn56bMZ9OE8
```

To download automatic subtitles into their own files without any video or audio, simply remove `--embed-subs` and add `--skip-download` (command set 22):
```sh
ytdlp --skip-download --write-auto-subs --sub-lang .*-orig --force-overwrites https://youtu.be/Kn56bMZ9OE8
```

<div class="page"/>

You may notice how there's more options for subtitle formats than just `vtt`. Let's look at the subtitle list we got from earlier (command set 23):
```
$ ytdlp --list-subs https://youtu.be/Kn56bMZ9OE8
...
[info] Available subtitles for Kn56bMZ9OE8:
Language Name                     Formats
en       English                  vtt, srt, ttml, srv3, srv2, srv1, json3
en-GB    English (United Kingdom) vtt, srt, ttml, srv3, srv2, srv1, json3
```

Let's download both non-automatic subtitle tracks in `srt` format using `--sub-format` (command set 24):
```sh
$ ytdlp -f 396 --skip-download --write-subs --sub-langs all --sub-format srt --force-overwrites https://youtu.be/Kn56bMZ9OE8
...
[info] Writing video subtitles to: This bus transforms into a train [Kn56bMZ9OE8].en.srt
[download] Destination: This bus transforms into a train [Kn56bMZ9OE8].en.srt
...
[info] Writing video subtitles to: This bus transforms into a train [Kn56bMZ9OE8].en-GB.srt
[download] Destination: This bus transforms into a train [Kn56bMZ9OE8].en-GB.srt
...
```

Note that it's not possible to download both automatic and non-automatic subtitles in a single command. We'll need 2 yt-dlp commands and another tool to merge them all, which we will cover later in the workshop.

<div class="page"/>

### Other Sites

We can download media from other sites, such as this video from Reddit: <https://www.reddit.com/r/unixporn/comments/1ruepe4/hyprland_this_hyprland_starting_animation_hits/>

We can list the formats, just like with YouTube (command set 25):
```
$ ytdlp -F https://www.reddit.com/r/unixporn/comments/1ruepe4/hyprland_this_hyprland_starting_animation_hits/
...
[info] Available formats for v5pagsfkr7pg1:
ID            EXT RESOLUTION FPS │   FILESIZE   TBR PROTO │ VCODEC        VBR ACODEC      ABR ASR MORE INFO
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
hls-7-audio_0 mp4 audio only     │                  m3u8  │ audio only        unknown             audio 0, low
hls-6-audio_0 mp4 audio only     │                  m3u8  │ audio only        unknown             audio 0, high
dash-6        m4a audio only     │ ~516.84KiB   68k https │ audio only        mp4a.40.2   68k 48k [en] DASH audio, m4a_dash
dash-7        m4a audio only     │ ~999.30KiB  132k https │ audio only        mp4a.40.2  132k 48k [en] DASH audio, m4a_dash
...
hls-1794      mp4 1280x720    30 │ ~ 13.26MiB 1794k m3u8  │ avc1.4d401f 1794k video only
fallback      mp4 1920x1080      │ ≈ 36.95MiB 5000k https │ h264        5000k video only          DASH video, mp4_dash
```

And to download it (command set 26):
```sh
ytdlp https://www.reddit.com/r/unixporn/comments/1ruepe4/hyprland_this_hyprland_starting_animation_hits/
```

<div class="page"/>

### Exercise

Let's try to form our own yt-dlp command that fits the following criteria:

* We want to download the video at <https://youtu.be/Kn56bMZ9OE8>.
* We want the highest resolution that fits within 1280x720.
* We want the best audio format.
* We want a video codec no newer than VP9 Profile 2. Failing that, we want it in the best codec (based on yt-dlp's codec preferences) available.
* We want the output filename to be in the following format (where `<>` represents a variable field): `<title> [<id>] (format limited).<file extension>`

Let's see if you can figure this one out yourself.

<div class="page"/>

## FFmpeg

We will now move on to FFmpeg. We will be working with these files from the pre-workshop:
* `sample_video.mov` (`Kn56bMZ9OE8`)
* `sample_video.webm` (`Kn56bMZ9OE8`)
* `subs1.vtt` (`en` track of `Kn56bMZ9OE8`)
* `subs2.vtt` (`en-GB` track of `Kn56bMZ9OE8`)

### FFprobe

We can use FFprobe to check what streams are present in the video file and information about their codecs (command set 27):
```
$ ffprobe sample_video.mov
...
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'sample_video.mov':
...
  Duration: 00:05:03.72, start: 0.000000, bitrate: 945 kb/s
  Stream #0:0[0x1]: Video: h264 (Main) (avc1 / 0x31637661), yuv420p(tv, bt709, progressive), 1280x720 [SAR 1:1 DAR 16:9], 808 kb/s, 29.97 fps, 29.97 tbr, 30k tbn (default)
    Metadata:
      handler_name    : ISO Media file produced by Google Inc.
      vendor_id       : FFMP
  Stream #0:1[0x2](eng): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 127 kb/s (default)
    Metadata:
      handler_name    : ISO Media file produced by Google Inc.
      vendor_id       : [0][0][0][0]
```

This tells us that:
* The video stream (Stream #0:0) is encoded with H.264 in 720p 30 FPS.
* The audio stream (Stream #0:2) is in English and encoded with AAC in stereo.

### Containers and Copying

Let's copy this to an MP4 container instead of a MOV container. To do this, we'll use `ffmpeg` with `-c copy` to simply copy the codec data as is without re-encoding it (command set 28):
```sh
ffmpeg -i sample_video.mov -c copy sample_video.mp4
```

<div class="page"/>

Normally FFmpeg will automatically determine the output container type from our chosen output name, but if we want to explicitly specify the container type, use `-f` (command set 29):
```sh
ffmpeg -i sample_video.mov -c copy -f mp4 sample_video_no_ext
```

We can confirm that they're identical from the command-line. On Windows, run `fc` (command set 30):
```
>fc sample_video.mp4 sample_video_no_ext
Comparing files sample_video.mp4 and SAMPLE_VIDEO_NO_EXT
FC: no differences encountered
```

On a POSIX-based system (Linux, macOS, etc.), `diff` should output nothing, meaning no difference (command set 31):
```sh
diff sample_video.mp4 sample_video_no_ext
```

Be sure to check if the container you're choosing supports the codecs you're copying. FFmpeg will generally warn you if this isn't the case. As for how to decide what container to use:
* MPEG-4 (MP4) is a very widely supported format, but isn't open.
  * It is one of 2 video formats Reddit will accept in an upload, the other being QuickTime Movie (MOV).
* Matroska (MKV) is an open format.
  * It's very versatile, able to contain an unlimited amount of video, audio or subtitle tracks in a single file. Recommended for arbitrary combinations of codecs<sup><a id="ref-1-usage" href="#ref-1-src">\[1]</a></sup>.
  * However, its use may not be supported everywhere.
* WebM is an open format.
  * It's based on MKV, but is not as versatile<sup><a id="ref-2-usage" href="#ref-2-src">\[2]</a></sup>.
    * For example, it doesn't support H.264 or H.265.
  * It's supported by most browsers.

We can extract the video or audio into their own file using the `-map` option, which lets us choose which streams of which files we want to use (command set 32):
```sh
ffmpeg -i sample_video.mp4 -map 0:v -c copy video_only.mp4
ffmpeg -i sample_video.mp4 -map 0:a -c copy audio_only.mp4
```

If we have multiple of the same type of stream, we can specify which one we want with an extra number. For example, `-map 0:a:0` and `-map 0:a:1` will select the first and second audio streams respectively.

You can learn more about `-map` in `man ffmpeg` (search for "Advanced options") or the online documentation:
<https://www.ffmpeg.org/ffmpeg-all.html#Advanced-options>

Another way to filter out video or audio is with the `-vn` and `-an` options (command set 33):
```sh
ffmpeg -i sample_video.mp4 -an -c copy no_audio.mp4
ffmpeg -i sample_video.mp4 -vn -c copy no_video.mp4
```

### Basic Transcoding

Before, we simply copied the codec data as is by specifying `-c copy`. By not specifying `-c`, FFmpeg will re-encode every stream using the default codec for each stream type (video, audio, etc.), which depends on your build of FFmpeg and output container format. This is why it's important to specify the target codec with `-c` when outputting to a non-codec-specific container.

Let's start with extracting the audio of our video to an audio-only file type. To extract it to an MP3 file (command set 34):
```sh
ffmpeg -i sample_video.mp4 audio.mp3
```

Because MP3 files are specific to the MP3 codec, we didn't need to specify the codec. This applies to other codec-specific audio file types, such as `.opus`. Beware that `.ogg` isn't specific to the Vorbis audio codec. While it did use to only support Vorbis, it now supports multiple codecs, even Theora, which is a video codec.

Let's transcode our video to some more modern codecs, we'll be using AV1 and Opus. We'll also only be processing the first 10 seconds of the video to save time. We'll be using the `-t`, `-c:v` and `-c:a` options for this (command set 35):
```sh
ffmpeg -i sample_video.mp4 -t 10 -c:v libsvtav1 -c:a libopus transcode.webm
```

The accepted time formats can be found in `man ffmpeg-utils` (search for "Time duration") or the online documentation: <https://www.ffmpeg.org/ffmpeg-utils.html#time-duration-syntax>

We can seek to a different starting point in the video using `-ss`. This will use the section between the 5 and 15-second mark (command set 36):
```sh
ffmpeg -ss 5 -i sample_video.mp4 -t 10 -c:v libsvtav1 -c:a libopus clip.webm
```

It's recommended that `-ss` is specified before inputs to avoid pointless decoding of everything before the seek point.

<div class="page"/>

We can specify the end point of the section we want instead of the duration using `-to`. This time, `-ss` must go after the input (command set 37):
```sh
ffmpeg -i sample_video.mp4 -ss 5 -to 15 -c:v libsvtav1 -c:a libopus clip.webm
```

If you specify both `-t` and `-to`, `-t` will take priority.

We're using WebM instead of MP4, due to being a more open format and the fact that the new codecs are supported by it. As for the values passed to `-c:v` and `-c:a`, these are encoders that are included in FFmpeg:
* `libsvtav1` refers to SVT-AV1, a multithreaded software encoder for AV1.
* `libopus` is the official library for working with the Opus codec.

Here's some other encoders you may be interested in:
* `libx264` - H.264/AVC
* `libx265` - H.265/HEVC
* `libvpx-vp9` - VP9
* `libaom-av1` - The original, reference encoder for AV1. It lacks multithreading support, so there's not much reason to use it over SVT-AV1.
* `pcm_s16le` - Signed 16-Bit Little-Endian PCM
  * The default audio format FFmpeg uses when outputting to WAV files.
  * This is equivalent to the uncompressed audio used on audio CDs.
* `libmp3lame` - LAME Encoder for MP3
* `aac` - Native FFmpeg AAC Encoder
  * Not the best encoder you can be using. These 2 are better, but require you to build and include them in FFmpeg yourself:
    * Fraunhofer FDK AAC (`libfdk_aac`)
    * Apple AudioToolbox Encoder (`aac_at`) - Licensing and platform issues make this annoying to implement.
* `libvorbis` - Vorbis
* `flac` - FLAC (Free Lossless Audio Codec)

We can list included encoders with `ffmpeg -encoders`. To list both decoders and encoders, use `ffmpeg -codecs`. We can even use hardware acceleration for encoding, but this is hardware dependant (Intel VA-API V.S. NVIDIA VDPAU) and out of scope.

<div class="page"/>

### Subtitles

Subtitles can be added to a video file in 2 ways:
* Adding them as a separate track.
* Baking them into the video stream.

### Subtitles - Separate Track

Let's try adding `subs1.vtt` as a separate track in our video files, starting with the WebM one (command set 38):
```sh
ffmpeg -i sample_video.webm -i subs1.vtt -c copy subbed.webm
```

The track will be visible as an option in VLC, but there will be no title or language indicator, it will simply show up as "Track 1".

We can label the language as English by specifying it as a metadata tag (command set 39):
```sh
ffmpeg -i sample_video.webm -i subs1.vtt -metadata:s:s language=eng -c copy subbed.webm
```

The track should now show up as English in the subtitle list.

The first `s` means "stream", since we're applying the tag to a media stream. The second `s` means "subtitle". The `language` tag uses ISO 629-2 language codes<sup><a id="ref-3-usage" href="#ref-3-src">\[3]</a></sup>, which can be found here:
* Wikipedia: <https://en.wikipedia.org/wiki/List_of_ISO_639-2_codes>
* U.S. Library of Congress: <https://www.loc.gov/standards/iso639-2/php/code_list.php>

You can learn more about the `-metadata` option in `man ffmpeg` (search for "Advanced options") or FFmpeg's documentation: <https://www.ffmpeg.org/ffmpeg.html#Advanced-options>

A track title of "Foo Bar" can also be set as a metadata tag (command set 40):
```sh
ffmpeg -i sample_video.webm -i subs1.vtt -metadata:s:s language=eng -metadata:s:s title="Foo Bar" -c copy subbed.webm
```

The track should now show up as "Foo Bar" in the subtitle list.

<div class="page"/>

When adding multiple subtitle tracks, we can specify which track to target by adding a colon and the index of the subtitle track. Note that this also requires selecting both subtitle files to be included via `-map`, otherwise only the first subtitle file will be included. `0`, `1` and `2` refer to `sample_video.webm`, `subs1.vtt` and `subs2.vtt` respectively (command set 41):
```sh
ffmpeg -i sample_video.webm -i subs1.vtt -i subs2.vtt -map 0 -map 1 -map 2 -metadata:s:s:0 language=eng -metadata:s:s:0 title="Foo Bar 1" -metadata:s:s:1 language=eng -metadata:s:s:1 title="Foo Bar 2" -c copy subbed.webm
```

The previous instructions apply to MKV, since WebM is directly based on it. MP4, on the other hand, requires different instructions. MP4 doesn't officially support subtitle tracks, but there are 2 popular representations for them, QuickTime and Nero chapters, both of which will be written by FFmpeg.

MP4 only supports the `mov_text` codec for subtitles, so we'll have to re-encode our subtitle file to it. Only subtitles will be re-encoded (command set 42):
```sh
ffmpeg -i sample_video.mp4 -i subs1.vtt -c copy -c:s mov_text subbed.mp4
```

Note that `-c:s mov_text` takes priority over `-c copy` because it was specified last, causing the subtitles to be transcoded instead of being copied like the other streams.

We can set the language code and track title using the same metadata tags from before (command set 43):
```sh
ffmpeg -i sample_video.mp4 -i subs1.vtt -metadata:s:s language=eng -metadata:s:s title="Foo Bar" -c copy -c:s mov_text subbed.mp4
```

There's a caveat however: Older versions of FFprobe, like the one I got from Ubuntu's repository, can't see the subtitle track's title set using this command (command set 44):
```sh
ffprobe subbed.mp4
```

To solve this, specify it in the `handler_name` tag as well. yt-dlp also does this when embedding subtitles in an MP4 download (command set 45):
```sh
ffmpeg -i sample_video.mp4 -i subs1.vtt -metadata:s:s language=eng -metadata:s:s title="Foo Bar" -metadata:s:s handler_name="Foo Bar" -c copy -c:s mov_text subbed.mp4
```

Note that Celluloid can't see the title of subtitle tracks in MP4 files. This issue has already been reported on its GitHub repository and should be fixed in the next update.

### Subtitles - Baked-In

We can bake subtitles into the video stream, ignoring limitations surrounding subtitle tracks entirely. This is achieved using the `subtitles` video filter (`-vf`). This requires re-encoding the video stream. We'll use AV1 and only process the first 10 seconds of the video to save time (command set 46):
```sh
ffmpeg -i sample_video.webm -vf subtitles=subs1.vtt -t 10 -c:v libsvtav1 -c:a copy baked.webm
```

You can see what other options are available for this video filter in `man ffmpeg-filters` (search for "subtitles") or FFmpeg's documentation: <https://ffmpeg.org/ffmpeg-filters.html#subtitles-1>

### Resolution

We can transcode quality elements of video and audio streams.

To change the resolution (or "frame size" as FFmpeg's documentation calls it) of a video, use `-s`. For example, to set the resolution to 640x360 (again, only processing the first 10 seconds to save time) (command set 47):
```sh
ffmpeg -i sample_video.webm -t 10 -s 640x360 -c copy -c:v libsvtav1 lowres.webm
```

To change the framerate of a video, use `-r`. For example, to set the framerate to 10 FPS (command set 48):
```sh
ffmpeg -i sample_video.webm -t 10 -r 10 -c copy -c:v libsvtav1 lowfps.webm
```

You may notice that VLC doesn't always seem to show the framerate of a video under Codec Information. Fortunately, FFprobe and Celluloid/mpv do (command set 49):
```
$ ffprobe lowfps.webm
...
  Stream #0:0(eng): Video: av1 (Main), yuv420p(tv, bt709, progressive), 1920x1080, SAR 1:1 DAR 16:9, 10 fps, 10 tbr, 1k tbn (default)
...
```

To change the audio sample rate of a video, use `-ar`. For example, to set the rate to 12000 Hz (command set 50):
```sh
ffmpeg -i sample_video.webm -t 10 -ar 12000 -c copy -c:a libopus lowfreq.webm
```

Note that most codecs will have restrictions on what sample rates they support. Opus only supports sample rates of 8000, 12000, 16000, 24000 and the default value of 48000 Hz<sup><a id="ref-4-usage" href="#ref-4-src">\[4]</a></sup>.

Notice how the audio quality is much lower because the lower sample rate doesn't support higher frequencies. According to the Nyquist–Shannon sampling theorem, a sample rate of 12000 Hz can't support frequencies above 6000 Hz. Don't use this as a means of reducing file size however. Instead, consider reducing the bitrate.

### Rate Control

If our goal is to reduce the size of a media file, a good way to do so is by reducing the bitrate. There's 3 methods for controlling the video bitrate of media:
* Constant Bitrate - Targets an average bitrate across the whole duration. May not perfectly achieve said average bitrate.
* Constant Quality - Targets a subjective quality value across the whole duration, increasing bitrate at sections with more detail or motion to maintain the quality target.
* Constrained Bitrate - Like constant quality, but with an upper-limit on how high the bitrate can be at any given moment. This helps to ensure the media stays within a certain file size.
* 2-Pass - Useful for more size-efficient encodes targetting a specific bit rate at the cost of the consistency of frame-to-frame quality.

To set a constant bitrate of 1500 kBit/s with SVT-AV1, use `-rc 2` and the `-b:v` option. Note that the bitrate value only applies to the video stream (command set 51):
```sh
ffmpeg -i sample_video.webm -t 10 -c:v libsvtav1 -rc 2 -b:v 1500k -c:a copy ctbr.webm
```

`-rc` is an SVT-AV1 parameter that specifies the rate control mode. A value of 2 indicates that we want constant bitrate encoding. You can learn more about this option here: <https://gitlab.com/AOMediaCodec/SVT-AV1/-/blob/master/Docs/Parameters.md#rate-control-options>

Let's move on to constant quality. SVT-AV1 uses constant quality encoding by default. For most codecs, the quality target can be set using the `-crf` option. This stands for Constant Rate Factor. Lower values correspond to higher quality and larger file sizes. For AV1, the CRF ranges from 0-63 and defaults to 35. Let's try transcoding with the default CRF and a lower-quality CRF (command set 52):
```sh
ffmpeg -i sample_video.webm -t 10 -c:v libsvtav1 -c:a copy cq.webm
ffmpeg -i sample_video.webm -t 10 -c:v libsvtav1 -crf 47 -c:a copy cq47.webm
```

As we can see, the second file is smaller despite both being the first 10 seconds of the video.

<div class="page"/>

Let's try transcoding with a constrained bitrate target of 1500 kBit/s, using `-b 1500k` (command set 53):
```sh
ffmpeg -i sample_video.webm -t 10 -c:v libsvtav1 -maxrate 1500k -c:a copy crbr.webm
```

This file is slightly smaller than the constant bitrate one, due to scenes where that much bitrate isn't needed to reach the target CRF.

And finally, we'll move on to 2-pass. Unfortunately, SVT-AV1 doesn't support it, and while `libaom-av1` does, its lack of multithreading support makes it not worth using. Fortunately, `libvpx-vp9` supports it. In fact, it's actually the recommended way of using that encoder.

Using two-pass with FFmpeg requires 2 commands. This is how to do it in VP9 while targetting an average bitrate of 1500 kBit/s, which allows for more control of the file size. We'll only process the first 5 seconds due to the slower nature of `libvpx-vp9`. `-row-mt 1` enables row-based multithreading for `libvpx-vp9`.(command set 54):
```sh
ffmpeg -i sample_video.webm -t 5 -c:v libvpx-vp9 -row-mt 1 -b:v 1500k -pass 1 -an -f null -
ffmpeg -i sample_video.webm -t 5 -c:v libvpx-vp9 -row-mt 1 -b:v 1500k -pass 2 -c:a copy 2pab.webm
```

You may notice the existence of a log file used to share information between each command. It should be called something like `ffmpeg2pass-0.log`. `-f null` prevents a media file from being outputted as it's not necessary on the first pass. The output option is conventionally set to `-` when doing this.

We can use 2-pass VP9 with constant quality instead of just an average bitrate by setting `-b:v` to 0 and optionally specifying `-crf`. For VP9, the CRF ranges from 0-63 and the recommended value depends on the output resolution. For our 1280x720 video, a CRF of 32 is recommended and will be automatically used by FFmpeg (command set 55):
```sh
ffmpeg -i sample_video.webm -t 5 -c:v libvpx-vp9 -row-mt 1 -b:v 0 -pass 1 -an -f null -
ffmpeg -i sample_video.webm -t 5 -c:v libvpx-vp9 -row-mt 1 -b:v 0 -pass 2 -c:a copy 2pcq.webm
```

To see recommended CRFs for each resolution, visit: <https://developers.google.com/media/vp9/settings/vod/>

<div class="page"/>

The required settings for each bitrate control method depend on the encoder being used, so be sure to check the FFmpeg wiki pages for each encoder. Here are ones for popular codecs:
* H.264/AVC: <https://trac.ffmpeg.org/wiki/Encode/H.264>
* H.265/HEVC: <https://trac.ffmpeg.org/wiki/Encode/H.265>
* VP9: <https://trac.ffmpeg.org/wiki/Encode/VP9>
* AV1: <https://trac.ffmpeg.org/wiki/Encode/AV1>

To set audio bitrate, use `-b:a`. FFmpeg defaults to 96 kBit/s for `libopus`. This sets the audio bitrate to 10 kBit/s (command set 56):
```sh
ffmpeg -i sample_video.webm -t 10 -c:v copy -c:a libopus -b:a 10k abr.webm
```

VLC doesn't seem to show the bitrate of an Opus stream. Fortunately, Celluloid/mpv does.

Opus supports a bitrate between 6-150 kBit/s<sup><a id="ref-5-usage" href="#ref-5-src">\[5]</a></sup>. 10 kBit/s was chosen to make the effect of the reduced bitrate obvious. For bitrate recommendations, visit: <https://wiki.xiph.org/Opus_Recommended_Settings>

### Exercise

Let's try to form our own FFmpeg command that fits the following criteria:
* We want to process the third minute of `sample_video.mov`.
* We want to change the resolution to 960x540.
* We want to change the framerate to 15 FPS.
* We want to add `subs2.vtt` as a subtitle track.
  * It must be labelled as an English track.
  * It must be called "UK English".
* We want to export it in H.264, Vorbis and the original subtitle codec to an MKV file called `reduced.mkv`.

Let's see if you can figure this one out yourself.

<div class="page"/>

## References

* <a id="ref-1-src" href="#ref-1-usage">\[1]</a>: <https://web.archive.org/web/20180204034747/https://www.matroska.org/technical/specs/index.html>
  > using more than 127 tracks is not encouraged, though the design allows an unlimited number
* <a id="ref-2-src" href="#ref-2-usage">\[2]</a>: <https://www.webmproject.org/about/faq/>
  > The WebM file structure is based on the Matroska media container.
* <a id="ref-3-src" href="#ref-3-usage">\[3]</a>: <https://ffmpeg.org/doxygen/7.0/group__metadata__api.html>
  > main language in which the work is performed, preferably in ISO 639-2 format.
* <a id="ref-4-src" href="#ref-4-usage">\[4]</a>: <https://opus-codec.org/docs/opus_api-1.6.pdf>, Page 19.
  > Sample rate to decode at (Hz). This must be one of 8000, 12000, 16000, 24000, or 48000.
* <a id="ref-5-src" href="#ref-5-usage">\[5]</a>: <https://www.opus-codec.org/>
  > Bitrates from 6 kb/s to 510 kb/s
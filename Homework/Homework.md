# yt-dlp and FFmpeg Workshop - Homework

yt-dlp can't download both non-automatic and automatic subtitles in the same command. This also means that it can't embed both of them in the same video file. What we can do however, is download them separately and merge them together with FFmpeg.

What I want you to do is figure out how to do all this in 3 commands. Requirements:
* Target Video: <https://youtu.be/Kn56bMZ9OE8>
* Target Video Formats: `398` and Best Audio
* Target Subtitles:
  * Non-Automatic: `en-GB`, called "English (UK)" in the final file.
  * Automatic: `es-en`, called "Spanish from English (auto)" in the final file.
  * They must be labelled with the correct language code too.
* Final Output Name: `dual_subbed.webm`

Hints:
* Use `-o` and `--sub-format` to make the names of the files you're working with more predictable.
* The video and audio should merge into a WebM file.
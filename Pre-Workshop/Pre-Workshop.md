# yt-dlp and FFmpeg Workshop - Pre-Workshop

It is assumed that you know basic CLI usage.

You will need to install `yt-dlp` and `ffmpeg`. You will also need to install VLC Media Player or Celluloid (based on mpv, only available on Linux), as these players support a wide range of formats and have good statistics displays for showing information such as the codecs being used. VLC is recommended as it shows information for subtitle tracks, which is important for this workshop.

Create a directory/folder for all of the work you will be doing in the workshop, and change into it. Copy the files under "Resources" into it, these will be used in the FFmpeg section of the workshop.

## Installing yt-dlp

yt-dlp can be found on GitHub: <https://github.com/yt-dlp/yt-dlp/releases/latest>

A JavaScript runtime should be installed for full functionality of yt-dlp. This is the warning you'll get from running yt-dlp without it:
```
WARNING: [youtube] No supported JavaScript runtime could be found. Only deno is enabled by default; to use another runtime add  --js-runtimes RUNTIME[:PATH]  to your com
mand/config. YouTube extraction without a JS runtime has been deprecated, and some formats may be missing. See  https://github.com/yt-dlp/yt-dlp/wiki/EJS  for details on
 installing one
```

As the warning says, Deno is recommended.

### Windows

Download `yt-dlp.exe` to your workshop directory. If you are on an ARM laptop, download `yt-dlp_arm64.exe` instead. We'll assume none of you are still on 32-bit Windows, but if you are, download `yt-dlp_x86.exe`.

Rename the executable to `ytdlp.exe`. This is to make sure everyone can follow the same instructions.

Open the command prompt and test if yt-dlp is executable with:
```
ytdlp --help
```

To install Deno, visit <https://docs.deno.com/runtime/getting_started/installation/> and follow the Windows PowerShell installation instructions. You'll have to restart your terminal session for Deno to be detected by yt-dlp.

<div class="page"/>

### macOS/Linux

On most Linux distros, yt-dlp can be installed via your system's package manager, but due to YouTube's ongoing efforts to stop unofficial clients from working, the version provided by your distro's repositories will likely be out-of-date. For this reason, it is recommended that you download yt-dlp directly from the developers. However, it's still useful to install it via your package manager as well to access its manpage via `man`, which will be relevant to this workshop.

Download the version of yt-dlp for your platform. You can do it from the command-line, make sure you are in your workshop directory:
```sh
# macOS
wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp_macos
# Linux
wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp_linux
```

To make it executable, run:
```sh
# macOS
chmod +x yt-dlp_macos
# Linux
chmod +x yt-dlp_linux
```

Test if it's executable with:
```sh
# macOS
./yt-dlp_macos --help
# Linux
./yt-dlp_linux --help
```

To run it without prefixing it with `./`, create a source script that defines a shell function called `ytdlp` (to avoid conflicting with the version installed via a package manager) to call it for you. On macOS:
```sh
ytdlp() { "$(pwd)/yt-dlp_macos" $@; }
```

On Linux:
```sh
ytdlp() { "$(pwd)/yt-dlp_linux" $@; }
```

<div class="page"/>

Call it something like `shortcut` and run `. shortcut` or `. ./shortcut` at the start of the shell session.

To install Deno, visit <https://docs.deno.com/runtime/getting_started/installation/> and follow the Shell installation instructions for your platform. You'll have to refresh your shell session for Deno to be detected by yt-dlp. You can either do this by starting a new shell session or running `exec $SHELL`.

## Installing FFmpeg

### Windows

You will need a tool to open 7-Zip archives, such as 7-Zip itself. Get it from here: <https://www.7-zip.org/>

Visit <https://ffmpeg.org/download.html>, click the blue Windows rectangle and click "Windows builds from gyan.dev". Under "latest git master branch build", download "ffmpeg-git-full.7z".

Locate and enter the "bin" folder in the archive. Extract `ffmpeg.exe` and `ffprobe.exe` to your workshop directory.

Open the command prompt and test if FFmpeg is executable with:
```
ffmpeg --help
```

"ffmpeg-git-essentials.7z" is not sufficient for this workshop due to the lack of an encoder (`libsvtav1`) used in it.

### macOS

Visit <https://ffmpeg.org/download.html>, click the grey macOS rectangle and click "Static builds for macOS 64-bit". Find the latest normal release, it should have a normal version number (X.Y.Z) and not a git commit hash
(ffmpeg-NNNNN-gXXXXXXXXXX), as the latter is for in-development snapshots. You'll have the option to download it as a zip archive or a 7-Zip archive. Repeat the same instructions for FFprobe.

Extract `ffmpeg` and `ffprobe` to your workshop directory.

To make them executable, run:
```sh
chmod +x ffmpeg ffprobe
```

Open the terminal and test if FFmpeg is executable with:
```
./ffmpeg --help
```

<div class="page"/>

To run it without prefixing it with `./`, append this to the `shortcut` script written for yt-dlp:
```sh
ffmpeg() { "$(pwd)/ffmpeg" $@; }
ffprobe() { "$(pwd)/ffprobe" $@; }
```

Re-run `. shortcut` or `. ./shortcut` to access the new functions.

### Debian/Ubuntu-Based Linux Distro

Install the FFmpeg package:
```sh
sudo apt install ffmpeg
```

Test if it's executable with:
```sh
ffmpeg --help
```
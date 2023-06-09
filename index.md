# My Articles

My articles about working with audio in Linux.

* [Extracting Audio & Subtitles From Movies](./extracting-audio-and-subtitles-from-movies.md)



# My Bash Scripts

* [CUE Splitter](https://github.com/ponomarevandrey/automation-scripts/blob/master/audio/cue-splitter.sh) - Splits FLAC, APE or WV using CUE sheet.  If the file is not FLAC, it also converts it to FLAC. The script also copies ID3v2 metadata from CUE sheet to each audio file.
* [Multiple `wv` To Multiple `flac`](https://github.com/ponomarevandrey/automation-scripts/blob/master/audio/multiple-wv-to-multiple-flac.sh) - Convert multiple WV files into multiple FLAC files


# Helpful Links

* [Advanced Multimedia on the Linux Command Line](https://avi.alkalay.net/2016/09/multimedia-linux-cli.html)



# Audio Editors

Audacity

* Install
  
  ```sh
  flatpak install flathub org.audacityteam.Audacity
  ```



# Audio File Management

## `kid3`

* [Doc](https://docs.kde.org/trunk5/en/kid3/kid3/kid3-cli.html)

Based on my experience for the past 5 years, during which I've tried all existing ID3 tagging software for Linux, kid3 is the best one. It supports both CLI (`kid3-cli`) and GUI.

`kid3` reads ID3 tags of any files, including MP3 and FLAC. Don't worry about underlying ID3 tag names, `kid3` handles everything nicely and universally.

Just an example of how to issue commands in non-interactive mode:
```shell
# write multiple genre tags

kid3-cli "/mnt/DE68EB6A64EB53BK/music_archive" \
  -c "select '/mnt/DE68EB6A64EB53BK/music_archive/DJ Q - Brandy & Coke VIP.mp3'" \
  -c "set genre[0] 'UK Garage'" \
  -c "set genre[1] '2-Step'" \
  -c "save"
  
# read multiple genre tags:

kid3-cli "/mnt/DE68EB6A64EB53BK/music_archive" \
  -c "get genre[0]"
  -c "get genre[1]"
```


## `ffmpeg`

**IMPORTANT**. DO NOT USE IT FOR EDITING ID3 TAGS. INSTEAD USE `kid3-cli`. It is way more convenient for handling metadata across all types of audio files.

* [How to install](https://linuxize.com/post/how-to-install-ffmpeg-on-ubuntu-18-04/)



### How-to

* **Convert `.wv` to `.flac`:**
  ```
  ffmpeg \
    -i audiofile.wv audiofile.flac
  ```
  
  Or use standard linux `sox` util: 
  ```
  sox audiofile.wv audiofile.flac
  ```

* **Convert `mp3` to `mp4` video with a static cover:** 
  ```
  ffmpeg \
    -loop 1 \
    -i image.jpg \
    -i audio.mp3 \
    -c:a copy \
    -c:v libx264 \
    -shortest out.mp4`
  ```

* **Record OS audio:**
  ```
  ffmpeg \
    -f pulse i default output.wav
  ```` 
  It captures all system's audio output (including browser) and writes to disk.

* **[How can I make ffmpeg be quieter/less verbose?](https://superuser.com/questions/326629/how-can-i-make-ffmpeg-be-quieter-less-verbose):**
  ```
  ffmpeg \
    -hide_banner \
    -loglevel error
  ```

* **[Split an audio file into multiple files?](https://unix.stackexchange.com/a/283547))**
  ```
  ffmpeg -i somefile.mp3 -f segment -segment_time 3 -c copy out%03d.mp3
  ```
  * `-segment_time` is the amount of time you want per each file (in seconds).
  * `%03d` is for a sequence, it means 3 digits, like `img001.pn`g, `img002.png`, etc.
  
  > **TIP** 1 hour = 3600 seconds


* **Converrt to lower bitrate**
  ```
  ffmpeg -i input.mp3 -codec:a libmp3lame -b:a 128k output.mp3
  ```


## `flac` 

* [Doc](https://xiph.org/flac/documentation_tools_flac.html) 



## `metaflac`

* [Doc](https://linux.die.net/man/1/metaflac)

---

Program to list, add, remove, or edit metadata in one or more FLAC files

* list flac metadata: `metaflac --list --block-number=2` (but the block number may be different)



## `shntool`

Main features:

* view and/or modify WAVE data and properties (split, join, convert, compare ...) 
* view detailed file information (not id3 tags, but detailed file data: length, props of WAV file)
* generate CUE sheets, split/join files based in CUE sheets



## `bpm-tools`

* [Doc](http://manpages.ubuntu.com/manpages/bionic/man1/bpm.1.html)




# Radio Scheduling & DJ Software

## IDJC

Similar to Proppfrexx. [IDJC](https://idjc.sourceforge.io/tour_main.html) is the best (feature rich) software of this type that exists for Linux. You can use it on Ubuntu server for automation of your online radio.



## `mixxx`

Similar to Traktor but open-source.

* Install:
  ```shell
  sudo add-apt-repository ppa:mixxx/mixxx

  sudo apt-get update

  sudo apt-get install mixxx
  ```

* Uninstall:
  ```shell
  sudo apt-get remove --autoremove mixxx
  ```



# DAWs

## Bitwig \[Commercial\]

Depending on extension of your bitwig sintallation file you will install it either with `flaptack` or `dpkg` package manager.

Regarding `dpkg`, install everything as usual.

* **Install/uninstall Bitwig with flatpack:**
  1. Install Flatpak package manager to be able to execute `.flatpak` file: `sudo apt install flatpak`
  2. Install Bitwig: `flatpak install "Bitwig Studio 4.3.flatpak"`
  3. **Uninstall Bitwig: `sudo apt remove bitwig-studio`
  4. Uninstall flatpack: `sudo apt remove flatpak`

* **Activation.** The doc for the crack instructs: 
  > Replace "bitwig.jar" from the FLARE folder to "/opt/bitwig-studio/bin/"

  You should put this file not in your Ubuntu profile's `.../douglasquaid/opt`, but in system-wide `/opt`, like this: `sudo cp ./bitwig.jar /opt/bitwig-studio/bin`

* **Deleting a Bitwig Activation file on my computer**
  1. `cd home/[username]/.BitwigStudio/`
  2. Find and delete any files called `.activation-1-1` or with other numbers
  3. Restart Bitwig Studio



## `ardour`

**IMPORTANT.** If you work a lot with MIDI, [Qtracktor](https://qtractor.org/qtractor-screenshots.html) has a more advanced MIDI featres than Ardour, prefer it.

[Ardour](https://ardour.org/) is a DAW. Open-Source replacement of Ableton/Cubase/... Good but heavily lacks many MIDI features.



# OS Audio Configuration

## `jack`

* [JackAudio Official website](https://jackaudio.org/downloads/)
* [Demystifying JACK – A Beginners Guide to Getting Started with JACK](https://linuxaudio.github.io/libremusicproduction/html/articles/demystifying-jack-%E2%80%93-beginners-guide-getting-started-jack.html)

---

* [Focusrite Scarlett 2i2 working flawlessly on Ubuntu with JACK](https://dragly.org/2014/01/12/focusrite-scarlett-2i2-flawlessly-working-on-ubuntu-with-jack/)
* Also check out `jack_mixer` (haven't explored it yet)



## `pipewire`

PulseAudio replacement to get read of delays.

Read [here](./pipewire.md)


# Misc

## `spek`

[Spek](http://spek.cc/) – Acoustic Spectrum Analyser


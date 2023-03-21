# Extracting Audio & Subtitles From Movies

# Audio Quality

In this tutorial `fmpeg` won't re-encode audio streams during extraction - they will be extracted as is, keeping the same bitrate and all other parameters, so the quality of your audio depends solely on how good the audio in your original video file.

Also, a few words about audio formats:

* `ac3` stores audio in *loosy* form. Usually it varies between 96kbps and 320kbps. Anyway, lossles format is bad if you plan to further process and chop the audio for sampling purposes.
* `dts` stores audio in *loosless* form.

Hence, if you need to have audio with the highest bitrate possible - extract `dts` stream(s).


  

## Extracting From `mkv` files

### Extracting Audio

1. **Check what audio channels exist in `.mkv` file.** 

   You can do this in two ways. Automatically with the help of `grep`:
   ```shell
   ffprobe \
     -hide_banner \
     -i movie_filename.mkv \
     2>&1 | \
   grep -i "audio"
   ```
   ...or manually, which is more combersome:
   ```shell
   ffmpeg \
     -hide_banner \
     -i movie_filename.mkv 
   ```
   ... now manually scroll searching for words "Audio". There should be at least one audio track.
   
   Depending on a particular `mkv` file, the output will look as follows:
   ```
       ...
       Stream #0:5(eng): Audio: dts (DTS), 48000 Hz, 5.1(side), fltp, 1536 kb/s # <--- .dts Audio Stream
       Metadata:
         title           : DTS 5.1 @ 1536 kbps |Оригинал|      
       Stream #0:6(rus): Subtitle: subrip                                       # <--- Subtitle Stream
       Metadata:
         title           : forced
       ...
   
   # or more concise, like this:
   
   Stream #0:1(rus): Audio: dts (DTS), 48000 Hz, 5.1(side), fltp, 1536 kb/s (default)
   Stream #0:2(rus): Audio: ac3, 48000 Hz, mono, fltp, 224 kb/s
   ```
  
2. **Using the info above, extract one or more separate audio streams (i.e. tracks).** Note, that you should set the output file extension similar to the stream's extention (`dts` in example above), otherwise you'll get an error. 

   If you need to extract multiple audio tracks at the same time, just add another `-map` flag. 
    
   ```sh
   ffmpeg \
     -hide_banner \
     -i movie_filename.mkv \
     -map 0:a:4 \
     -c copy movie_filename.dts
   ```
   * `a` means "audio stream"
   * `4` means "fourth *audio* stream". Notice: `4`th, not `5`th (explained below). 
  
   > **STREAMS NUMBERING** 
   >
   > Technically, according to `ffmpeg`'s output above (`Stream #0:5(eng): Audio: dts (DTS)`), you might think that you  need to extract the *fifth* stream (`0:a:5`) and if you really attempt to do this, you will end up with an error, because there is no fifth *audio* track. 
   >
   > Here's an example. Suppose after running `ffmpeg -i some_file.mkv` you've discovered that the file contains five  following streams:
   > ```
   > Stream #0 Video
   > Stream #1 Subtitles              # 0 - first subtitles stream (numbering always starts at 0) 
   > Stream #2 Subtitles              # 1 - second subtitle stream
   > Stream #3 Audio: dts             # 0 - first audio stram
   > Stream #4 Audio: ac3             # 1 - second audio stream
   > ```
   > So the numbering actually is not absolute, it is relative to each type of stream. **Yes, the streams numbering in `ffmpeg` output is rather misleading, so be cautious.**


3. **A Note On Extraction Of `dts` Files.** Unlike `ac3`, extracting lossless `dts` audio stream is a bit more complex. The initial steps are 100% the same as explained above, but we need to do a bit more work.

   Suppose we ended up with a `dts` file extracted from `mkv`. The `dts` file contains five separate audio tracks. You can play it in audio player and open for editing in Audacity, but usually it's much more convinient to split the `dts` into five separate `wav` track files. Let's do this.
   
   We will extract each `wav` file by setting up channel mappings and then assigning those mappings to each output file. To do this we need to know a few things: 1) what channels our `dts` audio file offers, 2) which channels to extract and 3) what the two letter abbreviation for the channel is within `ffmpeg`.

   ```
   # 1.Check out the DTS version of your file
   
   ffmpeg \
     -hide_banner
     -i your_movie_filname.dts
   ```
   This is the output of the command:
   ```
   ...
   Input #0, dts, from 'Иван Васильевич меняет профессию.BDRip1080p.dts':
     Duration: 01:30:02.47, start: 0.000000, bitrate: 1536 kb/s
     Stream #0:0: Audio: dts (DTS), 48000 Hz, 5.1(side), fltp, 1536 kb/s           # <--- !
   ```
   As we see the DTS version is 5.1.
   
   Now run [this script](https://github.com/ponomarevandrey/automation-scripts/blob/master/audio/dts-to-multiple-wav.sh) - it will do the main job.
   
   That's it!
   
   > ([source](https://randocity.com/2016/04/10/audio-tip-how-to-decode-5-1-dts-ac3-to-6-wav-files/#alternatives)) Note that there are a lot of different possible mappings for various types of audio input files. Since there are now many formats of soundtrack audio available such as Dolby Atmos, Dolby DTS, AC3 and various others, you should first determine the format of the input audio with `ffprobe` (see below) to better understand how to map and extract the audio.
   >
   > The channels available for possible extraction in `ffmpeg` include the following: `ffmpeg -layouts -hide_banner`
   >
   > **Using `ffprobe`**.
   >
     
   


### Extracting Subtitles

Most of the stuff was explained in section about extracting audio, so here I will show only code:


To extract subtitles (the subtitle stream for example):

```sh
# 1. Find all subtitles streams

ffmpeg \
  -hide_banner \
  -i movie_filename.mkv \
  2>&1 | \
grep -i "subtitle"


# 2. Choose the one you want to extract and issue the command

ffmpeg \
  -hide_banner \
  -i movie_filename.mkv \
  -map 0:s:0 \
  -c copy movie_filename_eng-sub.srt
```


# Sources

* [\[Stackoverflow\] Extract every audio and subtitles from a video with ffmpeg](https://stackoverflow.com/a/32925753/13156302)
* [Audio Tip: How to Decode 5.1 DTS / AC3 to 6 WAV Files](https://randocity.com/2016/04/10/audio-tip-how-to-decode-5-1-dts-ac3-to-6-wav-files/#alternatives)

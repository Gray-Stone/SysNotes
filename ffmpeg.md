# FFMPEG

Yes! this tool with millions of nobs

## Record webcam in shell script.

Some time when running a script that actually impact real life, it might be good to have a video recording.

```
ffmpeg -i /dev/video0 outfile.mkv
``` 

This will record the device of `/dev/video0` to file

You might want to specify the input format to get every the camera have (frame rate, etc)
[stack overflow link](https://stackoverflow.com/questions/47292785/recording-from-webcam-using-ffmpeg-at-high-framerate)

```
-input_format mjpeg
```

The default captured resolution is quite small, might want to specify it.

```
-video_size 1280x720
```

## Segmented output file 

`-f segment` is needed for all following segment to work

> segment_wrap *limit*:    
>
>    Wrap around segment index once it reaches limit.

> segment_format *format*: 
>
>    Override the inner container format, by default it is guessed by the filename extension.

> segment_time *time*:
>
>    Set segment duration to time, the value must be a duration specification. Default value is "2". See also the segment_times option.
>
>    Note that splitting may not be accurate, unless you force the reference stream key-frames at the given time. See the introductory notice and the examples below.

`-strftime 1` is needed if output file needs time date formatting

a common date time format
```
%Y-%m-%d_%H:%M:%ST%z
```

coulde even use the date format to create sub-dir and group them 

> strftime_mkdir
>
>   Used together with -strftime_mkdir, it will create all subdirectories which is expanded in filename.

```
ffmpeg -i in.nut -strftime 1 -strftime_mkdir 1 -hls_segment_filename '%Y%m%d/file-%Y%m%d-%s.ts' out.m3u8
```

example command 
```
ffmpeg -nostdin -nostats -video_size 1280x720 -i /dev/video0 -input_format mjpeg -f segment -segment_wrap 10 -segment_time $((60*5)) -strftime 1 ~/Downloads/ffmpeg/rec_%Y-%m-%d_%H:%M:%ST%z.mkv
```

## Small Detail for Scripter

`--nostats` will not show a rolling status bar when capturing, it will kept other info level logs. This will prevent your log file from this bash script being swamped with status update.

`--nostdin` will turn off ffmpeg's stdin reading. User could control the recording on the fly using keyboard, which is not wanted in scripts




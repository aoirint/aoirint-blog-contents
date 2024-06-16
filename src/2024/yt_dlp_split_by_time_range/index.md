# yt-dlp, 時間で分割してダウンロードする

## デモ用MP4ファイルの作成

デモ用にMP4ファイルを作成します。

```shell
$ ffmpeg -version
ffmpeg version 4.4.2-0ubuntu0.22.04.1 Copyright (c) 2000-2021 the FFmpeg developers
built with gcc 11 (Ubuntu 11.2.0-19ubuntu1)
configuration: --prefix=/usr --extra-version=0ubuntu0.22.04.1 --toolchain=hardened --libdir=/usr/lib/x86_64-linux-gnu --incdir=/usr/include/x86_64-linux-gnu --arch=amd64 --enable-gpl --disable-stripping --enable-gnutls --enable-ladspa --enable-libaom --enable-libass --enable-libbluray --enable-libbs2b --enable-libcaca --enable-libcdio --enable-libcodec2 --enable-libdav1d --enable-libflite --enable-libfontconfig --enable-libfreetype --enable-libfribidi --enable-libgme --enable-libgsm --enable-libjack --enable-libmp3lame --enable-libmysofa --enable-libopenjpeg --enable-libopenmpt --enable-libopus --enable-libpulse --enable-librabbitmq --enable-librubberband --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libsrt --enable-libssh --enable-libtheora --enable-libtwolame --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzimg --enable-libzmq --enable-libzvbi --enable-lv2 --enable-omx --enable-openal --enable-opencl --enable-opengl --enable-sdl2 --enable-pocketsphinx --enable-librsvg --enable-libmfx --enable-libdc1394 --enable-libdrm --enable-libiec61883 --enable-chromaprint --enable-frei0r --enable-libx264 --enable-shared
libavutil      56. 70.100 / 56. 70.100
libavcodec     58.134.100 / 58.134.100
libavformat    58. 76.100 / 58. 76.100
libavdevice    58. 13.100 / 58. 13.100
libavfilter     7.110.100 /  7.110.100
libswscale      5.  9.100 /  5.  9.100
libswresample   3.  9.100 /  3.  9.100
libpostproc    55.  9.100 / 55.  9.100
```

```shell
$ ffmpeg -hide_banner -t 60 -s 1920x1080 -f rawvideo -pix_fmt rgb24 -r 60 -i /dev/zero -c:v h264 -f lavfi -i anullsrc=channel_layout=stereo:sample_rate=44100 -c:a aac -shortest a.mp4
Input #0, rawvideo, from '/dev/zero':
  Duration: N/A, start: 0.000000, bitrate: 2985984 kb/s
  Stream #0:0: Video: rawvideo (RGB[24] / 0x18424752), rgb24, 1920x1080, 2985984 kb/s, 60 tbr, 60 tbn, 60 tbc
Input #1, lavfi, from 'anullsrc=channel_layout=stereo:sample_rate=44100':
  Duration: N/A, start: 0.000000, bitrate: 705 kb/s
  Stream #1:0: Audio: pcm_u8, 44100 Hz, stereo, u8, 705 kb/s
Stream mapping:
  Stream #0:0 -> #0:0 (rawvideo (native) -> h264 (libx264))
  Stream #1:0 -> #0:1 (pcm_u8 (native) -> aac (native))
Press [q] to stop, [?] for help
[libx264 @ 0x55f752a038c0] using cpu capabilities: MMX2 SSE2Fast SSSE3 SSE4.2 AVX FMA3 BMI2 AVX2
[libx264 @ 0x55f752a038c0] profile High 4:4:4 Predictive, level 4.2, 4:4:4, 8-bit
[libx264 @ 0x55f752a038c0] 264 - core 163 r3060 5db6aa6 - H.264/MPEG-4 AVC codec - Copyleft 2003-2021 - http://www.videolan.org/x264.html - options: cabac=1 ref=3 deblock=1:0:0 analyse=0x3:0x113 me=hex subme=7 psy=1 psy_rd=1.00:0.00 mixed_ref=1 me_range=16 chroma_me=1 trellis=1 8x8dct=1 cqm=0 deadzone=21,11 fast_pskip=1 chroma_qp_offset=4 threads=12 lookahead_threads=2 sliced_threads=0 nr=0 decimate=1 interlaced=0 bluray_compat=0 constrained_intra=0 bframes=3 b_pyramid=2 b_adapt=1 b_bias=0 direct=1 weightb=1 open_gop=0 weightp=2 keyint=250 keyint_min=25 scenecut=40 intra_refresh=0 rc_lookahead=40 rc=crf mbtree=1 crf=23.0 qcomp=0.60 qpmin=0 qpmax=69 qpstep=4 ip_ratio=1.40 aq=1:1.00
Output #0, mp4, to 'a.mp4':
  Metadata:
    encoder         : Lavf58.76.100
  Stream #0:0: Video: h264 (avc1 / 0x31637661), yuv444p(tv, progressive), 1920x1080, q=2-31, 60 fps, 15360 tbn
    Metadata:
      encoder         : Lavc58.134.100 libx264
    Side data:
      cpb: bitrate max/min/avg: 0/0/0 buffer size: 0 vbv_delay: N/A
  Stream #0:1: Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 128 kb/s
    Metadata:
      encoder         : Lavc58.134.100 aac
frame= 3600 fps=136 q=-1.0 Lsize=     368kB time=00:00:59.97 bitrate=  50.3kbits/s speed=2.26x
video:255kB audio:15kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 36.037613%
[libx264 @ 0x55f752a038c0] frame I:15    Avg QP: 9.20  size:   491
[libx264 @ 0x55f752a038c0] frame P:907   Avg QP:12.07  size:    77
[libx264 @ 0x55f752a038c0] frame B:2678  Avg QP:15.67  size:    69
[libx264 @ 0x55f752a038c0] consecutive B-frames:  0.8%  0.0%  0.1% 99.1%
[libx264 @ 0x55f752a038c0] mb I  I16..4:  0.0% 100.0%  0.0%
[libx264 @ 0x55f752a038c0] mb P  I16..4:  0.0%  0.0%  0.0%  P16..4:  0.0%  0.0%  0.0%  0.0%  0.0%    skip:100.0%
[libx264 @ 0x55f752a038c0] mb B  I16..4:  0.0%  0.0%  0.0%  B16..8:  0.0%  0.0%  0.0%  direct: 0.0%  skip:100.0%
[libx264 @ 0x55f752a038c0] 8x8 transform intra:100.0%
[libx264 @ 0x55f752a038c0] coded y,u,v intra: 0.0% 0.0% 0.0% inter: 0.0% 0.0% 0.0%
[libx264 @ 0x55f752a038c0] i16 v,h,dc,p:  0%  0% 100%  0%
[libx264 @ 0x55f752a038c0] i8 v,h,dc,ddl,ddr,vr,hd,vl,hu:  0%  0% 100%  0%  0%  0%  0%  0%  0%
[libx264 @ 0x55f752a038c0] Weighted P-Frames: Y:0.0% UV:0.0%
[libx264 @ 0x55f752a038c0] kb/s:34.79
[aac @ 0x55f752a05300] Qavg: 65536.000
```

- [shell - How to generate a 2hour-long blank video - Stack Overflow](https://stackoverflow.com/questions/11453082/how-to-generate-a-2hour-long-blank-video)
- [adding silent audio in ffmpeg - Stack Overflow](https://stackoverflow.com/questions/12368151/adding-silent-audio-in-ffmpeg)

```shell
$ sha256sum a.mp4
b495d430f500e9bf8afb3fcef4ba97e676ca4983051e6778a0167927ef11282b  a.mp4
```

## ffprobe

```shell
$ ffprobe -version
ffprobe version 4.4.2-0ubuntu0.22.04.1 Copyright (c) 2007-2021 the FFmpeg developers
built with gcc 11 (Ubuntu 11.2.0-19ubuntu1)
configuration: --prefix=/usr --extra-version=0ubuntu0.22.04.1 --toolchain=hardened --libdir=/usr/lib/x86_64-linux-gnu --incdir=/usr/include/x86_64-linux-gnu --arch=amd64 --enable-gpl --disable-stripping --enable-gnutls --enable-ladspa --enable-libaom --enable-libass --enable-libbluray --enable-libbs2b --enable-libcaca --enable-libcdio --enable-libcodec2 --enable-libdav1d --enable-libflite --enable-libfontconfig --enable-libfreetype --enable-libfribidi --enable-libgme --enable-libgsm --enable-libjack --enable-libmp3lame --enable-libmysofa --enable-libopenjpeg --enable-libopenmpt --enable-libopus --enable-libpulse --enable-librabbitmq --enable-librubberband --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libsrt --enable-libssh --enable-libtheora --enable-libtwolame --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzimg --enable-libzmq --enable-libzvbi --enable-lv2 --enable-omx --enable-openal --enable-opencl --enable-opengl --enable-sdl2 --enable-pocketsphinx --enable-librsvg --enable-libmfx --enable-libdc1394 --enable-libdrm --enable-libiec61883 --enable-chromaprint --enable-frei0r --enable-libx264 --enable-shared
libavutil      56. 70.100 / 56. 70.100
libavcodec     58.134.100 / 58.134.100
libavformat    58. 76.100 / 58. 76.100
libavdevice    58. 13.100 / 58. 13.100
libavfilter     7.110.100 /  7.110.100
libswscale      5.  9.100 /  5.  9.100
libswresample   3.  9.100 /  3.  9.100
libpostproc    55.  9.100 / 55.  9.100
```

```shell
$ ffprobe -hide_banner -select_streams v -show_entries frame -of json -v quiet a.mp4
{
    "frames": [
        {
            "media_type": "video",
            "stream_index": 0,
            "key_frame": 1,
            "pkt_pts": 0,
            "pkt_pts_time": "0.000000",
            "pkt_dts": 0,
            "pkt_dts_time": "0.000000",
            "best_effort_timestamp": 0,
            "best_effort_timestamp_time": "0.000000",
            "pkt_duration": 256,
            "pkt_duration_time": "0.016667",
            "pkt_pos": "48",
            "pkt_size": "1180",
            "width": 1920,
            "height": 1080,
            "pix_fmt": "yuv444p",
            "pict_type": "I",
            "coded_picture_number": 0,
            "display_picture_number": 0,
            "interlaced_frame": 0,
            "top_field_first": 0,
            "repeat_pict": 0,
            "chroma_location": "left",
            "side_data_list": [
                {
                    "side_data_type": "H.26[45] User Data Unregistered SEI message"
                }
            ]
        },
...
        {
            "media_type": "video",
            "stream_index": 0,
            "key_frame": 0,
            "pkt_pts": 921344,
            "pkt_pts_time": "59.983333",
            "best_effort_timestamp": 921344,
            "best_effort_timestamp_time": "59.983333",
            "pkt_duration": 256,
            "pkt_duration_time": "0.016667",
            "pkt_pos": "277157",
            "pkt_size": "75",
            "width": 1920,
            "height": 1080,
            "pix_fmt": "yuv444p",
            "pict_type": "P",
            "coded_picture_number": 3597,
            "display_picture_number": 0,
            "interlaced_frame": 0,
            "top_field_first": 0,
            "repeat_pict": 0,
            "chroma_location": "left"
        }
    ]
}
```

```shell
$ ffprobe -hide_banner -select_streams v -show_entries frame -of json -v quiet a.mp4 | jq .frames[0]
{
  "media_type": "video",
  "stream_index": 0,
  "key_frame": 1,
  "pkt_pts": 0,
  "pkt_pts_time": "0.000000",
  "pkt_dts": 0,
  "pkt_dts_time": "0.000000",
  "best_effort_timestamp": 0,
  "best_effort_timestamp_time": "0.000000",
  "pkt_duration": 256,
  "pkt_duration_time": "0.016667",
  "pkt_pos": "48",
  "pkt_size": "1180",
  "width": 1920,
  "height": 1080,
  "pix_fmt": "yuv444p",
  "pict_type": "I",
  "coded_picture_number": 0,
  "display_picture_number": 0,
  "interlaced_frame": 0,
  "top_field_first": 0,
  "repeat_pict": 0,
  "chroma_location": "left",
  "side_data_list": [
    {
      "side_data_type": "H.26[45] User Data Unregistered SEI message"
    }
  ]
}
```

```shell
$ ffprobe -hide_banner -select_streams v -show_entries frame -of json -v quiet a.mp4 | jq .frames[-1]
{
  "media_type": "video",
  "stream_index": 0,
  "key_frame": 0,
  "pkt_pts": 921344,
  "pkt_pts_time": "59.983333",
  "best_effort_timestamp": 921344,
  "best_effort_timestamp_time": "59.983333",
  "pkt_duration": 256,
  "pkt_duration_time": "0.016667",
  "pkt_pos": "277157",
  "pkt_size": "75",
  "width": 1920,
  "height": 1080,
  "pix_fmt": "yuv444p",
  "pict_type": "P",
  "coded_picture_number": 3597,
  "display_picture_number": 0,
  "interlaced_frame": 0,
  "top_field_first": 0,
  "repeat_pict": 0,
  "chroma_location": "left"
}
```

```shell
$ ffprobe -hide_banner -select_streams a -show_entries frame -of json -v quiet a.mp4
{
    "frames": [
        {
            "media_type": "audio",
            "stream_index": 1,
            "key_frame": 1,
            "pkt_pts": 0,
            "pkt_pts_time": "0.000000",
            "pkt_dts": 0,
            "pkt_dts_time": "0.000000",
            "best_effort_timestamp": 0,
            "best_effort_timestamp_time": "0.000000",
            "pkt_duration": 1024,
            "pkt_duration_time": "0.023220",
            "pkt_pos": "1391",
            "pkt_size": "6",
            "sample_fmt": "fltp",
            "nb_samples": 1024,
            "channels": 2,
            "channel_layout": "stereo"
        },
...
        {
            "media_type": "audio",
            "stream_index": 1,
            "key_frame": 1,
            "pkt_pts": 2644992,
            "pkt_pts_time": "59.977143",
            "pkt_dts": 2644992,
            "pkt_dts_time": "59.977143",
            "best_effort_timestamp": 2644992,
            "best_effort_timestamp_time": "59.977143",
            "pkt_duration": 1008,
            "pkt_duration_time": "0.022857",
            "pkt_pos": "277381",
            "pkt_size": "6",
            "sample_fmt": "fltp",
            "nb_samples": 1024,
            "channels": 2,
            "channel_layout": "stereo"
        }
    ]
}
```

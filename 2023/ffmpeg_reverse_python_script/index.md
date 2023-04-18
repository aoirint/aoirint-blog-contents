---
title: 'FFmpegで動画を逆再生化するPythonスクリプト'
date: '2023-04-18T21:40:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: 動画編集
tags:
  - 動画編集
  - FFmpeg
  - Python
---
# FFmpegで動画を逆再生化するPythonスクリプト

[MATVTool](https://github.com/aoirint/matvtoolpy)に組み込むかもしれませんが、いまのところ詳細な動作検証をするほど需要がないので、簡易的にここに置いておきます。

動画ファイルによっては、フレームの欠け、重複が発生したり、変換に失敗するかもしれません。

- Python 3.11.3
- FFmpeg 4.2.7-0ubuntu0.1
- Ubuntu 20.04 (WSL2)

```shell
python3 main.py input.mp4 output.mp4
```

```python
# License: CC0-1.0
import os
import subprocess
import tempfile
import re
import math


def get_duration_seconds(input_file: str) -> float:
    proc = subprocess.run(
        [
            'ffmpeg',
            '-hide_banner',
            '-i',
            input_file,
        ],
        stderr=subprocess.PIPE,
    )
    lines = proc.stderr.decode(encoding='utf-8').splitlines()

    # hh:mm:ss.ff
    duration_string = None
    for line in lines:
        m = re.match(r'^\s*Duration:\s(.+?),.*$', line)
        if m:
            duration_string = m.group(1)
            break

    assert duration_string is not None

    hours = int(duration_string[0:2])
    minutes = int(duration_string[3:5])
    seconds = int(duration_string[6:8])
    milliseconds = float('0.' + duration_string[9:11])

    return hours * 3600 + minutes * 60 + seconds + milliseconds


def parse_time(string: str) -> float:
    """
        string: HH:MM:SS.FF
    """
    hours = int(string[0:2])
    minutes = int(string[3:5])
    seconds = int(string[6:8])
    milliseconds = float('0.' + string[9:11])

    return hours * 3600 + minutes * 60 + seconds + milliseconds


def format_time(seconds: int) -> str:
    hours_minutes = seconds // 60
    hours = hours_minutes // 60
    minutes = hours_minutes - hours * 60
    local_seconds = seconds - hours_minutes * 60
    return f'{hours:02d}:{minutes:02d}:{local_seconds:02d}'


def main():
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument('input_file', type=str)
    parser.add_argument('output_file', type=str)
    parser.add_argument('--split_duration', type=int, default=10)
    args = parser.parse_args()

    input_file = args.input_file
    output_file = args.output_file
    split_duration = args.split_duration

    duration_seconds = math.ceil(get_duration_seconds(input_file=input_file))
    count = math.ceil(duration_seconds / split_duration)

    work_dir_obj = tempfile.TemporaryDirectory()
    work_dir = work_dir_obj.name

    part_output_file_list = []
    for index in range(count):
        start = index * split_duration
        end = start + split_duration

        start_string = format_time(start)
        end_string = format_time(end)
        print(index, start_string, end_string)

        part_output_file = os.path.join(work_dir, f'output{count-index}.mp4')

        subprocess.run([
            'ffmpeg',
            '-hide_banner',
            '-ss',
            start_string,
            '-to',
            end_string,
            '-i',
            'input.mp4',
            '-vf',
            'reverse',
            '-af',
            'areverse',
            part_output_file,
        ])
        part_output_file_list.append(part_output_file)
    
    list_file = os.path.join(work_dir, 'list.txt')
    with open(list_file, 'w', encoding='utf-8') as fp:
        for part_output_file in part_output_file_list:
            fp.write(f"file '{part_output_file}'\n")

    subprocess.run([
        'ffmpeg',
        '-hide_banner',
        '-f',
        'concat',
        '-safe',
        '0',
        '-i',
        list_file,
        '-c',
        'copy',
        output_file,
    ])


if __name__ == '__main__':
    main()
```

## 参考

- [How to Reverse a Video using FFmpeg - OTTVerse](https://ottverse.com/reverse-a-video-using-ffmpeg/)
- [映像と音声を逆再生にエンコードする | ニコラボ](https://nico-lab.net/encode_video_and_audio_in_reverse/)
- [FFMPEGで動画を逆再生して保存する方法 | 技術的特異点](https://tecsingularity.com/ffmpeg/reverse/)
- [Concatenate – FFmpeg](https://trac.ffmpeg.org/wiki/Concatenate)

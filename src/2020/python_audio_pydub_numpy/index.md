---
# moved from https://aoirint.hatenablog.com/entry/2020/03/14/180502
title: pydub numpy
date: '2020-03-14T18:05:02+09:00'
draft: false
channel: 技術ノート
category: 音声処理
tags:
  - 音声処理
  - Python
---
# pydub numpy

```python
from pydub import *
import numpy as np
import time

# https://own-search-and-study.xyz/2017/11/19/numpy%E3%81%AEarray%E3%81%8B%E3%82%89pydub%E3%81%AEaudiosegment%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B/

# https://maoudamashii.jokersounds.com/archives/song_kouichi_the_milky_way.html
path = 'song_kouichi_the_milky_way.m4a'

# https://github.com/jiaaro/pydub/blob/master/API.markdown#audiosegmentfrom_file
sound = AudioSegment.from_file(path, format='m4a') # give format explicitly
samples = np.array(sound.get_array_of_samples())

print(path)
print('Sample width (Num of bytes of a sample):', sound.sample_width)
print('Frame rate (Num of samples per second):', sound.frame_rate)
print('Channels (Stereo/Mono):', sound.channels)
print('Shape (Length):', samples.shape)
print('Type:', samples.dtype)
print('Min/Max:', samples.min(), samples.max())

output = AudioSegment(
    samples.astype('int32').tobytes(),
    sample_width=4,
    frame_rate=44100,
    channels=2,
)

ts = time.time()
output.export('output.m4a')
elapsed = time.time() - ts
print('Exported as m4a: %f s' % elapsed)

ts = time.time()
output.export('output.mp3')
elapsed = time.time() - ts
print('Exported as mp3: %f s' % elapsed)

ts = time.time()
output.export('output.wav')
elapsed = time.time() - ts
print('Exported as wav: %f s' % elapsed)

```

```plain
song_kouichi_the_milky_way.m4a
Sample width (Num of bytes of a sample): 2
Frame rate (Num of samples per second): 44100
Channels (Stereo/Mono): 2
Shape (Length): (22339584,)
Type: int16
Min/Max: -32768 32767
Exported as m4a: 6.288757 s
Exported as mp3: 6.194534 s
Exported as wav: 6.064215 s
```

- 出力時間はフォーマットによって変わらない（誤差の範囲）

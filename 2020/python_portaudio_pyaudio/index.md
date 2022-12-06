---
# moved from https://aoirint.hatenablog.com/entry/2020/03/19/101700
title: PortAudio, pyaudio
date: '2020-03-19T10:17:00+09:00'
draft: false
channel: 技術ノート
category: Python
tags:
- Python
- 音声処理
---
# PortAudio, pyaudio

- [PortAudio: PortAudio API Overview](http://portaudio.com/docs/v19-doxydocs/api_overview.html)

---

- Host API
  - ALSA (Ubuntu)
  - Core Audio (Mac)
- Device
  - Speaker
  - Mic
- Stream
  - sample rate (num of samples per second)
  - sample format (num of bytes of a sample, integer or float)

---

- [PyAudio Documentation — PyAudio 0.2.11 documentation](https://people.csail.mit.edu/hubert/pyaudio/docs/)
- [Audio device detection w/ pyaudio](https://gist.github.com/mansam/9332445)

```sh
brew install portaudio
pip3 install pyaudio
```

- [macOSにpyaudioをインストールする - Qiita](https://qiita.com/mayfair/items/abb59ebf503cc294a581)

### Check Host APIs

```python
import pyaudio

pa = pyaudio.PyAudio()

api_count = pa.get_host_api_count()
print('Host API:', api_count)

for i in range(api_count):
    api_info = pa.get_host_api_info_by_index(i)
    print(api_info)

    device_count = api_info['deviceCount']
    for j in range(device_count):
        device_info = pa.get_device_info_by_host_api_device_index(i, j)
        print(device_info)

pa.terminate()
```

```
Host API: 1
{'index': 0, 'structVersion': 1, 'type': 5, 'name': 'Core Audio', 'deviceCount': 2, 'defaultInputDevice': 0, 'defaultOutputDevice': 1}
{'index': 0, 'structVersion': 2, 'name': 'Built-in Microphone', 'hostApi': 0, 'maxInputChannels': 2, 'maxOutputChannels': 0, 'defaultLowInputLatency': 0.0029478458049886623, 'defaultLowOutputLatency': 0.01, 'defaultHighInputLatency': 0.01310657596371882, 'defaultHighOutputLatency': 0.1, 'defaultSampleRate': 44100.0}
{'index': 1, 'structVersion': 2, 'name': 'Built-in Output', 'hostApi': 0, 'maxInputChannels': 0, 'maxOutputChannels': 2, 'defaultLowInputLatency': 0.01, 'defaultLowOutputLatency': 0.007800453514739229, 'defaultHighInputLatency': 0.1, 'defaultHighOutputLatency': 0.017959183673469388, 'defaultSampleRate': 44100.0}
```


### Check Devices

```python
import pyaudio

pa = pyaudio.PyAudio()

device_count = pa.get_device_count()
print('Device:', device_count)

for i in range(device_count):
    device_info = pa.get_device_info_by_index(i)
    print(device_info)

pa.terminate()
```

```
Device: 2
{'index': 0, 'structVersion': 2, 'name': 'Built-in Microphone', 'hostApi': 0, 'maxInputChannels': 2, 'maxOutputChannels': 0, 'defaultLowInputLatency': 0.0029478458049886623, 'defaultLowOutputLatency': 0.01, 'defaultHighInputLatency': 0.01310657596371882, 'defaultHighOutputLatency': 0.1, 'defaultSampleRate': 44100.0}
{'index': 1, 'structVersion': 2, 'name': 'Built-in Output', 'hostApi': 0, 'maxInputChannels': 0, 'maxOutputChannels': 2, 'defaultLowInputLatency': 0.01, 'defaultLowOutputLatency': 0.007800453514739229, 'defaultHighInputLatency': 0.1, 'defaultHighOutputLatency': 0.017959183673469388, 'defaultSampleRate': 44100.0}
```

### Stream

※ waveだけ鳴らせればいい場合は`wave`モジュールを使ってください（[参照](https://people.csail.mit.edu/hubert/pyaudio/docs/#example-blocking-mode-audio-i-o)）

```sh
pip3 install pydub
brew install ffmpeg
```

```python
import pyaudio
from pydub import AudioSegment

pa = pyaudio.PyAudio()

device_index = 1 # Output device
device_info = pa.get_device_info_by_index(device_index)
print(device_info)

# https://maoudamashii.jokersounds.com/archives/song_kouichi_the_milky_way.html
sound_path = 'song_kouichi_the_milky_way.m4a'
# https://github.com/jiaaro/pydub/blob/master/API.markdown#audiosegmentfrom_file
sound = AudioSegment.from_file(sound_path, format='m4a') # give format explicitly

# See here: https://people.csail.mit.edu/hubert/pyaudio/docs/#pasampleformat
sample_width = sound.sample_width # 2
format = pyaudio.get_format_from_width(sample_width) # 8
sample_rate = sound.frame_rate # 44100
channels = sound.channels # 2 (Stereo)

print(format, '(%d)' % sample_width)
print(sample_rate)
print(channels)

stream = pa.open(
    output_device_index=device_index,
    rate=sample_rate,
    channels=channels,
    format=format,
    output=True,
)

samples = sound.raw_data # bytes of samples
# samples = bytes(sound.get_array_of_samples()) # Equivalent

# block; cannot interrupt with Ctrl+C.
# For non-blocking, see here: https://people.csail.mit.edu/hubert/pyaudio/docs/#example-callback-mode-audio-i-o
stream.write(samples)

stream.stop_stream()
stream.close()

pa.terminate()
```

- [Non-blocking usage (Example: Callback Mode Audio I/O @ PyAudio Documentation — PyAudio 0.2.11 documentation)](https://people.csail.mit.edu/hubert/pyaudio/docs/#example-callback-mode-audio-i-o)
- [pydub AudioSegment.from_file (pydub/API.markdown at master · jiaaro/pydub)](https://github.com/jiaaro/pydub/blob/master/API.markdown#audiosegmentfrom_file)

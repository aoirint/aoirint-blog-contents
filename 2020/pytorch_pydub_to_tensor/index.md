---
# moved from https://aoirint.hatenablog.com/entry/2020/03/26/111041
title: 'PyTorch pydub.AudioSegmentをtorch.Tensorに変換する'
date: '2020-03-26T11:10:41+09:00'
draft: false
channel: 技術ノート
category: 音声処理
tags:
  - 音声処理
  - Python
  - pydub
  - PyTorch
  - TorchAudio
---
# PyTorch pydub.AudioSegmentをtorch.Tensorに変換する

```python
import numpy as np
import torch
import torchaudio
import torchaudio.transforms as T

'''
in: pydub.AudioSegment
out: torch.Tensor (float32)
'''
def to_tensor(audio):
    sample_width = audio.sample_width
    sample_bits = 8 * sample_width
    sample_max_int = 2 ** sample_bits
    sample_channels = audio.channels

    samples = np.asarray(audio.get_array_of_samples())
    samples = samples.reshape((-1, 2)).transpose((1, 0)) # LRLR -> Channel, Samples

    samples = samples.astype('f') / sample_max_int

    samples = torch.from_numpy(samples).type(torch.float32)

    return samples
```

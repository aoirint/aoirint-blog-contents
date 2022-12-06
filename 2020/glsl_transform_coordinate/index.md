---
# moved from https://aoirint.hatenablog.com/entry/2020/03/23/011222
title: GLSL 座標変換 テンプレート
date: '2020-03-23T01:12:22+09:00'
draft: true
channel: 技術ノート
category: GLSL
tags:
- GLSL
- OpenGL
---
# GLSL 座標変換 テンプレート

### Vertex shader

```glsl
#version 410 core

in vec3 vPosition;
out vec2 vTextureCoord;

void main(void) {
    vTextureCoord = vec2((vPosition.x + 1.0) / 2, (vPosition.y + 1.0) / 2);

    float x = vPosition.x;
    float y = vPosition.y;

    gl_Position = vec4(x, y, vPosition.z, 1.0);
}
```

### Fragment shader

```glsl
#version 410 core

uniform sampler2D vTexture;
in vec2 vTextureCoord;
out vec4 flagColor;

void main(void) {
    flagColor = texture(vTexture, vTextureCoord).rgba;
}
```

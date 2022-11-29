---
# moved from https://aoirint.hatenablog.com/entry/2020/03/23/011157
title: OpenGL 2.1/4.1（GLFW）でOpenCVの画像を表示する（Python, Mac）
date: '2020-03-23 01:11:57'
draft: false
channel: 技術ノート
category: 画像処理
tags:
  - 画像処理
  - OpenCV
  - OpenGL
  - 'Computer Vision'
  - Python
---
# OpenGL 2.1/4.1（GLFW）でOpenCVの画像を表示する（Python, Mac）

```sh
pip3 install PyOpenGL glfw
```

モジュールバージョン
```
PyOpenGL==3.1.5
glfw==1.11.0
```

システムのOpenGLバージョン
```sh
Vendor : b'Intel Inc.'
GPU : b'Intel Iris OpenGL Engine'
OpenGL version : b'4.1 INTEL-14.4.23'
```

### OpenGL 2.1

- [mcfletch/pyopengl: Repository for the PyOpenGL Project](https://github.com/mcfletch/pyopengl)
- [Python3で始めるOpenGL4 - CodeLabo](https://codelabo.com/posts/20200228175104)
- [Python GLFWでOpenGLバージョン指定とウィンドウ表示 - CodeLabo](https://codelabo.com/posts/20200228180254)
- [PythonでOpenCVの画像をOpenGLで表示する - Qiita](https://qiita.com/a2kiti/items/39eba7616036fdd6fd36)

```sh
Vendor : b'Intel Inc.'
GPU : b'Intel Iris OpenGL Engine'
OpenGL version : b'2.1 INTEL-14.4.23'
```

#### glDrawPixels

```python
import cv2
from OpenGL.GL import *
import glfw

if __name__ == '__main__':
    img = cv2.imread('lena.png', 1)
    img_gl = cv2.cvtColor(cv2.flip(img, 0), cv2.COLOR_BGR2RGB)

    glfw.init()

    # These parameters need to be changed according to your environment.
    # Mac: https://support.apple.com/ja-jp/HT202823
    # glfw.window_hint(glfw.CONTEXT_VERSION_MAJOR, 4)
    # glfw.window_hint(glfw.CONTEXT_VERSION_MINOR, 1)
    # glfw.window_hint(glfw.OPENGL_FORWARD_COMPAT, True)
    # glfw.window_hint(glfw.OPENGL_PROFILE, glfw.OPENGL_CORE_PROFILE)

    width = img.shape[1]
    height = img.shape[0]
    window = glfw.create_window(width, height, 'Lena', None, None)
    glfw.make_context_current(window)

    print('Vendor :', glGetString(GL_VENDOR))
    print('GPU :', glGetString(GL_RENDERER))
    print('OpenGL version :', glGetString(GL_VERSION))

    while not glfw.window_should_close(window):
        glClearColor(0, 0, 0, 1)
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

        glDrawPixels(width, height, GL_RGB, GL_UNSIGNED_BYTE, img_gl)

        glfw.swap_buffers(window)
        glfw.poll_events()

    glfw.destroy_window(window)
    glfw.terminate()
```

#### Texture

```python
import cv2
from OpenGL.GL import *
import glfw

if __name__ == '__main__':
    img = cv2.imread('lena.png', 1)
    img_gl = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

    glfw.init()

    # These parameters need to be changed according to your environment.
    # Mac: https://support.apple.com/ja-jp/HT202823
    # glfw.window_hint(glfw.CONTEXT_VERSION_MAJOR, 4)
    # glfw.window_hint(glfw.CONTEXT_VERSION_MINOR, 1)
    # glfw.window_hint(glfw.OPENGL_FORWARD_COMPAT, True)
    # glfw.window_hint(glfw.OPENGL_PROFILE, glfw.OPENGL_CORE_PROFILE)

    window = glfw.create_window(256, 256, 'Lena', None, None)
    glfw.make_context_current(window)

    print('Vendor :', glGetString(GL_VENDOR))
    print('GPU :', glGetString(GL_RENDERER))
    print('OpenGL version :', glGetString(GL_VERSION))

    height, width = img.shape[:2]
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, img_gl)

    while not glfw.window_should_close(window):
        glClearColor(0, 0, 0, 1)
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

        glEnable(GL_TEXTURE_2D)
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

        glBegin(GL_QUADS)
        glTexCoord2d(0.0, 1.0)
        glVertex3d(-1.0, -1.0, 0.0)
        glTexCoord2d(1.0, 1.0)
        glVertex3d(1.0, -1.0, 0.0)
        glTexCoord2d(1.0, 0.0)
        glVertex3d(1.0, 1.0, 0.0)
        glTexCoord2d(0.0, 0.0)
        glVertex3d(-1.0, 1.0, 0.0)
        glEnd()

        glfw.swap_buffers(window)
        glfw.poll_events()

    glfw.destroy_window(window)
    glfw.terminate()
```


### OpenGL 4.1 / GLSL

- [PythonでVAOによるGLSLシェーダープログラミング！ - CodeLabo](https://codelabo.com/posts/20200228182137)
- [Suspected fragment shader problem, No color, OpenGL 4, GLFW 3, GLEW - OpenGL / OpenGL: Basic Coding - Khronos Forums](https://community.khronos.org/t/suspected-fragment-shader-problem-no-color-opengl-4-glfw-3-glew/70399)
- [床井研究室 - 第２回 テクスチャの割り当て](marina.sys.wakayama-u.ac.jp/~tokoi/?date=20040914)


```sh
Vendor : b'Intel Inc.'
GPU : b'Intel Iris OpenGL Engine'
OpenGL version : b'4.1 INTEL-14.4.23'
```

```python
import sys
from OpenGL.GL import *
import glfw
import numpy as np
import cv2


vertex_shader_text = '''
#version 410 core

in vec3 vPosition;
out vec2 vTextureCoord;

void main(void) {
    vTextureCoord = vec2((vPosition.x + 1.0) / 2, (vPosition.y + 1.0) / 2);
    gl_Position = vec4(vPosition, 1.0);
}
'''

fragment_shader_text = '''
#version 410 core

uniform sampler2D vTexture;
in vec2 vTextureCoord;
out vec4 flagColor;

void main(void) {
    flagColor = texture(vTexture, vTextureCoord).rgba;
}
'''

def init_context():
    print('Initializing context..')
    glfw.init()

    glfw.window_hint(glfw.CONTEXT_VERSION_MAJOR, 4)
    glfw.window_hint(glfw.CONTEXT_VERSION_MINOR, 1)
    glfw.window_hint(glfw.OPENGL_FORWARD_COMPAT, True)
    glfw.window_hint(glfw.OPENGL_PROFILE, glfw.OPENGL_CORE_PROFILE)

    global window
    window = glfw.create_window(512, 512, __file__, None, None)
    glfw.make_context_current(window)

    print('Vendor :', glGetString(GL_VENDOR))
    print('GPU :', glGetString(GL_RENDERER))
    print('OpenGL version :', glGetString(GL_VERSION))

def init_texture():
    print('Initializing texture..')

    img = cv2.imread('lena.png', 1)
    img_gl = cv2.cvtColor(cv2.flip(img, 0), cv2.COLOR_BGR2RGB)

    global width, height
    height, width = img.shape[:2]

    global texture
    texture = glGenTextures(1)

    glActiveTexture(GL_TEXTURE0)
    glBindTexture(GL_TEXTURE_2D, texture)

    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)
    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

    glPixelStorei(GL_UNPACK_ALIGNMENT, 1)
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, img_gl)

def init_shader():
    print('Initializing shader..')

    global vertex_shader
    vertex_shader = glCreateShader(GL_VERTEX_SHADER)
    glShaderSource(vertex_shader, vertex_shader_text)
    glCompileShader(vertex_shader)
    if not glGetShaderiv(vertex_shader, GL_COMPILE_STATUS):
        print('Vertex shader is not OK')
        print(glGetShaderInfoLog(vertex_shader))
        sys.exit(1)
    else:
        print('Vertex shader is OK')

    global fragment_shader
    fragment_shader = glCreateShader(GL_FRAGMENT_SHADER)
    glShaderSource(fragment_shader, fragment_shader_text)
    glCompileShader(fragment_shader)
    if not glGetShaderiv(fragment_shader, GL_COMPILE_STATUS):
        print('Fragment shader is not OK')
        print(glGetShaderInfoLog(fragment_shader))
        sys.exit(1)
    else:
        print('Fragment shader is OK')

    global program
    program = glCreateProgram()
    glAttachShader(program, vertex_shader)
    glDeleteShader(vertex_shader)
    glAttachShader(program, fragment_shader)
    glDeleteShader(fragment_shader)
    glLinkProgram(program)
    if not glGetProgramiv(program, GL_LINK_STATUS):
        print('Shader program is not OK')
        print(glGetProgramInfoLog(program))
        sys.exit(1)
    else:
        print('Shader program is OK')

def init_vao():
    print('Initializing vao..')

    # anti-clockwise
    vertices = np.array([
        -1.0, -1.0, 0.0, # left bottom
        1.0, -1.0, 0.0, # right bottom
        -1.0, 1.0, 0.0, # left top

        -1.0, 1.0, 0.0, # left top
        1.0, -1.0, 0.0, # right bottom
        1.0, 1.0, 0.0, # right top
    ], dtype=np.float32)

    global vertex_vbo
    vertex_vbo = glGenBuffers(1)

    global vertex_vao
    vertex_vao = glGenVertexArrays(1)
    glBindVertexArray(vertex_vao)

    glEnableVertexAttribArray(0)

    glBindBuffer(GL_ARRAY_BUFFER, vertex_vbo)
    glBufferData(GL_ARRAY_BUFFER, vertices.nbytes, vertices, GL_STATIC_DRAW)
    glVertexAttribPointer(0, 3, GL_FLOAT, False, 0, None)

    glBindVertexArray(0)

def render():
    glUseProgram(program)

    glUniform1i(glGetUniformLocation(program, 'vTexture'), 0)

    glBindVertexArray(vertex_vao)
    glDrawArrays(GL_TRIANGLES, 0, 6)
    glBindVertexArray(0)

if __name__ == '__main__':
    init_context()
    init_texture()
    init_shader()
    init_vao()

    print('Start rendering..')
    while not glfw.window_should_close(window):
        glClearColor(0, 0, 0, 1)
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

        render()

        glfw.swap_buffers(window)
        glfw.poll_events()

    glfw.destroy_window(window)
    glfw.terminate()

```

- [PyOpenGL Documentation](http://pyopengl.sourceforge.net/documentation/)
- [PyOpenGL · PyPI](https://pypi.org/project/PyOpenGL/)
- [glfw · PyPI](https://pypi.org/project/glfw/)

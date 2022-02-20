---
# moved from https://aoirint.hatenablog.com/entry/2018/12/15/105618
title: 'json, bson, sqlite3 IOの実験メモ'
date: '2018-12-15 10:56:18'
draft: false
channel: 技術ノート
category: Python
tags:
  - 'Python'
---
# json, bson, sqlite3 IOの実験メモ

- Windows 10
- Python 3.6.6
- bson==0.5.7
- SSD

## JSON

```python
import json
import random
import time

file = 'test.tmp'
N = 500

# Generate
start = time.time()
entries = []
for i in range(N):
    title = ''.join([ chr(random.randint(ord('あ'), ord('ん')+1)) for i in range(32) ])
    body = ''.join([ chr(random.randint(ord('あ'), ord('ん')+1)) for i in range(30000) ])
    entries.append({
        'id': i,
        'title': title,
        'body': body,
    })

data = {
    'entries': entries,
}

end = time.time()
print('Generate: %.1fs' % (end - start, ))
```

```
Generate: 38.6s
```

```python
import os

# Write
start = time.time()
with open(file, 'w', encoding='utf-8') as fp:
    json.dump(data, fp, ensure_ascii=False)

end = time.time()
print('Write: %.3fs' % (end - start, ))

print('%.2fMB' % (os.path.getsize(file)/(1024**2), ))

# Read
start = time.time()
with open(file, 'r', encoding='utf-8') as fp:
    data = json.load(fp)

end = time.time()
print('Read: %.3fs' % (end - start, ))
```

```
Write: 0.252s
42.98MB
Read: 0.214s
```

## BSON

```shell
pip install bson
```

```python
import bson

# Write
start = time.time()
s = bson.dumps(data)
print('%.2fMB' % (len(s)/(1024**2), ))

dumpEnd = time.time()

with open(file, 'wb') as fp:
    fp.write(s)

end = time.time()

print('Write: %.3fs (Dump: %.3fs)' % (end - start, dumpEnd - start))


# Read
start = time.time()

with open(file, 'rb') as fp:
    s = fp.read()

loadStart = time.time()

data = bson.loads(s)
end = time.time()

print('Read: %.3fs (Load: %.3fs)' % (end - start, end - loadStart))
```

```
42.98MB
Write: 0.830s (Dump: 0.638s)
Read: 0.156s (Load: 0.095s)
```

## SQLite3

```python
import sqlite3

if os.path.exists(file):
    os.unlink(file)

start = time.time()
with sqlite3.connect(file) as db:
    cur = db.cursor()

    cur.execute('CREATE TABLE entries(id INTEGER AUTO INCREMENT, title TEXT, body TEXT)')
    for entry in data['entries']:
        cur.execute('INSERT INTO entries VALUES(?,?,?)', (entry['id'], entry['title'], entry['body'], ))
    
end = time.time()

print('Write: %.3fs' % (end - start, ))

start = time.time()
with sqlite3.connect(file) as db:
    cur = db.cursor()

    results = []
    for row in cur.execute('SELECT * FROM entries'):
        results.append(row)

end = time.time()

print('Read: %.3fs' % (end - start, ))


print('%.2fMB' % (os.path.getsize(file)/(1024**2), ))

db.close()
```

```
Write: 1.093s
Read: 0.243s
43.22MB
```

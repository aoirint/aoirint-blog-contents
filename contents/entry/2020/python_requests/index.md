---
canonical_url: ./
title: 'Python Requests'
# og_image:
# twitter_card: summary_large_image
og_description: 'Python Requests'
date: '2020-09-28 11:30:00'
draft: false
category: Python
tags:
  - Python
  - Requests
---
# Python Requests

```python
ses = requests.Session()
```

## Headers
```python
ses.headers.update({
})
```

## Copy cookies from Selenium
```python
ses.cookies.clear()
for cookie in driver.get_cookies():
    rc = requests.cookies.create_cookie(domain=cookie['domain'], name=cookie['name'], value=cookie['value'])
    ses.cookies.set_cookie(rc)
```

## Download a file
```python
file_url: str
dest_path: str

with tempfile.NamedTemporaryFile() as fp:
    with ses.get(file_url, stream=True) as r:
        r.raise_for_status()

        for chunk in r.iter_content(chunk_size=8192):
            fp.write(chunk)

    fp.flush()
    shutil.copy(fp.name, dest_path)
```

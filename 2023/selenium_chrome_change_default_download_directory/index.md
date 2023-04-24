---
title: 'Selenium デフォルトダウンロードディレクトリを変更する（Chrome, Python）'
date: '2023-04-24T16:00:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Selenium
tags:
  - Selenium
  - ChromeDriver
  - Python
---
# Selenium デフォルトダウンロードディレクトリを変更する（Chrome, Python）

- Selenium 4.9.0
- Chrome 112
- ChromeDriver 112.0.5615.49

```python
from selenium.webdriver import (
    Chrome,
    ChromeOptions,
)


download_dir = "./downloads"
os.makedirs(download_dir, exist_ok=True)

options = ChromeOptions()
options.add_experimental_option("prefs", {
    "profile.default_content_settings.popups": 0,
    "download.default_directory": os.path.realpath(download_dir),
    "download.prompt_for_download": False,
    "download.directory_upgrade": True,
})

driver = Chrome(
    options=options,
)
```

- [python - How to change download directory location path in Selenium using Chrome? - Stack Overflow](https://stackoverflow.com/questions/71716460/how-to-change-download-directory-location-path-in-selenium-using-chrome)

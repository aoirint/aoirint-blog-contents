---
title: 'Selenium HTTPリクエストのURLを記録する（Chrome, Python）'
date: '2023-04-24T16:30:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Selenium
tags:
  - Selenium
  - ChromeDriver
  - Python
---
# Selenium HTTPリクエストのURLを記録する（Chrome, Python）

- Selenium 4.9.0
- Chrome 112
- ChromeDriver 112.0.5615.49
- Python 3.11

```python
import time
import json
from selenium.webdriver import (
    Chrome,
    DesiredCapabilities,
)


desired_capabilities = DesiredCapabilities.CHROME
desired_capabilities["goog:loggingPrefs"] = {
    "performance": "ALL",
}

driver = Chrome(
    desired_capabilities=desired_capabilities,
)
driver.implicitly_wait(5)

driver.get("https://www.google.com/")

known_url_set = set()

while True:
    performance_log_entries = driver.get_log("performance")

    for log_entry in performance_log_entries:
        log_message = json.loads(log_entry.get("message", "{}")).get("message", {})

        method = log_message.get("method")
        params = log_message.get("params", {})

        if method == "Network.responseReceived":
            response = params.get("response", {})
            url = response.get("url")

            if url in known_url_set:
                continue
            known_url_set.add(url)

            print(url)

    time.sleep(1)
```

- [Python+SeleniumでChromeデベロッパーツールのNetworkタブ相当の情報を取得する - Qiita](https://qiita.com/nextpenguin/items/4827fb1da062581518e7)
- [java - Using Selenium how to get network request - Stack Overflow](https://stackoverflow.com/questions/45847035/using-selenium-how-to-get-network-request)

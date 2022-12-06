---
title: 'docker-composeによるPython + Selenium環境'
# og_image:
# twitter_card: summary_large_image
og_description: 'docker-composeでPython + Seleniumを使える環境を整備する'
date: '2020-09-28T11:00:00+09:00'
updated: '2022-08-06T21:30:00+09:00'
draft: false
channel: 技術ノート
category: Python
tags:
  - Docker
  - Docker Compose
  - Python
  - Selenium
---
# docker-composeによるPython + Selenium環境

## 2022-08-06 追記

Seleniumのバージョンが上がって一部非推奨化したりしたので、そちらに対応した版を作成しました。

- リポジトリ: [https://github.com/aoirint/compose-selenium-python-template](https://github.com/aoirint/compose-selenium-python-template)

---

## 2020-09-28

### docker-compose.yml

```yaml
version: '3.8'
services:
  app:
    build: ./app/
    entrypoint: [ "wait-for-it", "selenium:4444", "--", "python3", "/code/main.py" ]
    volumes:
      - ./work:/work
    environment:
      SELENIUM_URL: http://selenium:4444/wd/hub
    depends_on:
      - selenium
  selenium:
    image: selenium/standalone-chrome
    volumes:
      - /dev/shm:/dev/shm
```

### app/Dockerfile

```dockerfile
FROM python:3

WORKDIR /work

RUN apt update && apt install -y \
  wait-for-it

ADD requirements.txt /tmp/
RUN pip3 install -r /tmp/requirements.txt

ADD code/ /code
```

### app/requirements.txt

```requirements
requests >= 2.24.0
selenium
```

### app/code/main.py

```python
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

selenium_url = os.environ['SELENIUM_URL']
website_url: str = None

driver = webdriver.Remote(
    command_executor=selenium_url,
    desired_capabilities=DesiredCapabilities.CHROME,
)

driver.get(website_url)
```

---
title: はてなブログから記事を移しました
date: '2022-02-22 20:40:00'
draft: false
channel: 技術ノート
category: アナウンス
tags:
  - アナウンス
  - Blogging
---
# はてなブログから記事を移しました

新しい記事 blog.aoirint.com へ、
古い記事は aoirint.hatenablog.com に残っていましたが、
blog.aoirint.com に記事を移しました。

古い記事にはリダイレクトと移動先へのリンクを設置しています。

- <https://github.com/x-motemen/blogsync>

blogsyncで記事を取得したあと、以下のようなスクリプトでfrontmatterを変換しました。

以前の移行時の作業ミスでMarkdown記事がHTMLになってしまっていたので、
手動でMarkdownに書き戻しました。

```python
import glob
import yaml
from datetime import datetime
from pathlib import Path
import frontmatter as FM

for path in glob.glob('**/*.md', recursive=True):
  if path.startswith('output'):
    continue

  print(path)
  with open(path, 'r') as fp:
    frontmatter = FM.load(fp)
    # docs = yaml.safe_load_all(fp)
    # frontmatter = next(docs)
    # body = next(docs)

  body = frontmatter.content
  print(frontmatter)
  print(body)

  title = frontmatter['Title']
  if title.startswith('（移動済）'):
    print(f'Skipped: {title}')
    continue

  print(f'Processing: {title}')

  original_url = frontmatter['URL']
  date: datetime = frontmatter['Date']
  draft = frontmatter.get('Draft', False)

  tags = frontmatter.get('Category')
  category = tags[0] if tags else None

  new_frontmatter = {
    'title': title,
    'date': date.strftime('%Y-%m-%d %H:%M:%S'),
    'draft': draft,
    'channel': '技術ノート',
  }
  if category:
    new_frontmatter['category'] = category
  if tags:
    new_frontmatter['tags'] = tags

  output = yaml.dump(new_frontmatter, default_flow_style=False, sort_keys=False, allow_unicode=True)
  output_lines = output.split('\n')
  output_lines.insert(0, f'---')
  output_lines.insert(1, f'# moved from {original_url}')
  output_lines.insert(-1, f'---')
  output = '\n'.join(output_lines)
  output += f'# {title}\n\n' + body + '\n'

  dest = Path('output', path)
  dest.parent.mkdir(parents=True, exist_ok=True)

  with open(dest, 'w') as fp:
    fp.write(output)
```

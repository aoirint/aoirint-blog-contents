---
title: 'AssistantSeika + 偽装ちゃん + NCVによるニコ生コメント読み上げ'
date: '2022-02-07T16:08:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Windows
tags:
  - Windows
  - hosts
---
# AssistantSeika + 偽装ちゃん + NCVによるニコ生コメント読み上げ

## 構成

- 音声合成ソフト（A.I. VOICE、ガイノイドTalk、VOICEVOX）
- AssistantSeika
- 偽装ちゃん
- NiconamaCommentViewer

## 初回設定手順

1. 上記ソフトウェアをダウンロード・インストール
2. 音声合成ソフトを起動、起動完了を待つ（AssistantSeikaのドキュメントを参照して各ソフトが動作する状態になっているか、設定を確認）
3. AssistantSeikaを起動、製品スキャンを実行、スキャン完了を待つ（1つ以上の音声合成ソフトが認識されていることを確認）
4. 偽装ちゃんの実行ファイル`FakeChan.exe`を`BouyomiChan.exe`という名前でコピー
5. `BouyomiChan.exe`から偽装ちゃんを起動
6. NiconamaCommentViewerを起動
7. メニューから「設定」を開き、「読み上げ」タブを開く。「コメント読み上げを使用する」にチェックを入れる。
8. 設定項目から「棒読みちゃん」を選択し、「棒読みちゃんの場所」に先ほどコピーした`BouyomiChan.exe`のパスを設定
9. 適当な放送を選び、URLから放送番号（`lv***`という文字列)をコピー（URL全体でもOK）、NiconamaCommentViewerの放送番号入力欄に貼り付ける、「接続」ボタンを押す

## 2回目以降の手順

1. 音声合成ソフトを起動、起動完了を待つ（AssistantSeikaのドキュメントを参照して各ソフトが動作する状態になっているか、設定を確認）
2. AssistantSeikaを起動、製品スキャンを実行、スキャン完了を待つ（1つ以上の音声合成ソフトが認識されていることを確認）
3. `BouyomiChan.exe`から偽装ちゃんを起動
4. NiconamaCommentViewerを起動
5. 放送番号を入力、「接続」ボタンを押す
5. 「読み上げ ON/OFF」ボタンが緑色（ONというラベル）になっている場合、押して灰色（OFFというラベル）にする

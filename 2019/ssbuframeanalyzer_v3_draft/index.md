---
# moved from https://aoirint.hatenablog.com/entry/2019/08/05/194314
title: SSBUFrameAnalyzer v3
date: '2019-08-05 19:43:14'
draft: false
channel: 技術ノート
category: スマブラSP
tags:
- スマブラSP
- 画像処理
---
# SSBUFrameAnalyzer v3

<h3><a class="keyword" href="http://d.hatena.ne.jp/keyword/History">History</a></h3>
<p><a href="https://github.com/aoirint/SSBUFrameAnalyzer">GitHub - aoirint/SSBUFrameAnalyzer</a></p>
<p><iframe class="embed-card embed-blogcard" style="display: block; width: 100%; height: 190px; max-width: 500px; margin: 10px 0px;" title="SSBUFrameAnalyzer - aoirint's note" src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Faoirint.hatenablog.com%2Fentry%2F2019%2F07%2F21%2F054224" frameborder="0" scrolling="no"></iframe></p>
<p><iframe class="embed-card embed-blogcard" style="display: block; width: 100%; height: 190px; max-width: 500px; margin: 10px 0px;" title="SSBUFrameAnalyzer v2 - aoirint's note" src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Faoirint.hatenablog.com%2Fentry%2F2019%2F07%2F21%2F205320" frameborder="0" scrolling="no"></iframe></p>
<h3>What's New</h3>
<ul>
<li>「ストック」を取得できるようにした（精度イマイチ）
<ul>
<li>3ストックまで想定（4ストック以上は未実装）</li>
<li>一応<a class="keyword" href="http://d.hatena.ne.jp/keyword/%C3%C4%C2%CE%C0%EF">団体戦</a>考慮してストックごとにキャ<a class="keyword" href="http://d.hatena.ne.jp/keyword/%A5%E9%A5%AF">ラク</a>ター推定</li>
<li>精度イマイチな原因かも：背景、ファイター順によってちょっと決め打ち座標がズレてる、画像が小さすぎる→大きく補間してから<a class="keyword" href="http://d.hatena.ne.jp/keyword/HOG">HOG</a>？</li>
</ul>
</li>
<li>勇者（Hero）のデータを追加（v3のdictionary）
<ul>
<li>コマンド出てるときは（顔に被るから）邪魔で難しいと思う...</li>
</ul>
</li>
</ul>
<p><img src="https://cdn-ak.f.st-hatena.com/images/fotolife/k/kanomiya/20190805/20190805195125.png" alt="f:id:kanomiya:20190805195125p:plain" title="f:id:kanomiya:20190805195125p:plain" class="hatena-fotolife" itemprop="image" /> <img src="https://cdn-ak.f.st-hatena.com/images/fotolife/k/kanomiya/20190805/20190805195144.png" alt="f:id:kanomiya:20190805195144p:plain" title="f:id:kanomiya:20190805195144p:plain" class="hatena-fotolife" itemprop="image" /></p>
<p><a href="https://github.com/aoirint/SSBUFrameAnalyzer/tree/v3">GitHub - aoirint/SSBUFrameAnalyzer at v3</a></p>
<h3>TODO</h3>
<ul>
<li>ファイター数の推定</li>
<li>bboxのズレ解消</li>
<li>bboxの自動算出</li>
<li>残タイムの推定</li>
<li>画面中のファイター位置の推定
<ul>
<li>深層学習（YOLO/R-CNN系かセグメンテーション）になりそう、データがない...</li>
</ul>
</li>
<li>精度向上
<ul>
<li>normalizeしたり、<a class="keyword" href="http://d.hatena.ne.jp/keyword/%A5%B3%A5%F3%A5%C8%A5%E9">コントラ</a>スト上げたり、周辺ぼかすフィルタ掛けたりしたら精度上がらないかな</li>
</ul>
</li>
<li>チャージ・残量系キャラ（<a class="keyword" href="http://d.hatena.ne.jp/keyword/%A5%EB%A5%D5%A5%EC">ルフレ</a>、<a class="keyword" href="http://d.hatena.ne.jp/keyword/%A5%AF%A5%E9%A5%A6%A5%C9">クラウド</a>、<a class="keyword" href="http://d.hatena.ne.jp/keyword/%A5%EA%A5%C8%A5%EB%A1%A6%A5%DE%A5%C3%A5%AF">リトル・マック</a>、<a class="keyword" href="http://d.hatena.ne.jp/keyword/%A5%A4%A5%F3%A5%AF%A5%EA%A5%F3%A5%B0">インクリング</a>、ジョーカー、勇者他？）のチャージ・残量推定
<ul>
<li><img src="https://cdn-ak.f.st-hatena.com/images/fotolife/k/kanomiya/20190805/20190805195955.png" alt="f:id:kanomiya:20190805195955p:plain" title="f:id:kanomiya:20190805195955p:plain" class="hatena-fotolife" itemprop="image" /> <img src="https://cdn-ak.f.st-hatena.com/images/fotolife/k/kanomiya/20190805/20190805195555.png" alt="f:id:kanomiya:20190805195555p:plain" title="f:id:kanomiya:20190805195555p:plain" class="hatena-fotolife" itemprop="image" /> <img src="https://cdn-ak.f.st-hatena.com/images/fotolife/k/kanomiya/20190805/20190805200110.png" alt="f:id:kanomiya:20190805200110p:plain" title="f:id:kanomiya:20190805200110p:plain" class="hatena-fotolife" itemprop="image" /> <img src="https://cdn-ak.f.st-hatena.com/images/fotolife/k/kanomiya/20190805/20190805195453.png" alt="f:id:kanomiya:20190805195453p:plain" title="f:id:kanomiya:20190805195453p:plain" class="hatena-fotolife" itemprop="image" /> <img src="https://cdn-ak.f.st-hatena.com/images/fotolife/k/kanomiya/20190805/20190805200005.png" alt="f:id:kanomiya:20190805200005p:plain" title="f:id:kanomiya:20190805200005p:plain" class="hatena-fotolife" itemprop="image" /> <img src="https://cdn-ak.f.st-hatena.com/images/fotolife/k/kanomiya/20190805/20190805195717.png" alt="f:id:kanomiya:20190805195717p:plain" title="f:id:kanomiya:20190805195717p:plain" class="hatena-fotolife" itemprop="image" /></li>
</ul>
</li>
<li>ステージの推定（データ...）</li>
</ul>

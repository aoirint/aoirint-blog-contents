# このブログ「えやみぐさ」のデプロイをDocker化して高速化した

## ブログ・静的サイトジェネレータとの付き合い

SSGの悩み事: ビルド時間

SSGを意識して使い始めたのは、2020年9月にPython製のMiyadaikuを使ってブログを組み立てたところからだと思う。
2021年11月にNode.js/TypeScript製のGatsbyに移った。

- <https://blog.aoirint.com/entry/2020/miyadaiku_github_actions/>

このaoirint名義で残っているのは、2017年9月の記事が最初だろうか。

2020年はQiitaに何本か上げている。

1年ごとに個人ブログシステムを乗り換えているけれど、技術的なことを書く個人ブログとしては、この「えやみぐさ」で安定したと思う。

どうにも秋になるとブログを書きたくなるらしい。


## デプロイ時間

旧デプロイシステムでは、npmやGatsbyのキャッシュが効いていて9分半程度だった。

![](images/old_workflow_duration.png)

新デプロイシステムでは、2分半程度になった。

![](images/new_workflow_duration.png)

## やったこと

- GatsbyプロジェクトをDockerイメージ化
- GitHub Pagesのデプロイ元をgh-pagesブランチからGitHub Actionsに変更
    - <https://github.blog/changelog/2022-07-27-github-pages-custom-github-actions-workflows-beta/>
    - <https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow>

## GatsbyプロジェクトをDockerイメージ化

- シンプルに`node:14`ベースで`npm ci`したイメージを作っただけ
- `gosu`を使って一般ユーザで実行
- 依存関係のバイナリビルドなどのインストール時処理が事前に済ませられる
    - Node.jsの一部パッケージはインストール時にバイナリビルドするものがおそらくある
    - Pythonにもインストール時にC/C++コンパイラやCMakeを要求するパッケージがある、ただしローカル環境に対応するwheelが用意されていない場合
      - [r9r9/pyopenjtalk](https://github.com/r9y9/pyopenjtalk)
      - たぶんNumPyとかOpenCVとか
      - 新しすぎるPython本体バージョン・armなど特殊な環境だと、wheelが用意されていなくてローカルビルドになることも（それでも開発者がビルドできるように整えてくれていると使える）

![](images/old_workflow_steps_duration.png)

- npmパッケージのインストール時処理時間は支配的だった
    - 本質的なGatsbyプロジェクトのビルド処理が30秒程度
    - 依存関係のインストール時処理は6分半から7分程度
- ソースコードの代わりに各環境向けのビルド済みバイナリを提供することでインストールが簡便・高速になるなどの利点を受けられる仕組み（プロプライエタリ？ 知らない子ですね...）
    - apt, PPA
    - Python wheel
- 単純化すると「ビルド時間をストレージ容量で代替する」


## GitHub Pagesのデプロイ元をgh-pagesブランチからGitHub Actionsに変更

### gh-pagesブランチを使ったデプロイ

- `gh-pages`ブランチの更新には、[peaceiris/actions-gh-pages@v3](https://github.com/peaceiris/actions-gh-pages/tree/v3)を使っていた
    - たぶんかなり有名（3000スター）で、自分でも布教している
- 2021年12月から、`gh-pages`ブランチにpushすると、`pages-build-deployment`というWorkflowが実行され、GitHub Pagesのデプロイ処理が見えるようになった
    - <https://github.blog/changelog/2021-12-16-github-pages-using-github-actions-for-builds-and-deployments-for-public-repositories/>
    - <https://github.blog/2022-08-10-github-pages-now-uses-actions-by-default/>

![](images/pages_build_deployment.png)

### GitHub Actionsを使ったデプロイ

- 2022年7月にベータリリースされたGitHubの機能で、`gh-pages`ブランチを介さずにGitHub Pagesをデプロイできるようになった
    - <https://github.blog/changelog/2022-07-27-github-pages-custom-github-actions-workflows-beta/>

![](images/github_actions_deployment.png)

![](images/github_actions_deployment_suggestion.png)

- GitHub公式が提供しているGitHub ActionsのテンプレートStarter WorkflowにJekyll、Hugo、Gatsby、Next.js、NuxtJSなどいくつかのフレームワークを使ったサンプルがある
  - <https://github.com/actions/starter-workflows/tree/5e60d02759829e0e27db87bd924ea5b45dd8503d/pages>

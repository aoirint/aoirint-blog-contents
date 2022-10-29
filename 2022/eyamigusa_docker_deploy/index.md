# このブログ「えやみぐさ」のデプロイをDocker化して高速化した

SSGの悩み事: ビルド時間

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

## GitHub Pagesのデプロイ元をgh-pagesブランチからGitHub Actionsに変更

- 2022年7月にベータリリースされたGitHubの機能
    - <https://github.blog/changelog/2022-07-27-github-pages-custom-github-actions-workflows-beta/>
- `gh-pages`ブランチの更新には、[peaceiris/actions-gh-pages@v3](https://github.com/peaceiris/actions-gh-pages/tree/v3)を使っていた
- このリリースの頃から、`gh-pages`ブランチにpushすると、`pages-build-deployment`というWorkflowが実行され、GitHub Pagesのデプロイ処理が見えるようになった

![](images/pages_build_deployment.png)

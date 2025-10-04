---
layout: post
title: "GitHub Actions を自作して Marketplace で公開してみた"
date: 2024-04-29
---

# はじめに

- GitHub  では GitHub Actions と言って CI/CD  のための機能がある
    - たとえば push 後に GitHub サーバー側で、テストを実行する等
- この Action で再利用可能なパーツとして actions/checkout 等がある
- このパーツを自作することもでき、それを GitHub Marketplace で公開できる。

# やり方

参考文献に挙げたサイトが分かりやすい。

今回作ったアクション
- リポジトリ :
[https://github.com/takahiro-itou/install-cppunit-action][link01]
- マーケット :
[https://github.com/marketplace/actions/install-cppunit][link02]

やり方の概要

- アクションを保存するためのリポジトリを GitHub に作る。
    - リポジトリはパブリックにする
- リポジトリ直下に action.yml を作成する
- さらに README.md や LICENSE も作成する
- ウェブブラウザで GitHub にアクセスし、リポジトリを開く。
- 所定のファイルがあれば、「マーケットプレイスに公開」というボタンが現れる
- 利用規約に同意して、
- カテゴリとタグを設定してリリースボタンを押せば完了。


# 参考文献

- [https://docs.github.com/en/actions/creating-actions][link03]
- [https://docs.github.com/en/actions/creating-actions/publishing-actions-in-github-marketplace][link04]
- [https://qiita.com/eggplants/items/ad949f6934fb292a9d4f][link05]
- [https://meetup-jp.toast.com/3528][link06]


[link01]: https://github.com/takahiro-itou/install-cppunit-action
[link02]: https://github.com/marketplace/actions/install-cppunit
[link03]: https://docs.github.com/en/actions/creating-actions
[link04]: https://docs.github.com/en/actions/creating-actions/publishing-actions-in-github-marketplace
[link05]: https://qiita.com/eggplants/items/ad949f6934fb292a9d4f
[link06]: https://meetup-jp.toast.com/3528

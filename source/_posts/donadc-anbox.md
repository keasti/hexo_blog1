---
title: Anboxのご紹介
date: 2018-12-25 00:00:00 
tags:
---
> これは [mstdn.maud .io Advent Calendar 2018](https://adventar.org/calendars/2892) 25日目の記事です  
> 昨日は [@zgock999](https://gist.github.com/zgock999/6c74f16ae4b1958e8b8977ec3d26deb5) さん でした

> これは2018/12/13に開催された [東海道らぐ 2018年12月オフな集まり in 名古屋](https://tokaidolug.connpass.com/event/112711/) で発表した内容をブログ向けにまとめ直したものとなります

## Android App On Linux
皆さんはLinuxデスクトップ上でAndroidアプリを動かしたくなったことはありませんか？これまでそれを実現する方法としてはエミュレータを使う方法がありました。だだしこれらは動作速度であったり、他のLinuxソフトウェアとの併用に難があるのが問題でした。そこで今回はこれらの問題を解決してくれるかもしれない新たな方法を紹介しようと思います。

## "Anbox" ~ Android in Box
今回紹介するのは[Anbox](https://anbox.io/)というツールです。これはLXCという技術を使い、Linuxカーネルをホストと共有しAndroidアプリをコンテナ上で実行しようとするものです。これによって、AndroidアプリをLinuxデスクトップ上であたかもネイティブソフトウェアのように実行することができます。

### インストール
<!--more-->
インストール方法は[公式ドキュメント](https://github.com/anbox/anbox/blob/master/docs/install.md)にまとめられています。Ubuntu系ディストリであればその方法でインストールできると思われます。わたしは Ubuntu 18.04 / 18.10 で動作確認しました。以下にコマンドをまとめておきます。
* カーネルモジュールのインストール

 - PPA追加

    ```sh
    $ sudo add-apt-repository ppa:morphis/anbox-support
    $ sudo apt update
    $ sudo apt install anbox-modules-dkms
    ```

  - カーネルモジュールの有効化と確認

    ```sh
    $ sudo modprobe ashmem_linux
    $ sudo modprobe binder_linux
    $ ls -1 /dev/{ashmem,binder} #確認
    #以下の2つが存在すればOK
    /dev/ashmem
    /dev/binder
    ```

* 本体のsnapパッケージのインストール

  ```sh
  $ sudo snap install --devmode --beta anbox
  ```

以上でインストール完了となります。インストール完了後は、念のために再起動を行うことをおすすめします。

#### パッケージの更新について
インストール時に ``--devmode`` オプションをつけている関係上、自動での更新は行ってくれません。手動で更新するためには、以下を実行してください:

```sh
$ snap refresh --beta --devmode anbox
```

### インターネット接続
インストール時点では、コンテナ内からインターネットに接続することができません。接続するためには、ホストのネットワークをコンテナにブリッジさせる必要があります。そのためのスクリプトが[公式リポジトリ](https://github.com/anbox/anbox)に用意されています。以下を実行してください:

```sh
$ git clone https://github.com/anbox/anbox
$ cd anbox/scripts
$ sudo bash anbox-bridge.sh start
```

これでコンテナ内に `eth0` が生え、インターネットに接続することが可能なはずです。この方法でのインターネット接続は、ホストOSの再起動ごとに行う必要があります。

## 他のディストリでの使用
### ArchLinux系
公式snapパッケージはUbuntu系ディストリ向けに作成されているため、他のディストリでは使用することができません。そこで、わたしはAURに一通りのパッケージが存在するArchLinuxへのインストールを行ってみました。
#### インストール
インストール方法は、検索したところ[こちらのフォーラムの投稿](https://forum.manjaro.org/t/running-android-applications-on-arch-using-anbox/53332)を発見しました。こちらの方法で無事インストール、動作の確認ができました。まとめると以下のようになります。

* お好みのAURヘルパー（今回は[Yay](https://github.com/Jguer/yay)）から、AURにあるパッケージ3つをインストール

  ```sh
  $ yay -S anbox-git anbox-modules-dkms-git anbox-image
  ```

* カーネルモジュールの有効化

  ```sh
  $ sudo modprobe ashmem_linux
  $ sudo modprobe binder_linux
  $ ls -1 /dev/{ashmem,binder} #確認
  #以下の2つが存在すればOK
  /dev/ashmem
  /dev/binder
  ```

* systemdサービスの有効化

  ```sh
  $ sudo systemctl enable anbox-container-manager.service
  ```

以上が済んだら、再起動を行ってください。これでインストール完了です。  
インターネット接続は、上のUbuntu系と同じ方法で行うことができました。

## 使ってみる
### Androidアプリのインストール
AnboxはデフォルトでADB接続が可能です。それを利用し、ホストに用意したapkファイルからアプリのインストールを行うことができます。
#### GooglePlay
AnboxへのGooglePlayのインストール方法は、上のArchLinuxへのインストール方法が書かれたフォーラム投稿にあるスクリプトを実行することで可能でした。手元で行う際には、セキュリティ等のためスクリプトの内容をよく確認してから実行されることを強く推奨します。

### アプリ動作チェック
いくつかのアプリを動かしてみた結果は以下の通りです。  

| アプリ名 | 動作可否 | 備考 |
|:---------|:---------|:---------|
| FireFox | ◎ | 概ね動作OK |
| Google Chrome | × | 起動はするもののフリーズ |
| Tusky | ○ | 概ね動作OK |
| SubwayTooter | ○ | 概ね動作OK　タブレットモード可 |
| カスタムキャスト | × | 起動不可 |
| AngryBirds | × | 起動不可 |
| Hole.io | ○ | ほぼプレイ可 |  

動かしてみた感覚としては、ユーティリティ系は基本動作する、ゲーム系のアプリは動作しないものが多いが動作しないものもある、といった感じでした。時間があれば追加で検証しようと思っていますので検証してほしいアプリがあればお教えください。

### 使ってみて気づいた点
* 戻るボタンが存在しなく、いくつかのアプリがまともに扱えない
  - Escキーで代用可能ではある
* 動作が不安定
  - アプリの起動→終了を繰り返していると次第に動作が怪しくなり、最終的にはホストを再起動させないと動作しなくなったことがあった
    - ホストとカーネル共有している関係で発生？
* 開発者オプションが開けない

## 最後に
* Anboxはやはりまだアルファ版であり、動作しないアプリが多くあったり、動作が不安定だったりします。ただ、動作するアプリであればエミュレータ上よりは快適に使えそうでははあると感じました。
* 現在、Android 8 Oreo / 9 Pie のイメージも開発中のようです。時間があれば手元のでビルドも試してみようかと思っています。
* これからの改良や発展に期待します。

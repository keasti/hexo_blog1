---
title: Fire HD 8 (2018) に LineageOS 16.0 をインストールする
date: 2019-12-06 00:00:00
tags:
---
> これは [mstdn.maud .io Advent Calendar 2019](https://adventar.org/calendars/3963) 6日目の記事です  
> 昨日は [@umezou](https://vivid-rabbit.net/20191205) さん でした
## 通称光る板
あの Amazon が発売している Fire HD 8 という格安タブレットPCですが、これにはAndroid ベースにカスタマイズし Amazon サービスに最適化された Fire OS がインストールされており、デフォルトの状態では Play ストアなどの Google サービスが一切利用できないことはみなさんご存知でしょう。（非公式で Play ストアをインストールすることが可能なようですが…）  
ただし、Fire シリーズは以前からHackが盛んであり、これまでに Fire HDX や Fire 7 などのモデルのHack方法が確立されてきました。この度XDA民の謎の力によって Fire HD 8 最新モデル（2018）に LineageOS などのカスタムROMをインストールする方法が発見されたため、その手法を紹介しようと思います。

### 参考にしたXDAスレッド
* https://forum.xda-developers.com/hd8-hd10/orig-development/unlock-fire-hd-8-2018-karnak-amonet-3-t3963496

<!--more-->

> ## Disclaimer / 警告
> 以下の内容にはデバイスのファームウェアを直接変更する作業が含まれるため、失敗した場合は一切デバイスが起動しなくなる（文鎮化）可能性があります。実行する場合は完全に自己責任で行ってください。

## 試した環境
### Fire HD 8 (2018)
* 2019/11初頭に新品購入
* 16GBモデル
* ファームウェアバージョン: 6.3.1.2

### PC
* Arch Linux をインストール済み

## メモ
* セットアップ時にWi-Fi接続しない（自動アップデート防止）
  - SSID選択画面で一旦をSSID選択してからキャンセルするとスキップボタンが現れる
* 開発者オプションの有効化
  - 設定 - デバイスオプション - バージョン情報 - シリアル番号 を7回連打 

## 1. 準備
### 1-1. 必要パッケージの準備
#### Arch Linux での必要パッケージは以下
* `python`
* `python-pyserial`
* `android-tools`
* `dos2unix`

#### Ubuntu の場合は上記XDAスレッドを参照

### 1-2. ModemManagerの無効化
```sh
$ sudo systemctl stop ModemManager
$ sudo systemctl disable ModemManager
```
 - 作業終了後は有効化を忘れずに

### 1-3. ファイルの準備
* 上記XDAのスレッドから `amonet-karnak-v3.0.1.zip` と `brick-karnak.zip` をDLする
* これらを展開し、`brick-karnak.zip` 内の `amonet` フォルダ内を `amonet-karnak-v3.0.1.zip` 内の `amonet` フォルダにコピー（これは必要ないかもしれない）

## 2. Unlock実行
1. 開発者オプションからADBの有効化
2. PCと接続する
  - USBの接続モードをPTPモードに切り替え
  - ADB 接続確認 
    - `$ adb devices`
3. ブートローダー（fastboot）へ
  - `$ adb reboot bootloader`
  - "=> FASTBOOT mode... " 表示
  - （fastboot コマンドは `sudo` をつけないと動作しないかも）
4. `amonet` フォルダに移動しbrickスクリプト実行
  - `$ ./brick-6312.sh`
5. "YES"と入力するとbrick処理開始
  - "All good ~" 表示で完了
  - 電源ボタン長押しでFireの電源オフ
6. PCと切断
7. Bootrom ステップ実行
  - `$ sudo ./bootrom-step.sh`
  - "Waiting for bootrom" が出たらPC接続
  - Enter で続行
8. 終了後一旦PCと切断し、再接続
  - hacked fastboot mode で起動
  - `$ sudo ./fastboot-step.sh`
9. 再起動後、TWRPが起動
10. Unlock完了！

## 3. LineageOS のインストール
  - LineageOS 16.0 は [こちら](https://forum.xda-developers.com/hd8-hd10/orig-development/rom-lineage-16-0-t3981685)
  - インストール方法はいつもの
  - お好みで Magisk 等
    - 最新バージョンでBrickの情報？
    - v20.0で問題ないことを確認
  - 現状 Bluetooth やヘッドホンが使えないなどの不具合あり
    - ヘッドホンの修正パッチあり？（未確認）
    - https://forum.xda-developers.com/hd8-hd10/orig-development/rom-lineage-16-0-t3981685/post81058737#post81058737

#### Magisk などのアップデートについて
  - Unlock 手法の関係上 boot/recovery イメージはOS上から焼いてはいけないため、Magisk などそれらのイメージを書き換えるものは必ず TWRP から焼く

#### Bluetooth が無限にエラーを吐く対策
  1. ROM本体のみをインストールし、一旦セットアップを完了させる
  2. 開発者オプションからADB有効化、PCと接続
  3. `$ adb shell pm disable com.android.bluetooth`
  4. Fire再起動
  5. お好みで Magisk → Gapps の順にインストール
  6. Profit!

#### Gappsのバージョンについて
  - ブートループの可能性があるので1003/1006バージョンの使用を推奨

## 最後に
今回は Fire HD 8 に LineageOS をインストールし、通常の Android タブレットとして使えるようにする方法を紹介しました。まだ Bluetooth やヘッドホンが使えないなど不安定なところはありますが、通常のAndroidタブレットより安く購入できる Fire HD 8 にカスタムROMがインストール可能になったのは大きな魅力でしょう。公開されているリポジトリを見ると開発は活発であり、LineageOS 17 の移植も進められているようなので、これからにも期待できそうです。

> 以上 [mstdn.maud .io Advent Calendar 2019](https://adventar.org/calendars/3963) 6日目の記事でした  
> 明日は [@siki_uta](https://sikilog.blogspot.com/2019/12/linux-install-battle.html) さん です

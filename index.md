---
layout: post
title: Top page
---

このページはTwitter社が公開しているOSS、StormのTipsをまとめるページです。

## Stormとは？
Stormとは、フリーでオープンソースな『フォールトトレラントなリアルタイム分散処理フレームワーク』です。  
簡単に言うと、「一部で障害が発生しても動き続ける早いフレームワーク」。

Stormは連続的なストリームデータを安定して処理することを実現しています。  
Stormはバッチ処理におけるHadoopのような役割をストリーム処理に対して実現します。  
StormはシンプルなAPIでさまざまなプログラミング言語から利用することが可能です。

## このページの目的
このページは、実際にStormを使用してみて困った点やわかった点をまとめています。  
そのため、「Stormとは何か？」の１つを取っても内部構造の話が出てくることも多く、Stormについてある程度わかっていることを前提としています。

尚、このページはStormの背景や概要、全体像を示すことを目的としていません。  
そのため、Stormについて背景や概要から全体像を知りたい方は下記のページを参照してください。  
[StormProjectのメインページ（英語）](http://storm-project.net/)  
[StormProjectの日本語訳ページ](http://stormjp.github.com/storm-website-jp/)

## Stormを利用する際の環境
* [開発環境](/environment/develop.html)
* [クラスタ環境](/environment/cluster.html)

## StormのAPIの説明
* [SpoutBoltで使用できるAPI](/api/spoutboltapi.html)

## Stormの内部動作
* [Nimbus起動時の動作]
* [TopologySubmit時の動作]
* [Supervisor起動時の動作]
* [ヘルスチェックの動作]

## Stormを利用する際に気をつけること
* [環境周りの注意点]
* [Topology起動時の注意点]
* [SpoutBoltの注意点]
* [その他の注意点]

## Stormトラブルシューティング

## Stormソースコードリーディング

## 参考

### 参考ページ
* [StormProjectのメインページ（英語）](http://storm-project.net/) 本家サイト。Stormの基本情報から発表資料までそろっています。
* [StormProjectの日本語訳ページ](http://stormjp.github.com/storm-website-jp/) 日本語で全体的に知りたい場合、このページを確認してください。

### 参考資料
* [Twitterのリアルタイム分散処理システム「Storm」入門](http://www.slideshare.net/AdvancedTechNight/twitterstorm)  
日本語で「Stormとは何か？」がコンパクトにまとまった資料です。本家サイトの前にまずこの資料を見るとわかりやすいです。

* [Stormの注目の新機能TridentAPI](http://www.slideshare.net/AdvancedTechNight/stormtridentapi)  
Stormで「演算」を記述する際に簡易に開発が可能なTridentAPIの紹介資料。

### 参考書籍
* [Getting Started with Storm](http://shop.oreilly.com/product/0636920024835.do)  
Stormの薄い本。Storm-Projectページにのっていること以上のことが書かれていないうえバージョンも古い。  
そのためお勧めできません。  
ただ、サンプルとして書かれているアプリケーションについては面白いです。

* [Stormをはじめよう](http://www.oreilly.co.jp/books/9784873116013/)  
「Getting Started with Storm」の日本語訳版。  
やはりバージョンは古いのですが、日本語なので日本語で全体的に知りたい場合有効です。


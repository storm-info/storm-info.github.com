---
layout: post
title: 環境周りの注意点
---

このページではStormを利用する際に環境周りで注意が必要な点をまとめます。

### StormのNimbus、UI、Supervisorにはデフォルトでは終了コマンドはない
そのため、[Storm-Installer](https://github.com/acromusashi/storm-installer)のようにラッピングしたコマンドを作成するか、killコマンドで落とすしかありません。

### Stormクラスタ自体のアップデートを確実に行う手順
* 1.動作中のTopologyを全て終了させる
* 2.storm-nimbus、storm-ui、storm-supervisorを終了させる
* 3.ZooKeeper上の/storm 配下のデータを削除する
* 4.【Stormインストール先】のnimbus、supervisor、workersディレクトリを削除
* 5.クラスタのアップデートを実施
* 6.storm-nimbus、storm-ui、storm-supervisorを起動させる

尚、Stormにとっては動作しながらのクラスタ自体のアップデートへ対応する優先度は低いとされています。  
そのため、開発を待つのはなかなか時間がかかりそうです。

### Stormの動作がおかしくなった場合の対処方法
Supervisorが起動しなくなる、Workerが起動しなくなるなどStormが起動しなくなった場合、ZooKeeperか、ローカルのファイルの状態がおかしくなった可能性が大きい。  
そのため、以下の対処を順に行う。1でなおらなければ、2を行う。  
1.【Stormインストール先】のsupervisor、workersディレクトリを削除する  
2.ZooKeeper上の/storm 配下のデータを削除する  
3.【Stormインストール先】のnimbusディレクトリを削除  
  
Nimbusは基本起動し続けるためか、あまり状態がおかしくなることはない。そのため優先度は下げている。  
尚、これらの問題はバージョンが進むたびにどんどん対応されていく（バージョン0.8.2系で発生する問題がバージョン0.8.3系では発生しないなど）ため、バージョンアップを適宜行っていけば問題は発生しにくくなる。


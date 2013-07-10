---
layout: post
title: Spout/Boltの動作
---

このページではWorkerプロセス中のSpout/Boltの動作についてまとめています。

### Spout/Boltで例外が発生してcatchされなかった場合はWorkerプロセスごとフェールオーバーする
Spout/Boltで例外が発生してcatchされなかった場合はWorkerプロセスごとフェールオーバーします。  
Spout/Boltで例外が発生してユーザが開発したコード部でcatchされなかった場合は対象Spout/Boltのスレッドが停止します。  
その結果、Workerプロセスごとフェールオーバーする形になります。  
Workerプロセスごとフェールオーバーする理由としてはNimbusは各スレッド単位で生存確認を行っているためです。  

尚、上記の場合発生した例外の内容がStorm-UIに表示されます。  
そのため、例外を投げる場合にはWorkerプロセスごとフェールオーバーすることを前提で投げる必要があります。  
__但し、Storm0.8.2以降であればReportedFailedExceptionを投げればスレッドを停止させずにStorm-UIに情報を表示させることができる模様。__  
_上記の判定とStorm-UIに表示させている個所は特定できていません。_  
_BoltExecutor系のクラスにおいてはログ出力を行ってはいるが、それだけしか記述されていないです。（要確認）_  

### Spout/Boltのライフサイクル
Spout/Boltのライフサイクルは下記のようになっています。  

* 1.トポロジを起動したプログラムにおいて生成される（コンストラクタが呼び出される）
* 2.シリアライズされ、Nimbusのローカルディレクトリに保存される
* 3.Workerの割り振りを受けたSupervisorがNimbusからThriftAPIを用いて受信する
* 4.各Workerプロセスでデシリアライズされる
* 5.Spoutならopenメソッド、Boltならprepareメソッドが呼び出される
* 6.Tupleの送信が開始され、SpoutならnextTupleメソッド、Boltならexecuteメソッドが連続して呼び出されるようになる

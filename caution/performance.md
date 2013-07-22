---
layout: post
title: 性能周りの注意点
---

このページではStormを利用する際に性能周りで注意が必要な点をまとめます。

### StormからのZookeeperへのファイルIO負荷はそれなりに高い
主な原因としては頻度の高いHeartBeatをZooKeeperを用いて行っているための可能性が大きい。  
ZooKeeperは状態更新が行われた際に各サーバへのファイルIOが発生するため、HeartBeatの度にファイルIOが発生していることとなる。  
尚、ファイルIO回数は多いものの、ファイルIOの量は少ないため、ZooKeeperのデータを保持する場所をSSD上に移すことで大きく状況が改善することもある模様。  

### 各Taskの多重度とWorkerのスロットの関係
storm.yamlの「supervisor.slots.ports」は『そのマシンでいくつWorkerプロセスを動作させるか』を示します。  
SpoutとBoltの多重度は合算されてStormクラスタ内のWorkerプロセスに等分されます。  
  
例として、WorkerSlotが2のマシンAと、WorkerSlotが3のマシンBがあって、その中でSpoutAlphaを多重度10、BoltBetaを多重度20、BoltGammaを多重度5で起動させたとすると、各Workerプロセスでは(10+20+5)　/ 5 ＝ 7のSpout/Boltが実行されることになります。  
その際、マシンAではWorkerプロセスが2プロセス、マシンBではWorkerプロセスが3プロセス起動される。

### Stormが主に使用するリソースはCPU
そのため、CPUの利用度合いをベースにサイジングを行う形になります。  
ただし、これはあくまでStormのみの場合。利用するライブラリ次第で傾向は変わります。  
尚、目安として、「CPUのコア数に対するTaskの目安はコア数×3程度」が一つの指標となるそうです。  

### Storm-UIはサンプリングレート次第で実際の値と微妙にずれることがある
Storm-UIで表示している性能情報は厳密に正確な値ではありません。Stormには結果を統計に反映するかを示すサンプリングレートが設定されており、Tupleを処理するごとに判定が行われます。  
例えば、サンプリングレートが「0.05(デフォルト)」の場合、Tupleを処理するごとに5%の確率で統計反映処理が走り、「該当のTupleの結果が20件分統計に反映」されます。  
この反映処理は単純に確率で判定されるため、最初の1件を処理した時点で20件処理したと表示される可能性もあります。結果、Spout/Boltの処理件数が微妙に不整合が発生しているように見えることがあります。

### Storm-UIのサンプリングレートは変更可能
上記で説明している「サンプリングレート」は設定項目「topology.stats.sample.rate」で設定可能です。  
この値を1にした場合はStorm-UIの統計の値が厳密になりますが、処理負荷としてはその分重くなります。逆にこの値を0.0001のように低くすれば統計情報が荒くなるものの、処理速度の向上が見込めます。

### ZeroMQネイティブ領域にメッセージが蓄積されてリークすることもある
StormはJavaのHeapの他にもZeroMQネイティブ領域にメッセージが蓄積されることがあります。そのため性能を確認する際にはネイティブ領域も確認することが必要です。

### ZooKeeperの負荷が大きい場合、サンプリングレートとHeartBeatの間隔を調整する
StormクラスタにおいてZooKeeperへの負荷が大きい場合、Storm-UIのサンプリングレート(設定項目「topology.stats.sample.rate」)と、ZooKeeperへのHeartBeat間隔(設定項目「supervisor.heartbeat.frequency.secs」「worker.heartbeat.frequency.secs」「task.heartbeat.frequency.secs」)を調整することで負荷を軽減することが可能です。

### 処理速度がSpout ＞ Boltの場合、Spoutの最大取得Tuple数の設定を行う
StormではSpoutとBoltが独立したスレッドで動作するため処理速度がSpout ＞ Boltの場合、StormのプロセスにTupleが大量に蓄積され、溢れます。そのため、処理速度がSpout ＞ Boltの場合においてはSpoutの最大取得Tuple数（設定項目「topology.max.spout.pending」）を設定する必要があります。
上記の設定項目でSpout1スレッドごとにBoltが処理完了していないTupleをいくつまで取得できるかが設定できます。（ただし、Ack/Fail機構を有効化していない場合に動作するかは未確認。）  
尚、この項目はデフォルト値では未設定となるため、明示的に指定しない限り有効になることはありません。


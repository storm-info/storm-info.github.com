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


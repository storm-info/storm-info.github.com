---
layout: post
title: 開発環境
---

このページでは開発環境構築の際に気をつける点、制限事項などをまとめています。

## Stormを用いたシステムが開発可能な環境
Stormを用いたシステムはJava6以上がインストールされた環境であれば開発が可能となる。  
開発と、StormLocalClusterの動作が可能。

## OSごとの制限

### Windows

* StormLocalCluster動作時に一時ファイル削除に失敗する。

StormLocalClusterを動作させた際に疑似ZooKeeper（Stormプロセスの中で動作する、StormLocalClusterの動作を管理するコンポーネント）用のファイル削除に失敗します。そのため、終了時などに下記の例外が発生するケースがある。  
発生した場合StormLocalClusterを実行するたびに1回あたり64MB程のゴミファイルが一時ディレクトリに残る。  
発生しなくてもテストコード実行時などはこっそり残っているケースもある。  
そのため、Jenkinsサーバ等でStormのテストコードを毎日実施する場合、ディスク容量に注意。  

#### ファイル削除失敗時発生エラー

    java.io.IOException: Unable to delete file: C:\Users\kimutansk\AppData\Local\Temp\72b0bc04-1a72-4e75-a8eb-65b60e4f7f72\version-2\log.1  
    	at org.apache.commons.io.FileUtils.forceDelete(FileUtils.java:1390)  
    	at org.apache.commons.io.FileUtils.cleanDirectory(FileUtils.java:1044)  
    	at org.apache.commons.io.FileUtils.deleteDirectory(FileUtils.java:977)  
    	at org.apache.commons.io.FileUtils.forceDelete(FileUtils.java:1381)  
    	at org.apache.commons.io.FileUtils.cleanDirectory(FileUtils.java:1044)  
    	at org.apache.commons.io.FileUtils.deleteDirectory(FileUtils.java:977)  
    	at org.apache.commons.io.FileUtils.forceDelete(FileUtils.java:1381)  
    	at backtype.storm.util$rmr.invoke(util.clj:413)  
    	at backtype.storm.testing$kill_local_storm_cluster.invoke(testing.clj:164)  
    	at backtype.storm.LocalCluster$_shutdown.invoke(LocalCluster.clj:32)  
    	at backtype.storm.LocalCluster.shutdown(Unknown Source)  
    	at storm.sample.topology.DecisionTestTopology.main(DecisionTestTopology.java:234)  



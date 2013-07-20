---
layout: post
title: Topologyコード例
---

このページではTopologyを作成する際のコード例やリンクについてまとめます。

### FieldGrouping で違うSpout/BoltからのTupleを１つのBoltから受信する記述方法
FieldGrouping で違うSpout/Boltからのタプルも同じ名前のフィールドを持っていればグループ化出来ます。このことを利用して下記のように実装を行うことで異なるBoltの結果を同じグルーピングの中に落とし込んで動作させることが可能です。

##### 複数のSpoutから同じFieldGroupingで1つのBoltが受信する記述例

    // wordというフィールドを保持するTupleを流すSpout
    builder.setSpout("Word1Spout", new TestWordSpout(), word1SpoutPara);
    // wordというフィールドを保持するTupleを流すSpout
    builder.setSpout("Word2Spout", new TestWordSpout(), word2SpoutPara);
    
    // Word1Spout、Word2Spoutの結果をwordフィールドでグルーピングして受信するBoltを追加
    builder.setBolt("JudgeBolt", new JudgeBolt(), judgeBoltPara).fieldsGrouping("Word1Spout", new Fields("word")).fieldsGrouping(
                "Word2Spout", new Fields("word"));
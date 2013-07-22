---
layout: post
title: Topology作成時の注意点
---

このページではTopology作成時に気をつける点などをまとめます。

### 独自エンティティクラスはTopology起動時にシリアライズすることを登録しておく
StormではKryoによるシリアライズを行っているが、Kryoは基本クラスを除き、明示的にシリアライズすることを登録しておいたクラスしかシリアライズを行いません。そのため、Topology起動時にbacktype.storm.Config#registerSerialization(java.lang.Class klass) メソッドを用いて登録しておく必要があります。  

#### Topologyコードにおけるシリアライズ登録方法

    Config stormCconfig = new Config();
    stormCconfig.registerSerialization(CustomEntity.class);

#### yamlファイル記述によるシリアライズ登録方法

    topology.kryo.register:
        - storm.sample.entity.CustomEntity
        - storm.sample.entity.CustomEntity2

シリアライズの設定がうまくいったかどうかは、LocalClusterでWorkerの数を複数に設定すると確認可能です。LocalClusterでもWorkerの数を複数にすればシリアライズは行われるためです。

### declareOutputFieldの設定にずれがある場合Topologyは起動しない

各Spout/Bolt間のイベントをつなぐdeclareOutputFieldにずれがある場合、下記のような例外(InvalidTopologyException)が発生して起動しません。そのため、InvalidTopologyExceptionが発生した場合はdeclareOutputField周りと、Topologyの定義を確認する必要があります。

#### Topology定義不正時例外例（Boltが存在しないフィールドを用いてFieldsGroupingを行っている）

    InvalidTopologyException(msg:Component: [SquareBolt] subscribes from stream: [default] of component [SingleIntValueSpout] with non-existent fields: #{"test"})
    	at backtype.storm.daemon.common$validate_structure_BANG_.invoke(common.clj:158)
    	at backtype.storm.daemon.common$system_topology_BANG_.invoke(common.clj:232)
    	at backtype.storm.daemon.nimbus$fn__3592$exec_fn__1228__auto__$reify__3605.submitTopologyWithOpts(nimbus.clj:909)
    	at backtype.storm.daemon.nimbus$fn__3592$exec_fn__1228__auto__$reify__3605.submitTopology(nimbus.clj:927)

#### Topology定義不正時例外例（Boltが存在しないStreamを読み込んでいる）

    InvalidTopologyException(msg:Component: [SquareBolt] subscribes from non-existent stream: [default] of component [SingleIntValueSpout])
    	at backtype.storm.daemon.common$validate_structure_BANG_.invoke(common.clj:152)
    	at backtype.storm.daemon.common$system_topology_BANG_.invoke(common.clj:232)
    	at backtype.storm.daemon.nimbus$fn__3592$exec_fn__1228__auto__$reify__3605.submitTopologyWithOpts(nimbus.clj:909)
    	at backtype.storm.daemon.nimbus$fn__3592$exec_fn__1228__auto__$reify__3605.submitTopology(nimbus.clj:927)

### 起動はしたものの、Topologyが動作しないときに疑うべき点
とりあえず起動はしたものの、Topologyが動作しない場合に疑うべき点は下記。[公式のトラブルシューティング](https://github.com/nathanmarz/storm/wiki/Troubleshooting)も参照。

* Topologyで使用しているホストは全てnslookupで名前が引ける状態になっているか？  
Stormにおいてはホストは全て名前で管理されます。そのため、名前が引けない場合他プロセスに対してメッセージを送信しても届かないケースが発生します。
* Stormが使用するポートがFirewall等でブロックされていないか？  
Stormでは2181（ZooKeeper）、6723（NimbusThrift）、6700～6703（Workerの通信）、8080（UI）等のポートを使用します。一つでも通信がうまくいかないと動作しません。
* Workerプロセスは全て確保されているか？  
SupervisorのWorkerスロットが足りない場合、Topologyは確保できるだけWorkerプロセスを確保して起動されます。ただし、その場合Spout/Boltが一部しかそろわないケースもあるため、処理が全て実行可能とは限りません。

### Workerプロセス上で使用可能なクラスはSupervisorが読んだJarとTopologyのJarに含まれるクラス

Topologyが動作するWorkerプロセスで使用可能なクラスはSupervisorが読んだJarとTopologyのJarに含まれるクラスのみです。  
そのため、Topologyで追加のライブラリを使用する場合は下記の２つのいずれかを取る必要があります。  

* Supervisorのライブラリとして追加のライブラリを配置する。_（Supervisorへのライブラリ追加はSupervisorを再起動しないと反映されないので注意！）_
* TopologyのJarファイルに依存ライブラリも含める（maven-assembly-pluginのjar-with-dependenciesオプションなどで可能です。）

### StormではAck/Failの機構を使用すると全メッセージを少なくとも１回処理できるが、重複処理は検知できない
StormではKestrel、RabbitMQ、Kafkaといったデータソースと適切に組み合わせると全メッセージを少なくとも１回処理させるという処理保証が可能です。  
ですが、「失敗するか、タイムアウトした場合に再実行」という動作であるため、タイムアウトの場合は処理が継続していても再実行が走る可能性があります。  
そのため、重複処理を防止するためにアプリケーション、またはデータストアにて対処を取る必要があります。

### 


---
layout: post
title: SpoutBolt作成時の注意点
---

このページではSpout/Bolt作成時に気をつける点などをまとめます。

### Spout/Boltのシリアライズ不可フィールドはWorkerプロセスに分散させる際にクリアされる
Spout/BoltはTopologySubmit時に作成されたインスタンスがシリアライズされて各Workerプロセスに配分されるためです。
そのため、コンストラクタで初期化されるフィールドであってもシリアライズ不可なフィールドはWorkerプロセスには持ち越せません。  
逆に言えば、TopologySubmit時にシリアライズ可能なフィールドに値を設定しておけば、Workerプロセスに分散した後でも設定された値を使うことが可能です。

### StormでCollectorに対してemitするクラスは全てシリアライズ化可能とすること
StormのTupleにはObjectが投入可能ですが、Spout/Bolt間の通信時に異なるWorkerプロセスに移動する場合、シリアライズが行われます。
そのため、Tupleにシリアライズ不可のクラスが混じっていると通信時に例外が発生します。Tupleに設定するクラスは全てシリアライズ化可能としておく必要があります。  
Workerプロセスを常時1で確認をしているとわからないため注意が必要です。  
特に、ArrayListのsubListメソッドで作成されるオブジェクトは意外なことにシリアライズ不可だったりします。注意。  

### Spout/Boltが落ちた場合、Spout/Boltが保持していたフィールドの情報は消滅する
Spout/Boltのフィールドに保持された情報はそのSpout/Boltが落ちた場合消滅します。  
自分自身が落ちた場合だけでなく、同じWorkerプロセスに所属するいずれかのSpout/Boltが落ちた場合も一緒に消えます。  
そのため、Spout/Boltに消えて困る情報は保持させず、外部のKVSやDB、またはファイル等に保存する必要があります。

### 終了メソッドはクラスタモードで呼ばれる保証はない
Spoutにはcloseメソッド、Boltにはcleanupメソッドという終了を示すメソッドが存在しますが、これらのメソッドが呼ばれる保証はありません。
そのため、呼ばれることをあてにした実装をするのはNG。リソースのクローズを念のため行っておく位が限度です。  
呼ばれない理由は下記の通りです。
* 1.終了時、WorkerプロセスはSupervisorに「kill -9」として終了される。結果、残りの処理はその時点で実施不可となるため
* 2.終了メソッドは「その他の全ての終了処理、待ち処理」が全て完了してから呼ばれる関係上、大抵呼ばれる前にタイムアウトしてプロセスごとkillされてしまうため。
尚、終了メソッドが呼ばれた場合は「Shut down executor」というログが出力されるため、判別は可能です。（参考は下記ソース）

#### executor.clj(Storm-0.8.2)

      (shutdown
        [this]
        (log-message "Shutting down executor " component-id ":" (pr-str executor-id))
        (disruptor/halt-with-interrupt! (:receive-queue executor-data))
        (disruptor/halt-with-interrupt! (:batch-transfer-queue executor-data))
        (doseq [t threads]
          (.interrupt t)
          (.join t))
    
        (doseq [user-context (map :user-context (vals task-datas))]
          (doseq [hook (.getHooks user-context)]
            (.cleanup hook)))
        (.disconnect (:storm-cluster-state executor-data))
        (when @(:open-or-prepare-was-called? executor-data)
          (doseq [obj (map :object (vals task-datas))]
            (close-component executor-data obj))) // Spoutであればclose、Boltであればcleanupメソッドが実行される
        (log-message "Shut down executor " component-id ":" (pr-str executor-id)))
        )))

### Spout/BoltでstormConfから取得可能な設定値はstorm.yamlに記述したパラメータとTopologySubmitに指定したパラメータ
Spout/Boltはopen/prepareメソッド実行時にstormConfオブジェクトが渡されます。  
この際取得可能な設定値はstorm.yamlに記述したパラメータとTopologySubmitにConfigオブジェクトに設定したパラメータとなります。  
但し、stormConfから取得できる設定値＝全てシリアライズ可能な設定値のため、Topology起動時に全てSpout/Boltに設定してしまうのがベストです。その方が確認も行いやすいです。

### 標準出力は使用してはいけない
Stormは標準出力をサブプロセスと通信を行うために使用しているという設計記述があります。そのため、標準出力に出力を行った結果、予期せぬ動作を引き起こす可能性があります。  
但し、Topologyを生成する際のプログラムについてはこの制限から除外されます。その時点ではまだ親プロセス／サブプロセスに分割されていないためです。  
StormクラスタにTopologySubmitに失敗したケースなど、標準出力を使った方がわかりやすいケースもあります。

### Ack/Fail機構を使用するにはSpoutでemitする際にMessageIdを指定する必要がある
Stormの売りでもあるメッセージの処理保証機構であるAck/Fail機構は以下のコード例のようにSpoutからTupleをemitする際にMessageIdを指定した場合のみ有効となる。  
指定しない場合はメッセージの処理失敗／タイムアウト等の障害は検知されない。

##### SpoutでTupleをemitする際にMessageIdを指定する例

        private SpoutOutputCollector collector;
        （中略）
        collector.emit(tuple, messageId); // emitメッセージの第２引数としてmessageIdを指定


### Ack/Failを使用する場合、次のBoltにemitしてからackを返す
途中のBoltで単にTupleをackしてしまうとその時点でSpoutにackが返り、最後のBoltまで処理されたかどうかはわからなくなるためです。  
そのため、途中のBoltでは、下記の順で処理を行う必要があります。  
1.受信Tupleをanchorとして次Bolt向けのTupleをemit
2.受信Tupleに対してackする

### Spoutのfailが呼ばれるのはタイムアウトするか、明示的にBolt側でfailメソッドを呼び出した場合。
タイムアウトのケースはまだ処理が正常に続いている場合でもfailが呼ばれてタイムアウト扱いになります。そのため、タイムアウトの時間はTopologyのメッセージ処理時間と相談して決めること。


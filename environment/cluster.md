---
layout: post
title: クラスタ環境
---

このページではクラスタ環境構築の際に気をつける点、制限事項などをまとめます。

### クラスタ環境はLinux系OS上でのみ構築可能、但しクラスタに対してTopologyをSubmitするのはWindowsからで可能
クラスタ環境については今までWindowsで構築出来たという事例は見当たらない。そのため、出来ないと考えるのが妥当？  
但し、Windows上から既に構築されているStormクラスタに対してTopologyをSubmitすることは可能です。

### ZeroMQ、JZMQがJavaから参照できない状況では動作しない
仮にZeroMQ、JZMQがインストールしてあっても、Javaから参照できない状態の場合Workerプロセス起動時に下記のようなエラーが発生し、動作しません。  
```
java.lang.UnsatisfiedLinkError: no jzmq in java.library.path  
        at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1738)  
        at java.lang.Runtime.loadLibrary0(Runtime.java:823)  
        at java.lang.System.loadLibrary(System.java:1028)  
        at org.zeromq.ZMQ.<clinit>(ZMQ.java:34)  
```

### ZeroMQ、JZMQの最新版を導入した場合動作しない
* ZeroMQ、JZMQはバージョンごとに内部動作やAPIの差分がある。そのため、Stormのページに書かれたバージョンを使用する必要があります。
違うバージョンを入れると下記のようなエラーが発生します。  
尚、StormではZeroMQを使用せずに純粋なJavaプロセスとして動作させることも検討ポイントに入っています。  
```
org.zeromq.ZMQException: Invalid argument(0x16)  
        at org.zeromq.ZMQ$Socket.setLongSockopt(Native Method)  
        at org.zeromq.ZMQ$Socket.setLinger(ZMQ.java:601)  
        at zilch.mq$set_linger.invoke(mq.clj:57)  
        at backtype.storm.messaging.zmq.ZMQContext.connect(zmq.clj:33)  
        at backtype.storm.daemon.worker $fn__3066$exec_fn__858__auto____3067$this__3077$iter__3080__3084$fn__3085.invoke(worker.clj: 137)  
        at clojure.lang.LazySeq.sval(LazySeq.java:42)  
```
また、32bitと64bitでインストールするものを誤ると下記のようなエラーが発生します。
```
java.lang.UnsatisfiedLinkError: /usr/lib/libjzmq.so.0.0.0: /usr/lib/libjzmq.so.0.0.0: wrong ELF class: ELFCLASS32 (Possible cause: architecture word width mismatch)  
        at java.lang.ClassLoader$NativeLibrary.load(Native Method)  
        at java.lang.ClassLoader.loadLibrary0(ClassLoader.java:1750)  
        at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1675)  
        at java.lang.Runtime.loadLibrary0(Runtime.java:840)  
        at java.lang.System.loadLibrary(System.java:1047)  
        at org.zeromq.ZMQ.<clinit>(ZMQ.java:34)  
        at java.lang.Class.forName0(Native Method)  
        at java.lang.Class.forName(Class.java:186)  
        at backtype.storm.messaging.zmq$loading__4784__auto__.invoke(zmq.clj:1)  
        at backtype.storm.messaging.zmq__init.load(Unknown Source)  
        at backtype.storm.messaging.zmq__init.<clinit>(Unknown Source)  
       // （省略）  
```

### Nimbus、Supervisor、workerは各々ローカルに一時ファイルを出力する
Stormの構成プロセスであるNimbus、Supervisor、workerは環境変数「STORM_HOME」配下に一時ファイルを出力します。  
そのため、ディスクフルなどでファイルの書き込み読み込みが出来ない場合は動作しません。  

### Storm-UIの起動ポート番号を変更する方法
Storm-UIのポート番号はデフォルト8080だが、ui.port: 1234・・・という形でstorm.yamlを下記のように設定すれば変更可能です。  
そのためTomcat等8080ポートを良く使うサービスが起動しているサーバでも使用することが可能です。  
```
ui.port: 1234
```


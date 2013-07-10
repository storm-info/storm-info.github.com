---
layout: post
title: Worker起動時の動作
---

このページではWorkerプロセス起動時の動作をまとめています。

### Workerプロセスを起動させる際のJVMパラメータ設定方法
SupervisorプロセスがWorkerプロセスを起動させる時のJVMパラメータはstorm.yamlの「topology.worker.childopts」で指定可能です。  
例えば、JVMサイズを拡張したい場合は下記のように設定値を記述すればOKです。  
但し、「topology.worker.childopts」はSupervisorが起動した時に読み込んだ値を使用するため、更新した場合はSupervisorを再起動する必要があります。  

    topology.worker.childopts: "-Xmx2048m"

尚、「topology.worker.childopts」に"%ID%"という記述を含めた場合、Workerの使用するポート番号で置換されて実行されます。  
そのため、下記のように値を指定しておけば各Workerプロセスを「26700、26701・・・」といったポートでリモートデバッグ待ち受けした状態で起動させることが可能です。  

##### Workerリモートデバッグ用設定

    topology.worker.childopts: "-Xmx768m -Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=2%ID%,suspend=n"

尚、上記起動処理のソースコードは下記です。

##### supervisor.clj(Storm-0.8.2)

    (defmethod launch-worker
        :distributed [supervisor storm-id port worker-id]
        (let [conf (:conf supervisor)
              stormroot (supervisor-stormdist-root conf storm-id)
              stormjar (supervisor-stormjar-path stormroot)
              storm-conf (read-supervisor-storm-conf conf storm-id)
              classpath (add-to-classpath (current-classpath) [stormjar])
              childopts (.replaceAll (str (conf WORKER-CHILDOPTS) " " (storm-conf TOPOLOGY-WORKER-CHILDOPTS)) // %ID%をポート番号で置換
                                     "%ID%"
                                     (str port))
              logfilename (str "worker-" port ".log")
              command (str "java -server " childopts
                           " -Djava.library.path=" (conf JAVA-LIBRARY-PATH)
                           " -Dlogfile.name=" logfilename
                           " -Dstorm.home=" (System/getProperty "storm.home")
                           " -Dlog4j.configuration=storm.log.properties"
                           " -cp " classpath " backtype.storm.daemon.worker "
                           (java.net.URLEncoder/encode storm-id) " " (:assignment-id supervisor)
                           " " port " " worker-id)]
          (log-message "Launching worker with command: " command)
          (launch-process command :environment {"LD_LIBRARY_PATH" (conf JAVA-LIBRARY-PATH)})
          ))


---
layout: post
title: TopologySubmit時の動作
---

このページではTopologySubmit時の動作をまとめています。

### TopologySubmit時の動作
* 1.Topology起動プログラムを起動する
    * 直接Javaクラスを起動するか、storm jarコマンドを用いて起動するかの２つのパターンがありますが、内部的には変わりません。
* 2.Topology起動プログラムが設定値（JSON形式）とTopologyJarファイル（指定時）をNimbusに対して投入する。
    * 2-1.Submit時に指定した設定値と起動時のコマンドライン（storm.options）、ローカルのstorm.yamlをマージする。
        * 優先度はstorm.options＞Submit時に指定した設定値＞ローカルのstorm.yaml
    * 2-2.マージした設定値をJSON形式に変換する。
    * 2-3.Submitする予定のTopologyと同名のTopologyが存在しないか確認
        * 存在する場合はSubmitエラーとなる。
    * 2-4.TopologyJarファイルをNimbusのThriftインタフェースを用いて投入する
        * 直接Javaクラスを起動する場合は「-Dstorm.jar」として指定したパスのJarファイルが使用される。
        * storm jarコマンドを使用した場合はその際に指定したJarファイルが使用される。
    * 2-5.Topology起動情報をNimbusのThriftインタフェースを用いて投入する
        * 投入するのは以下の5個
            *A.Topology名称
            *B.TopologyJarファイルパス
            *C.JSON形式の設定値
            *D.StormTopology（TopologyBuilderで生成したTopology定義）
            *E.Topology起動オプション（指定時のみ）
        * StormTopologyにシリアライズしたSpout/Boltが含まれているため、起動プログラムで設定した値が保持される。


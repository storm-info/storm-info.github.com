---
layout: post
title: TopologySubmit時の動作
---

このページではTopologySubmit時の動作をまとめています。

### TopologySubmit時の動作（クライアント）
* 1.Topology起動プログラムを起動する
    * 直接Javaクラスを起動するか、storm jarコマンドを用いて起動するかの２つのパターンがありますが、内部的には変わりません。
* 2.Topology起動プログラムが設定値（JSON形式）とTopologyJarファイルをNimbusに対して投入する。
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

### TopologySubmit時の動作（Nimbus）
* 1.TopologyJarファイルをクライアントプログラムから受信する
    * 受信先パスはStormインストール先のnimbus/inbox 配下
* 2.Topology起動情報をクライアントプログラムから受信する
    * 2-1.受信したTopology起動情報を検証する
        * Topology名称が不正ではないか
        * 既に同名のTopologyが起動していないか
        * Topology定義が不正ではないか（ユーザ定義のITopologyValidatorによる検証）
    * 2-2.NimbusのTopology起動カウントをインクリメント
        * 起動カウントを通し番号としてTopologyIDが決定される
    * 2-3.Topology起動用の情報を初期化する
        * 
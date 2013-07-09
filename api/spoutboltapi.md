---
layout: post
title: SpoutBoltで使用できるAPI
---

このページではSpoutBoltで使用できるAPIについてまとめます。

### Task内部で自分が何番目のTaskかを知る方法
* TopologyContext#getThisTaskId()＝Task全体での通し番号が取得できます。  
例）TaskAの並列度が3、TaskBの並列度が3の状況でTaskBで実行すると3～5の値が取得できる。

##### Spoutでの例

    public void open(Map conf, TopologyContext context, SpoutOutputCollector collector)
    {
        // Task全体での通し番号を取得
        int allIndex = context.getThisTaskId();

##### Boltでの例

    public void prepare(Map conf, TopologyContext context, OutputCollector collector)
    {
        // Task全体での通し番号を取得
        int allIndex = context.getThisTaskId();

* TopologyContext#getThisTaskIndex()＝コンポーネント内での通し番号が取得できます。  
例）TaskAの並列度が3、TaskBの並列度が3の状況でTaskBで実行すると0～2の値が取得できる。

##### Spoutでの例

    public void open(Map conf, TopologyContext context, SpoutOutputCollector collector)
    {
        // コンポーネント内での通し番号を取得
        int taskIndex = context.getThisTaskIndex();

##### Boltでの例

    public void prepare(Map conf, TopologyContext context, OutputCollector collector)
    {
        // コンポーネント内での通し番号を取得
        int taskIndex = context.getThisTaskIndex();


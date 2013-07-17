---
layout: post
title: Spoutコード例
---

このページではSpoutを作成する際のコード例やリンクについてまとめます。

### StormとKestrelを組み合わせることで『Kestrelから取得したメッセージをStormで処理完了したことを保証する』アプリケーションが構築可能
Kestrelは「Transaction」という仕組みを持っており、ackが返されなかったメッセージは一定時間でメッセージが復旧されます。そのため、Storm側で成功したらackを返し、失敗／タイムアウトしたらfailを返すようにKestrelとやり取りをすれば容易に処理保証を行うアプリケーションが構築可能です。  
上記の動作をする雛型は[Storm-Kestrel](https://github.com/nathanmarz/storm-kestrel)のKestrelThriftSpoutにあります。



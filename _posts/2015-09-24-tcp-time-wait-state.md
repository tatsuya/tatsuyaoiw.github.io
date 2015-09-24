---
layout: post
title: TCP ConnectionのTIME-WAIT Stateについて調べたときに参考にしたリンクたち
date: 2015-09-24 21:22
---

- [RFC 793 - Transmission Control Protocol](https://tools.ietf.org/html/rfc793)

TCPの仕様。全部読んだら詳しくなれるはず。

- [Transmission Control Protocol - Wikipedia, the free encyclopedia](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)

WikipediaのTCPのページ。概要を掴むならまずはここから。

- [TIME_WAIT and its design implications for protocols and scalable client server systems - AsynchronousEvents](http://www.serverframework.com/asynchronousevents/2011/01/time-wait-and-its-design-implications-for-protocols-and-scalable-servers.html)

今日はこれを読んだ。TIME-WAIT Stateがそもそもなぜ存在するのかというところを、TCPにおける通信の仕組みから順番に解説している。

TIME-WAITが存在する理由は、あるTCPのセグメントの送信に遅延が発生してしまった場合に後続の別のコネクションに影響が出てしまうことを防ぐためである。ただし、TIME-WAITがなくとも、実際に後続のコネクションに悪影響を及ぼすケースは非常に稀と言って良い。なぜなら、まず後続のコネクションの送信元ホストとポート、送信先ホストとポートの組み合わせが、当該コネクションのそれと同じでなければならない。通常、クライアントのポートはOSによってEphemeral Port Rangeから自動的に選択されるため、複数のコネクションが同じポートを使用する確率はその分少なくなる（多くのLinux KernelはEphemeral Portに32768〜61000を使用している）。さらに、遅延したTCPセグメントのシーケンス番号が、後続のコネクションにおいても有効である必要がある。これらの条件をすべて満たしてはじめて影響が出ることになるが、TIME-WAITの存在によってこの可能性を排除できる。


- [ubuntu - How to reduce number of sockets in TIME_WAIT? - Server Fault](http://serverfault.com/questions/212093/how-to-reduce-number-of-sockets-in-time-wait)

TIME-WAITにまつわる問題の一つに、並列処理による多数のTCPコネクションが実際の処理が完了しているにも関わらずTIME-WAIT Stateのまま残存してしまい、ローカルのEphemeral Portを使い果たしてしまう状況がある。

- [linux/tcp.h at dd5cdb48edfd34401799056a9acf61078d773f90 · torvalds/linux](https://github.com/torvalds/linux/blob/dd5cdb48edfd34401799056a9acf61078d773f90/include/net/tcp.h#L117)

TIME-WAITの長さはLinux Kernelにハードコードされている（60秒）ため設定ファイルから変更することはできない。

推奨されている2つのKernelのオプションが以下。

```
tcp_tw_recycle - BOOLEAN
        Enable fast recycling TIME-WAIT sockets. Default value is 0.
        It should not be changed without advice/request of technical
        experts.

tcp_tw_reuse - BOOLEAN
        Allow to reuse TIME-WAIT sockets for new connections when it is
        safe from protocol viewpoint. Default value is 0.
        It should not be changed without advice/request of technical
        experts.
```

- [Coping with the TCP TIME-WAIT state on busy Linux servers | Vincent Bernat](http://vincent.bernat.im/en/blog/2014-tcp-time-wait-state-linux.html)

上記のよく似ている二つのオプションを解説しているサイト。まだ読んでない。

- [Linuxカーネルの「TCP_TIMEWAIT_LEN」変更は無意味？ | スラド Linux](http://linux.srad.jp/story/15/09/09/0648258/)

> tcp_tw_reuse をセットすると１秒で TIME_WAITのsocketは再利用が始まる

なるほど。

- [TCP/IPの送信用ポート範囲を変更する | b.l0g.jp](http://b.l0g.jp/linux/ip-local-port-range/)
- [TCP/IP で TIME_WAIT が残る時間を短くする](http://network.station.ez-net.jp/server/linux/network/time_wait.asp)
- [ローカルポートを食いつぶしていた話 - ダウンロードたけし（寅年）の日記](http://d.hatena.ne.jp/download_takeshi/20091013/1255443592)
- [TCPのTIME_WAITを無くす！ | 田舎に住みたいエンジニアの日記](http://blog.ybbo.net/2013/05/27/tcp%E3%81%AEtime_wait%E3%82%92%E7%84%A1%E3%81%8F%E3%81%99%EF%BC%81/)

その他、実際のトラブルシューティングに関する情報や設定方法など。

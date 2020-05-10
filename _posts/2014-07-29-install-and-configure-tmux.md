---
layout: post
title: tmuxのインストールと設定
date: 2014-07-29 21:18
author: Tatsuya Oiwa
tags: [Japanese]
---

[tmux](http://tmux.sourceforge.net/)使い始めたので一応書いとく。

MacOSの場合、[Homebrew](http://brew.sh/)でインストール。

```
$ brew install tmux
```

tmuxの起動。

```
$ tmux
```

画面を左右二つに分割。

```
Ctrl-b %
```

画面を上下二つに分割。

```
Ctrl-b "
```

画面間の移動。

```
Ctrl-b o
```

tmuxはSession>Window>Paneという構造になっていて、上記の画面とは最小単位であるPaneを指す。

Windowは複数のPaneを持つことができ、Sessionは複数のWindowを持つことができる。

新しいWindowの作成。

```
Ctrl-b c
```

Window間の移動。

```
Ctrl-b n  # next
Ctrl-b p  # previous
```

SessionはDetach/Attach（バックグラウンド/フォアグラウンドへ移動）することができる。

```
Ctrl-b d  # detach
tmux attach  # attach
```

セッションの作成。

```
tmux new -s [session-name]
```

セッションの一覧を表示。

```
tmux list-sessions
```

その他、利用可能なコマンドを表示。

```
Ctrl-b ?
```

また、tmuxは`~/.tmux.conf`という名前の設定ファイルを自動で読み込む。

`.tmux.conf`の例。

```
# remap prefix from default Ctrl-b to Ctrl-t
set -g prefix C-t
unbind C-b
bind C-t send-prefix
```

あと、tmuxの画面内でスクロールしたい場合は、コピーモードに切り替える必要がある。コピーモードでは矢印キーやマウスによるスクロールの他に、テキストのコピー/ペーストやその他様々な操作を行うことができる。

コピーモードの開始。

```
Ctrl-b [
```

コピーモードの終了。

```
q
```

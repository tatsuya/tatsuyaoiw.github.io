---
layout: post
title: iTunes UでStanfordのProgramming Methodologyの授業観てる
date: 2015-09-07 10:10
---

今、第3回の講義見終わったのでそのメモ忘れないうちに書いておく。

## Break down approach & Bottom up approach

プログラミングで何か問題を解決しようとするとき、大きく2つのアプローチがある。ひとつはBreak down approach、もうひとつはBottom up approachだ。

Break down approachというのは、ある大きな問題をより粒度の小さい複数の問題へと分解していき、それらを一つ一つ解決していくことで最終的に大きな問題を解決する方法のことで、プログラミングにおいてはこのBreak down approachをとるのが良いとされている。

例えば「朝起きてから出かける準備をする」という現実の問題を考えたとき、実際に必要な準備をより小さなタスクへと分解していくと、だいたい以下のようになる。

1. ベッドから起きる
2. 歯を磨く
3. シャワーを浴びる

さらに、例えば2の「歯を磨く」というタスクは次のように分解できるだろう。

2-1. 歯ブラシをとる
2-2. 歯磨き粉をつける
2-3. 歯ブラシを歯の上で動かす

プログラミングもこれと同じで、解決したい一番大きな問題を最初に最上位のメソッドとして定義しておき、そのメソッドの本体により小さな問題を解決するメソッドのCallerを先に書いてしまう。

```
public void prepareToGoOut() {
  wakeUp()；
  brushTeeth();
  takeShower();
}
```

このとき、まだ`wakeUp()`、`brushTeeth()`、`takeShower()`の各メソッドは実装されていない。IDEを使っていると、`wakeUp()`、`brushTeeth()`、`takeShower()`の下には赤い波線か何かで "Method not implemented!"などとWarningが出るかもしれないが焦ってはいけない。

次に、先に定義していおいたCallerの実装を書く。その際、例えば`brushTeeth()`の実装が大きすぎるなと感じたら、またさらにそこから小さなメソッドへと分解していき、これを繰り返す。

```
public void prepareToGoOut() {
  wakeUp()；
  brushTeeth();
  takeShower();
}

private void wakeUp() {
  ...
}

private void brushTeeth() {
  takeBrush();
  takeToothpaste();
  moveOnTooth();
}

private void takeShower() {
  ...
};
```

Bottom up approachは、Break down approachとは対になる考え方で、最初に小さな単位のメソッドを実装してから徐々に大きな問題への解決へと向かっていく方法である。プログラミング初学者は、最初どうしてもこのBottom up approachになってしまうらしいが、訓練することでだんだんとBreak down approach的な考え方ができるようになってくるそうだ。

## Do one thing

Unix Philosophyや他にも色々なところで言われているが、各メソッドは基本的に一つの問題を解決すべきである。

また、各メソッドは目安としてだいたい1~15行程度におさまるように書く。メソッドの実装がそれ以上の場合、複数の問題を解決していないか疑うべきである。


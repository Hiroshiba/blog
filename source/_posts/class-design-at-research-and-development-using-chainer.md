---
title: Chainerを使った研究開発時のクラス設計
permalink: class-design-at-research-and-development-using-chainer
s: class-design-at-research-and-development-using-chainer
date: 2017-12-23 04:14:42
tags:
categories:
description: この記事はChainer Advent Calendar 2017の23日目の記事です。僕は普段、Chainerを使って研究開発しています。このとき、クラスをどう分けるべきかよく悩みます。いろいろやってみてある程度固まってきたので、自分なりにまとめてみました。
---


この記事は[Chainer Advent Calendar 2017](https://qiita.com/advent-calendar/2017/chainer)の23日目の記事です。

僕は普段、Chainerを使って研究開発しています。
このとき、クラスをどう分けるべきかよく悩みます。
いろいろやってみてある程度固まってきたので、自分なりにまとめてみました。

<!-- more -->

ChainerなどのDeepLearningフレームワークを使う理由は大きく分けて３段階ほどあります。

1. 再現実験
1. 試行錯誤を伴う実験
1. 学習済みモデルを用いたシステムづくり

世の中に転がっているChainerサンプルプログラムは大体(1)のもので、こちらは綺麗にまとまっているものが多いです。
一方で、何か新規に実験していると、どうしても試行錯誤が発生してコードが煩雑になります(2)。
そしてさらには、(1)や(2)で学習したモデルを使ってサービス応用しようとすることもあります(3)。

今回は研究開発用のコード、つまり、サービス応用を考えつつ実験コードを書く際に、どうクラスを切っていくべきか考えをまとめます。

-----

まず、細かいのも含めると、実験コードには次の構成要素があります。

* DataProcess : 入力・出力データの加工する
* Dataset : データをChainer用にまとめる
* Network : 汎用のニューラルネットワーク
* Loss : 損失の取り回し
* Model : ニューラルネットワーク全体
* Updater : モデルの更新（＋データの取り回し）
* Trainer : 便利モジュールとの連携

規模や実験内容に応じて`DataProcess`は`Dataset`に、`Network`と`Loss`は`Model`にまとめることもあります。
このうち、学習済みモデルを用いたサービスを作る際に必要なのは、`DataProcess`と`Model`だけです。

それぞれに関して、なんなのか、なぜそれが必要か、どういうときに必要かを書きます。

--------

## DataProcess
入力データや出力データを加工する関数、もしくは呼び出し可能なオブジェクトです。
画像を読み出す、クロップする、線画化する、などなど。
これらのデータ処理は、**`DatasetMixin`オブジェクトの`get_example`メソッドに書くこともできますが、こうしてしまうとあとで流用する際にそのオブジェクトの構造を意識する必要が出てきます。**
例えば１枚の画像を加工したいだけでも、`DatasetMixin`オブジェクトを作成し、`get_example(0)`しなければいけません。
最初からデータを加工する関数を切り出しておけば、後で簡単に流用できます。

## Dataset
データをChainer用にまとめるクラスです。`DatasetMixin`を継承して作るのが一般的です。
`DataProcess`にも書いたとおり、**ここに記述した処理は後で流用しづらいので、なるべく簡単なことしか書かないほうが良い**と思います。
僕は`DataProcess`を１つだけ受け取ってデータ加工する`Dataset`クラスをよく使っています。

{% codeblock lang:python line_number:false %}
class Dataset(chainer.dataset.DatasetMixin):
  def __init__(self, inputs, data_process):
    self._inputs = inputs
    self._data_process = data_process

  def __len__(self):
    return len(self._inputs)

  def get_example(self, i):
    return self._data_process(self._inputs[i])
{% endcodeblock %}

## Network
汎用のネットワークを書きます。簡単なモデルの場合はなくても良いと思います。
僕はよくBatchNormalizationとConvolution2Dをまとめたのを流用しています。

{% codeblock lang:python line_number:false %}
class BNConvolution2D(chainer.link.Chain):
  def __init__(self, in_channels, out_channels, ksize, stride=1, pad=0, **kwargs):
    super().__init__()
    with super().init_scope():
      self.conv = chainer.links.Convolution2D(in_channels, out_channels, ksize, stride, pad, nobias=True, **kwargs)
      self.bn = chainer.links.BatchNormalization(out_channels)

  def __call__(self, x):
    return chainer.functions.relu(self.bn(self.conv(x)))
{% endcodeblock %}

## Loss
損失関数を実装します。簡単なモデルの場合はなくても良いと思います。
Chainerの`Trainer`とloss周りの扱いはややこしく、`chainer.report`を使ったりする必要があります。
**`Loss`クラスの書き方は[chainer.links.Classifier](https://github.com/chainer/chainer/blob/4ce120d09b6543ae60a6d18830b4345992f1322d/chainer/links/model/classifier.py)がとても参考になります。**
コンストラクタで`Model`オブジェクトを受け取って`__call__`でフォワードし、得られた出力を元にlossを作ってreturnする設計です。
**`Loss`クラスが必要になるのはモデルが２種類以上あるとき**です。DCGANなどのタスクでは`Loss`クラスを作って、生成器と判別器用のlossを返すと綺麗にコードが書けます。

## Model
ニューラルネットワークをまとめたクラスです。
**`Optimizer`１つにつき`Model`１つ**と考えると理解しやすいです。
`chainer.link.Chain`や`chainer.link.ChainList`を継承して書くのが一般的です。

## Updater
こいつがむちゃくちゃしんどいです。
`Model`が１つしかなければ`chainer.training.StandardUpdater`を使うと大体うまく行きます。
`Model`が複数ある場合、`StandardUpdater`を継承した`Updater`クラスを自分で定義し、データの流れとモデルの更新を自分で書く必要があります。
[DCGANのサンプル実装](https://github.com/chainer/chainer/blob/7d0d6e70aab9763727802e2a8524744687e9086d/examples/dcgan/updater.py#L10)でちょっと雰囲気がつかめると思います。
`Loss`クラスをうまく切り出せてさえいればある程度綺麗に書けます。

## Trainer
Chainerが用意した学習用のクラスです。
`Updater`や`Model`を与えるとよしなに色々やってくれます。
これに関してはいろんな記事があるので説明は割愛します。

------

これらの方式で実験コードを書くと、ある程度煩雑になってきても大規模な改修は発生しづらくなります。
また、`DataProcess`と`Model`をライブラリ化すれば、サービス応用も比較的簡単に行なえます。

Chainerは柔軟でいろんなクラス設計が可能です。
試行錯誤を伴う実験をしていてもコードが散らばらないような設計があれば、ぜひ教えてください。
開発方針を自分の中で持っておいて、どんどん研究開発していきたいものです。

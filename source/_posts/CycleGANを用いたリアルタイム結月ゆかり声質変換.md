---
title: CycleGANノンパラレル結月ゆかり声質変換やってみた
permalink: became-yuduki-yukari-with-cycle-gan-power
date: 2018-04-22 08:21:21
tags:
categories:
description: 自分の声を結月ゆかりにしたい。前回はパラレルデータのアライメントが問題になったので、ノンパラレルデータの手法を試したい。CycleGANを使ったノンパラレル声質変換を試みた。アライメントしなくても聞き取れる音声が生成できた。しかし、言語性を保ちつつ声質変換できるパラメータは見つけられなかった。CycleGANを用いて性能の良い声質変換を得るのは難しいと思った。Identity以外のお手頃な制約手法が見つかれば，また挑戦してみたい。
---

### 目次

* （背景）自分の声を結月ゆかりにしたい。前回はパラレルデータのアライメントが問題になったので、ノンパラレルデータの手法を試したい。
* （手法）CycleGANを使ったノンパラレル声質変換を試みた。
* （結果）アライメントしなくても聞き取れる音声が生成できた。しかし、言語性を保ちつつ声質変換できるパラメータは見つけられなかった。
* （考察）CycleGANを用いて性能の良い声質変換を得るのは難しいと思った。Identity以外のお手頃な制約手法が見つかれば，また挑戦してみたい。

この記事は、技術系同人誌[SIGNICO vol.5](http://signico.hi-king.me/)の掲載記事「CycleGANを用いたリアルタイム結月ゆかり声質変換」の結果音声を中心に載せています。
詳しい手法や解説などは同人誌の記事をご参照ください。

<!-- more -->

### 結果

#### フィルタ幅を変えた時の結果

<figure>
  <figcaption>「あらゆる現実を、全て、自分の方へ捻じ曲げたのだ。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅5ms</th>
      <td><audio src="result-1-1-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅15ms</th>
      <td><audio src="result-1-2-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅1.28s</th>
      <td><audio src="result-1-3-A01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「予防や、健康管理、リハビリテーションのための制度を、充実していく必要があろう。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅5ms</th>
      <td><audio src="result-1-1-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅15ms</th>
      <td><audio src="result-1-2-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅1.28s</th>
      <td><audio src="result-1-3-B01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「六百人のお客さんの人いきれに、むし暑くて、扇子を使わずにいられない。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅5ms</th>
      <td><audio src="result-1-1-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅15ms</th>
      <td><audio src="result-1-2-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅1.28s</th>
      <td><audio src="result-1-3-C01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「十分間の休憩を与えられ、乱れた髪を結い直し、肩の汗をぬぐって、支度部屋で呼吸を整える。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅5ms</th>
      <td><audio src="result-1-1-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅15ms</th>
      <td><audio src="result-1-2-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅1.28s</th>
      <td><audio src="result-1-3-D01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「家に来た年賀状は、三百枚ほどで、丁度、出した分と同じぐらいだ。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅5ms</th>
      <td><audio src="result-1-1-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅15ms</th>
      <td><audio src="result-1-2-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>フィルタ幅1.28s</th>
      <td><audio src="result-1-3-E01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>

#### 損失の比率を変えたときの結果

<figure>
  <figcaption>「あらゆる現実を、全て、自分の方へ捻じ曲げたのだ。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-2-1-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:10</th>
      <td><audio src="result-2-2-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:100</th>
      <td><audio src="result-2-3-A01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「予防や、健康管理、リハビリテーションのための制度を、充実していく必要があろう。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-2-1-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:10</th>
      <td><audio src="result-2-2-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:100</th>
      <td><audio src="result-2-3-B01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「六百人のお客さんの人いきれに、むし暑くて、扇子を使わずにいられない。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-2-1-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:10</th>
      <td><audio src="result-2-2-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:100</th>
      <td><audio src="result-2-3-C01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「十分間の休憩を与えられ、乱れた髪を結い直し、肩の汗をぬぐって、支度部屋で呼吸を整える。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-2-1-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:10</th>
      <td><audio src="result-2-2-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:100</th>
      <td><audio src="result-2-3-D01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「家に来た年賀状は、三百枚ほどで、丁度、出した分と同じぐらいだ。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-2-1-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:10</th>
      <td><audio src="result-2-2-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:100</th>
      <td><audio src="result-2-3-E01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>

#### Identity制約を追加した結果

<figure>
  <figcaption>「あらゆる現実を、全て、自分の方へ捻じ曲げたのだ。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-3-1-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.1</th>
      <td><audio src="result-3-2-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.02</th>
      <td><audio src="result-3-3-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.01</th>
      <td><audio src="result-3-4-A01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.002</th>
      <td><audio src="result-3-5-A01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「予防や、健康管理、リハビリテーションのための制度を、充実していく必要があろう。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-3-1-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.1</th>
      <td><audio src="result-3-2-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.02</th>
      <td><audio src="result-3-3-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.01</th>
      <td><audio src="result-3-4-B01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.002</th>
      <td><audio src="result-3-5-B01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「六百人のお客さんの人いきれに、むし暑くて、扇子を使わずにいられない。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-3-1-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.1</th>
      <td><audio src="result-3-2-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.02</th>
      <td><audio src="result-3-3-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.01</th>
      <td><audio src="result-3-4-C01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.002</th>
      <td><audio src="result-3-5-C01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「十分間の休憩を与えられ、乱れた髪を結い直し、肩の汗をぬぐって、支度部屋で呼吸を整える。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-3-1-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.1</th>
      <td><audio src="result-3-2-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.02</th>
      <td><audio src="result-3-3-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.01</th>
      <td><audio src="result-3-4-D01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.002</th>
      <td><audio src="result-3-5-D01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>
<figure>
  <figcaption>「家に来た年賀状は、三百枚ほどで、丁度、出した分と同じぐらいだ。」</figcaption>
  <table>
    <tr>
      <th>入力</th>
      <td><audio src="hiho-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>目標</th>
      <td><audio src="E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:1</th>
      <td><audio src="result-3-1-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.1</th>
      <td><audio src="result-3-2-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.02</th>
      <td><audio src="result-3-3-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.01</th>
      <td><audio src="result-3-4-E01.mp3" controls></audio></td>
    </tr>
    <tr>
      <th>1:0.002</th>
      <td><audio src="result-3-5-E01.mp3" controls></audio></td>
    </tr>
  </table>
</figure>

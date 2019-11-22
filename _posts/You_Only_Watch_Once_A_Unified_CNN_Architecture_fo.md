---
layout: post
title: "Reading: YOWO"
date: 2019-11-22
comments: true
categories: reading
---

# You Only Watch Once: A Unified CNN Architecture for Real-Time Spatiotemporal Action Localization
Köpüklü O Wei X Rigoll G 

## Abstract
- Spatiotemporal Action localization
    - 前のフレーム系列からの時系列的情報 + キーフレームからの空間的情報
    - 従来手法はこれらを別々に抽出してfusion.
    - 提案手法では1つのNNだけで時系列的・空間的情報を同時に抽出し，bboxと行動カテゴリの確率を推定できる．
        - end-to-endに最適化される．
- リアルタイムで高精度:
    - 16フレームの入力に対し34fps, 8フレームの入力に対し62fpsで処理．
    - J-HMDB-21(71.1→74.4%), UCF101-24(75.0→87.2%)でSOTA.
![task: 時系列データ中でbboxと行動クラスを推定する．](https://paper-attachments.dropbox.com/s_CAD7B9EB9BCD94E01796BF03456C061D8447B4F268998E90AA225519C25FFC2B_1574264760873_Screen+Shot+2019-11-21+at+0.45.31.png)

![2D-CNN + 3D-CNN → CFAM → Class + bbox](https://paper-attachments.dropbox.com/s_CAD7B9EB9BCD94E01796BF03456C061D8447B4F268998E90AA225519C25FFC2B_1574264897234_image.png)

![Channel Fusion and Attention Module](https://paper-attachments.dropbox.com/s_CAD7B9EB9BCD94E01796BF03456C061D8447B4F268998E90AA225519C25FFC2B_1574264978607_image.png)

## Method
- 3D CNN (3D-ResNext-101)で時系列情報，2D CNN (Darknet-19)でキーフレームからの空間的情報を同時に抽出．
    - どんなアーキテクチャでもよい．（柔軟）
- Feature aggregation (CFAM)
    - A: サイズをあわせた2つの情報をチャンネル方向に結合→両方の情報をもつ特徴．
        - 謎の結合すぎるので，チャンネル間の依存関係を強調するようなアテンション機構を提案．
        - B(CxN行列)とその転置行列の行列積→Gram matrix: チャンネル間の特徴の相関を表す．→softmax→これがアテンションのフィルタになる．
        - このアテンションのフィルタとB(CxN)のアダマール積をとる(アテンション)
        - これと元の重み付き和（学習によってその重みを最適化）を計算→C
    - 最初と最後のConv.によって，異なるチャンネルが異なる分布をもつようになるらしい．
- Bbox regression
    - YOLOと同じかんじ．
    - $$[(5\times (NumCls + 5) \times H' \times W']$$
    - 5 prior anchors
    - 4 coordinates (x, y, h, w) + confidence score(物体らしさスコア)


## Experiments
![赤: gt, 緑: true positive, 橙: false positive](https://paper-attachments.dropbox.com/s_CAD7B9EB9BCD94E01796BF03456C061D8447B4F268998E90AA225519C25FFC2B_1574269140690_image.png)

- Dataset
    - UCF101-24: subset of UCF101. 24 classes, 3207 videos, spatiotemporal annotation
        - 1つの動画に複数の行動やだ→最初の方だけ使用(よくある話)
    - J-HMDB-21: subset of HMDB-51, 21 classes, 928 videsos
- Evaluation Metrics
    - frame-mAP: 各フレームの検出のAUCから計算．
    - video-mAP: 
- 単に2Dと3Dを結合するのではなくCFAMを使ったからうまくいった（すごくスコアが上がっている）．
- Localizationは3Dより2Dのほうが良い．しかし，CFAMを使うともっとよい．
- 軽量なモデルにすると，結果がかなり落ちる．
    - このへんは課題．あるいは量子化したときにどうなるかとか検証してみたい．
    - しかし，速度は十分そう．
- drawbacks
    - false positive detection before the action are performed.
        - キーフレームとクリップのすべての情報を使うので．
    - need enough temporal content to make correct action localization
        - フレーム数固定なので，行動が最初の方になかったらfpにつながる．


## 雑感
- SlowFastをaction recogに使ったやつと比較したい．
    - これよりもう少しスマートな気がする．
    - Weight-share的な意味で．
- CFAMはいいアイディア．
- リアルタイム論文多くなってきた．


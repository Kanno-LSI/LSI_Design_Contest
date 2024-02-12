# FPGA Development

## オートエンコーダを用いた災害ハザードマップ予測回路

## 目次
- [概要](#概要)
  - [目的](#目的)
  - [システム内容](#システム内容)
  - [チーム内の役割](#チーム内の役割)
- [オートエンコーダとは](#オートエンコーダとは)
- [FPGA評価ボードの紹介](#FPGA評価ボードの紹介)
- [回路設計](#回路設計)
- [実機による動作検証](#実機による動作検証)
- [参考文献](#参考文献)

## 概要
FPGAのZynq ZCU104を用いた災害ハザードマップシステムです。

標高情報からオートエンコーダを介して危険地帯の予測を行い、地形データと合わせてハザードマップの3D表示を行います。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/a98ee72b-df89-44f0-9c5e-6640cc0617d4" width="500">
    <br> <!--改行-->
    <b>3Dハザードマップ</b> <!--テキスト表示-->
</div>

### 目的

研究で使用し始めたFPGAの開発方法を学ぶとともに、チーム開発の一連の流れを掴むことを目的として開発を行いました。

### システム内容

PL部：オートエンコーダによる計算回路(今回のメイン)

PS部：ハザードマップの3D表示用アプリケーション

### チーム内の役割
今回は3人チームでの開発を行っています。それぞれの役割は以下の通りです。

| ヘッダー1 | ヘッダー2 |
|-----------|-----------|
|-----------|-----------|
| 行1, 列1  | 行1, 列2  |
| 行2, 列1  | 行2, 列2  |
| 行3, 列1  | 行3, 列2  |

## オートエンコーダとは

## FPGA評価ボードの紹介

## 回路設計

## 実機による動作検証

## 参考文献





#### 〇　Function
・PWM制御

周波数を制御し、音色を変更します。


・外部入力割り込み

使用しているマイクロプロセッサ(ATtiny84A)の入力仕様は、外部から信号を入力されると、進行中の動作を一度中断して関数を実行するようになっています。その特殊な特性に合わせて曲の種類変更に利用しています。LEDにも信号を送ることで、次の曲の種類が分かるようにする機能（曲予約）に対応しました。


・AD変換、タイマ割り込み

AD変換を利用するとボリューム抵抗の回し具合に応じた数値を返すため、この機能を曲の速度変更に使用しています。また、タイマ割り込み内でAD変換を行うことで、リアルタイムで曲の速度を変更できます。


・ボリューム抵抗（ブレッドボード上）

圧電サウンダ（スピーカー）に加わる電圧を変化させ、音量を変更します。


### Requirement
マイクロプロセッサ(ATtiny84A)のファームウェア作成・書き込み環境

電子回路設計


### References
ATtiny84Aデータシート(参照：2023,05,21)

https://ww1.microchip.com/downloads/en/DeviceDoc/ATtiny24A-44A-84A-DataSheet-DS40002269A.pdf


### Demo


# FPGA Development
FPGAによるシステム開発です。2023年10月から2024年1月まで取り組んだチーム開発の内容を説明しています。
<br><br><br><br>
## オートエンコーダを用いた災害ハザードマップ予測回路

## 目次
- [1.概要](#1-概要)
  - [目的](#〇-目的)
  - [システム内容](#〇-システム内容)
  - [チーム内の役割](#〇-チーム内の役割)
- [2.オートエンコーダ](#2-オートエンコーダ)
- [3.FPGA評価ボードの紹介](#3-FPGA評価ボードの紹介)
- [4.回路設計](#4-回路設計)
- [5.実機による動作検証](#5-実機による動作検証)
- [6.参考文献](#6-参考文献)

回路設計をメインに書いていますので、完成したシステムだけ見たい場合は「1.概要」の「システム内容」と、「5.実機による動作検証」の「」

## 1 概要
FPGAのZynq ZCU104を用いた災害ハザードマップシステムです。

標高情報から、機械学習の１つである「オートエンコーダ」を介して危険地帯の予測を行い、地形データと合わせてハザードマップの3D表示を行います。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/a98ee72b-df89-44f0-9c5e-6640cc0617d4" width="500">
    <br> <!--改行-->
    <b>3Dハザードマップ</b> <!--テキスト表示-->
</div>

### 〇 目的

研究で使用し始めたFPGAの開発方法を学ぶとともに、チーム開発の一連の流れを掴むことを目的として開発を行いました。

### 〇 システム内容

PL部：オートエンコーダによる計算回路(今回のメイン)

PS部：ハザードマップの3D表示用アプリケーション

### 〇 チーム内の役割
今回は3人チームでの開発であり、筆者はチームのリーダーです。開発における役割は以下の通りです。

| メンバー | 役割 |
|-----------|-----------|
| メンバー1 | データセット作成 (Python) <br> オートエンコーダの学習部 (Python) |
| メンバー2  | オートエンコーダ推論部の回路 (VHDL) <br> 計算部分の回路シミュレーション (VHDL)|
| 筆者  | 通信回路、ステートマシン (VHDL) <br> 回路全体のコーディング <br> システムアプリケーション (Python) <br> 実機動作|

## 2 オートエンコーダ
### オートエンコーダとは
オートエンコーダ（自己符号化器）とは、ニューラルネットワークの１つです。入力層と出力層の次元数が同一であり、入力情報を一旦圧縮し、そこから同じ次元数まで復元を行うネットワークとなっています。オートエンコーダは、入力データを圧縮して特徴量を抽出する「エンコーダ」と、圧縮された特徴量データを元のユニット数まで復元する「デコーダ」に分けられます。

![ニューロン](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/574b6c1d-e1b0-461e-acb2-2a2b162bbe7a)

オートエンコーダの応用例としてはコーデックやノイズ除去、画像補間などが挙げられますが、今回は**標高データから危険地帯を予測する**ために使用しています。標高データに対して、予測した危険地帯データを付加することでハザードマップを生成することができます。

![地形データからハザードマップ](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/b91edddb-8eba-4a33-aca9-8ac1ebeb7010)

### オートエンコーダの学習
オートエンコーダによる学習をPython上で行うことで、FPGAに実装する最適な重みやバイアスを取得します。重みとバイアス取得までの流れは、標高データ（訓練データ）の作成 ⇒ 教師データ（危険地帯データ）の作成 ⇒ オートエンコーダの学習 ⇒ 重みとバイアス取得　となります。

今回オートエンコーダで用いる標高データの仕様は以下の通りです。
|  |  |
|-----------|-----------|
| エリア面積 | 2000[m] × 2000[m] |
| 地点数 | 32 × 32 |
| 標高値の正規化 | データセットの最大値を255として<br> 0 ~ 255 (8bit) に正規化 |
| データセット数  | 土砂災害用：2128セット <br> 洪水用：672セット <br> 津波用：952セット |


## 3 FPGA評価ボードについて
### FPGAとは
FPGA (Field-Programmable Gate Array)はハードウェア記述言語によって、ユーザーが自由に論理機能を書き換えられるハードウェアです。ASICなどと比べて、ハードウェアの開発期間を大幅に短縮することができます。

### 使用するFPGA評価ボードについて
AMD-Xilinx社製FPGAのZynqを使用しました。本評価ボードはFPGAに加えてSoCを搭載しています。そのため、計算負荷の大きい部分をFPGAによるPL(Programmable Logic)部に、アプリケーションの実装をSoCによるPS(Processing System)部に行うことで、評価ボード1台でシステムを完結させることができます。本評価ボードの詳細は以下の通りです。

|  |  |
|-----------|-----------|
| FPGA | Zynq Ultra Scale+ <br> MPSoC ZCU104 |
| Logic Cell | 504,000 |
| DSP Slice  | 1,728 |
| Bram  | 11[Mb] |
| 開発環境  | Vivado 2023.1 |

![image](https://github.com/Kanno-LSI/FPGA_Development/assets/131650927/ceff9c37-3b67-4db0-9659-75f31c6164e8)


### PS部とPL部の役割
PS部とPL部は以下のような役割になっています。PL部ではオートエンコーダの推論計算、PS部ではハザードマップの3D表示を行っています。

![PS_PL役割](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/14da589a-a4f3-4b9f-8706-3033b8fcbc57)

PS部の仕様は以下の通りです。なお、PS部のOSで使用しているPYNQ(Python Productivity for Zynq)は、AMD-Xilinx社が提供するオープンソースプロジェクトであり、Pythonを用いたアプリケーション開発やPL部とのデータ通信を行えます。

|  |  |
|-----------|-----------|
| CPU | Cortex-A53 <br> 1.5GHz 670 CPU |
| メモリ | 2.0[Gb] |
| OS  | PYNQ　Linux <br> based on Ubuntu 22.04 |
| コンパイラ  | g++ 11.2.0 |
| 使用言語  | Python |

## 4 回路設計
### 全体図
以下に設計回路の概要を示します。この回路は、「PS部-PL部接続回路」、「オートエンコーダ計算回路」に大きく分けられます。
![回路全体](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/aa806be9-347a-4975-8128-1ca3aaa99f61)

### PS部-PL部接続回路
この回路はPS部とPL部のデータのやり取りを行う回路です。PS部からPL部に標高データを入力し、計算後の危険地帯データを出力します。オートエンコーダの計算における入出力のビット幅は8192bitですが、PS部とPL部の間のデータのビット幅は64bitであるため、データ形式の調整を行っています。

#### AXI-4 通信回路
PS部とPL部の通信にはAXI-4(Advanced eXtensible Interface 4)という通信用の規格を利用しています。AXI規格は、ARM社が仕様を策定したバスプロトコルであり、開発者側は共通のインタフェースを用いることで容易に接続することができます。AXI-4は以下のような種類があります。

| 規格名 | バースト転送 | アドレス指定 | 制御のしやすさ |
|-----------|-----------|-----------|-----------|
| AXI-4 | 〇 | 〇 | × |
| AXI-4 Stream | 〇 | × | 〇 |
| AXI-4 Lite  | × | 〇 | 〇 |

データ転送の際には、大容量のデータの入出力を行う場合と、複数の制御信号の入出力を行う場合があります。そのため制御のしやすさを考慮すると、大容量データの扱いにはデータの連続転送（バースト転送）が行えるAXI-4 Stream、複数の制御信号の扱いにはアドレス指定で柔軟な制御が行えるAXI-4 Liteが適しています。今回、標高データと危険地帯データにはAXI-4 Stream、PS部との制御にはAXI-4 Liteを使用しました。

![通信回路](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/71363d4b-4914-4d69-9a31-766eb0d2c88f)

具体的なモジュールの中身は編集中

#### 入出力回路

この回路は入力データと出力データのコントロール回路です。オートエンコーダでの処理は32×32地点×8bit = 8192bitの入出力データを処理するため、AXI-4通信回路に合わせて64bitのデータを結合、分離する役割をしています。

![入出力回路](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/50f7e387-8b16-4324-9555-5fd8c8d9f6b2)

#### PS部制御回路
この回路はPS Operation Statusという名称で、PS部からオートエンコーダ回路のモニタリングや制御を行う役割があります。主にモニタリングしている情報は、計算時間と計算終了セット数であり、制御情報としてPS部から、"IDLE"、"WRITE"、"READ"といった入出力の状態を転送します。

### オートエンコーダ計算回路
この回路はオートエンコーダの推論回路です。「ステートマシン」、「エンコーダ回路」、「デコーダ回路」で構成されており、ステートマシンの制御によってエンコーダとデコーダでのパイプラインを可能にしています。
![オートエンコーダ回路](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/f6a30017-f32e-4f31-8280-4a04f736c672)

#### ステートマシン
この回路ではエンコーダとデコーダの状態遷移を管理して、適切なタイミングでデータの入出力を行う役割があります。エンコーダとデコーダの計算に必要なクロック数が異なるため、エンコーダではデータを保持したり、保持しながら計算を行ったりする状態もあります。この回路によって、1セット目のデータをデコーダで計算しながら2セット目のデータをエンコーダで計算することができるため、約2倍の計算時間の高速化を行うことができます。
![状態遷移図](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/b92b7ffb-ad71-4327-abf6-255ee3f295aa)

#### エンコーダ回路
この回路は標高データの次元数32×32を隠れ層の次元数32まで圧縮する役割があります。学習により取得した重みと入力データを乗算、それらを隠れ層の次元ごとに加算、バイアスを加算し、最後に活性化関数であるReLU関数を適用します。
![エンコーダ回路](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/25a79e6d-ab9b-4bac-9086-64379c7d22fb)

#### デコーダ回路
この回路は隠れ層の次元数32から、危険地帯データの次元数32×32に再構成する役割があります。計算の流れはエンコーダ回路と同様ですが、各処理ごとに処理の回数が異なるため、並列化やパイプラインの処理も異なっています。
![デコーダ回路](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/bdff6152-7ad2-4a95-b7b2-224979c66e0c)

### パイプライン・並列処理について
今回の設計では、主に3か所でのパイプライン化と1か所での並列化を行っています。

下の図において(a)はエンコーダにおける乗算回路のパイプライン化の様子です。miniMultiplierは1つの乗算を行う回路であり、それを32並列でおこなっています。隠れ層を1次元分構成するためには32並列の乗算を32回行う必要があるため、その部分をパイプライン化しています。これにより従来96クロックかかる処理が34クロックまで、約2.8倍削減されています。

(b)はエンコーダとデコーダのパイプライン化の様子です。大まかな処理の流れは「Multiplier(重み乗算)」⇒「Adder(加算)」⇒「Bias Adder(バイアス加算)」⇒「ReLU(ReLu関数適用)」となっており、これらの部分をパイプライン化しています。これによりエンコーダの場合は3552クロックから1043クロックに約3.4倍、デコーダの場合は10240クロックから1050クロックに約9.8倍削減されています。

(c)はオートエンコーダのパイプライン化の様子です。エンコーダの処理は1043クロック、デコーダの処理は1050クロックと、必要クロック数が異なるため、ステートマシンによりタイミング制御を行うことでパイプライン化をしています。これにより複数セットの標高データを処理する場合には約2倍の処理時間が削減されます。

![パイプライン](https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/b2821d4c-5de2-4a83-aa11-32b2a2b409b8)


## 5 実機による動作検証
### FPGA回路規模・処理速度について

| Resources | Used resources | Utilization[%] |
|-----------|-----------|-----------|
| AXI-4 | 〇 | 〇 | × |
| AXI-4 Stream | 〇 | × | 〇 |
| AXI-4 Lite  | × | 〇 | 〇 |

### 災害ハザードマップ予測回路システム

## 6 参考文献
https://japan.xilinx.com/products/boards-and-kits/zcu104.html




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


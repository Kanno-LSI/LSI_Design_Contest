# FPGA Development
FPGAによるシステム開発です。2023年10月から2024年2月まで取り組んだチーム開発の内容を説明しています。
<br><br><br><br>
## オートエンコーダを用いた災害ハザードマップ予測回路
回路設計をメインに書いていますので、完成したシステムだけ見たい場合は「1.概要」と、「5.実機による動作検証」をご覧ください。


## 目次
- [1.概要](#1-概要)
  - [目的](#〇-目的)
  - [システム内容](#〇-システム内容)
  - [チーム内の役割](#〇-チーム内の役割)
  - [ハザードマップについて](#〇-ハザードマップについて)
- [2.オートエンコーダ](#2-オートエンコーダ)
  - [オートエンコーダとは](#〇-オートエンコーダとは)
  - [オートエンコーダの学習](#〇-オートエンコーダの学習)
- [3.FPGA評価ボードの紹介](#3-FPGA評価ボードの紹介)
  - [FPGAとは](#〇-FPGAとは)
  - [使用するFPGA評価ボードについて](#〇-使用するFPGA評価ボードについて)
  - [PS部とPL部の役割](#〇-PS部とPL部の役割)
- [4.回路設計](#4-回路設計)
  - [全体図](#〇-全体図)
  - [PS部-PL部接続回路](#〇-PS部-PL部接続回路)
    - [AXI-4 通信回路](#AXI-4-通信回路)
    - [入出力回路](#入出力回路)
    - [PS部制御回路](#PS部制御回路)
  - [オートエンコーダ計算回路](#〇-オートエンコーダ計算回路)
    - [ステートマシン](#ステートマシン)
    - [エンコーダ回路](#エンコーダ回路)
    - [デコーダ回路](#デコーダ回路)
  - [パイプライン・並列処理について](#〇-パイプライン・並列処理について)
- [5.実機による動作検証](#5-実機による動作検証)
  - [FPGA回路規模・処理速度について](#〇-FPGA回路規模・処理速度について)
  - [災害ハザードマップ予測回路システム](#〇-災害ハザードマップ予測回路システム)
    - [土砂災害](#土砂災害)
    - [洪水](#洪水)
- [6.工夫点](#6-工夫点)
  - [入出力データに適した通信回路の設計](#〇-入出力データに適した通信回路の設計)
  - [オートエンコーダのパイプライン処理](#〇-オートエンコーダのパイプライン処理)
  - [3Dハザードマップ](#〇-3Dハザードマップ)
  - [ハザードマップの合成](#〇-ハザードマップの合成)
- [7.参考文献](#7-参考文献)
- [8.ソースコード](#8-ソースコード)


## 1 概要
今回開発したものは、FPGAのZynq ZCU104を用いた災害ハザードマップシステムです。

標高情報から、機械学習の１つである「オートエンコーダ」を介して危険地帯の予測を行い、地形データと合わせてハザードマップの3D表示を行います。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/a98ee72b-df89-44f0-9c5e-6640cc0617d4" width="500">
    <br> <!--改行-->
    <b>3Dハザードマップ</b> <!--テキスト表示-->
</div>

### 〇 目的

研究で使用し始めたFPGAの開発方法を学ぶことに加えて、特に以下の２つが目的になります。

①持てる技術の手札を増やし、今後大きなものを開発するための布石にしたい

②チーム開発の一連の流れを掴みたい

### 〇 システム内容および特徴
システム内容は主に以下の２つで構成されています。

PL部：オートエンコーダによる計算回路(今回のメイン)

PS部：ハザードマップの3D表示用アプリケーション

<br>
また、システムの特徴は以下の通りです。

①地形情報から、エンコーダ部において特徴データを抽出し、デコーダ部で所望の災害に特化したネットワークの回路設計

②ハードウェア内でパイプライン化・並列化を行うことによる回路リソースの効率化と高速処理の両立

③出力するハザードマップの3Dモデル化

### 〇 チーム内の役割
今回は3人チームでの開発であり、筆者はチームのリーダーです。開発における役割は以下の通りです。

<div align="center">
    <b>メンバーと役割</b> <!--テキスト表示-->
  
| メンバー | 役割 |
|-----------|-----------|
| メンバー1 | データセット作成 (Python) <br> オートエンコーダの学習部 (Python) |
| メンバー2  | オートエンコーダ推論部の回路 (VHDL) <br> 計算部分の回路シミュレーション (VHDL)|
| 筆者  | 通信回路、ステートマシン (VHDL) <br> 回路全体のコーディング <br> システムアプリケーション (Python) <br> 実機動作|
</div>

### 〇 ハザードマップについて
ハザードマップは自然災害が発生した場合の被害を予測して、被災想定地域や被害の範囲、避難場所や避難経路などを地図上に表示したものです。近年も自然災害による被害は頻発しており、2024年1月1日に発生した能登半島地震は記憶に新しく、甚大なものでした。そのような自然災害から人命を守るためには、予め危険地帯を予測したハザードマップによって万全の対策を取る必要があります。

<div align="center">
    <img src="https://github.com/Kanno-LSI/FPGA_Development/assets/131650927/c1a7710c-bc51-4e31-905c-6acd8a00920a" width="300">
    <br> <!--改行-->
    <b>ハザードマップの一例（千葉大学周辺）</b> <!--テキスト表示-->
    <br> <!--改行-->
 　 <b>出展：千葉市地震・風水害ハザードマップ</b> 
</div>

<br>
ハザードマップの活用は災害による被害を最小限にとどめるために重要ですが、その一方で地域の全てをカバーしているわけではありません。実際に、国や都道府県の洪水・浸水に関するハザードマップでは、指定区域に定められていない都道府県管理の中小河川が約1万9000ほども存在しています。そのようないわゆる「空白地域」を無くしていったり、また災害により変化していった地域のハザードマップを迅速に更新したりしていくためにも、ハザードマップを予測する回路の設計は重要なのではないかと考えました。

<br>

## 2 オートエンコーダ
### 〇 オートエンコーダとは
オートエンコーダ（自己符号化器）とは、ニューラルネットワークの１つです。入力層と出力層の次元数が同一であり、入力情報を一旦圧縮し、そこから同じ次元数まで復元を行うネットワークとなっています。オートエンコーダは、入力データを圧縮して特徴量を抽出する「エンコーダ」と、圧縮された特徴量データを元のユニット数まで復元する「デコーダ」に分けられます。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/574b6c1d-e1b0-461e-acb2-2a2b162bbe7a" width="400">
    <br> <!--改行-->
    <b>オートエンコーダのモデル図</b> <!--テキスト表示-->
</div>

<br>

オートエンコーダの応用例としてはコーデックやノイズ除去、画像補間などが挙げられますが、今回は**標高データから危険地帯を予測する**ために使用しています。標高データに対して、予測した危険地帯データを付加することでハザードマップを生成することができます。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/b91edddb-8eba-4a33-aca9-8ac1ebeb7010" width="400">
    <br> <!--改行-->
    <b>オートエンコーダの役割</b> <!--テキスト表示-->
</div>

### 〇 オートエンコーダの学習
オートエンコーダによる学習をPython上で行うことで、FPGAに実装する最適な重みやバイアスを取得します。重みとバイアス取得までの流れは、標高データ（訓練データ）の作成 ⇒ 教師データ（危険地帯データ）の作成 ⇒ オートエンコーダの学習 ⇒ 重みとバイアス取得　となります。

今回オートエンコーダで用いる標高データの仕様は以下の通りです。なお、学習に必要となる地形情報データは国土地理院が提供している「基盤地図情報ダウンロードサービス」よりダウンロードし、危険地帯データは国土地理院の「重ねるハザードマップ」を参照して作成しました。

<div align="center">
    <b>標高データの仕様</b> <!--テキスト表示-->
  
|  |  |
|-----------|-----------|
| エリア面積 | 2000[m] × 2000[m] |
| 地点数 | 32 × 32 |
| 標高値の正規化 | データセットの最大値を255として<br> 0 ~ 255 (8bit) に正規化 |
| データセット数  | 土砂災害用：2128セット <br> 洪水用：672セット <br> 津波用：952セット |
</div>

<br>

## 3 FPGA評価ボードの紹介
### 〇 FPGAとは
FPGA (Field-Programmable Gate Array)はハードウェア記述言語によって、ユーザーが自由に論理機能を書き換えられるハードウェアです。ASICなどと比べて、ハードウェアの開発期間を大幅に短縮することができます。

### 〇 使用するFPGA評価ボードについて
AMD-Xilinx社製FPGAのZynqを使用しました。本評価ボードはFPGAに加えてSoCを搭載しています。そのため、計算負荷の大きい部分をFPGAによるPL(Programmable Logic)部に、アプリケーションの実装をSoCによるPS(Processing System)部に行うことで、評価ボード1台でシステムを完結させることができます。本評価ボードの詳細は以下の通りです。

<div align="center">
<table><tr>
<td>

<div align="center">
    <img src="https://github.com/Kanno-LSI/FPGA_Development/assets/131650927/ceff9c37-3b67-4db0-9659-75f31c6164e8" width="300">
    <br> <!--改行-->
    <b>Zynq Ultra Scale+ MPSoC ZCU104</b> <!--テキスト表示-->
</div>

</td>
<td>

<div align="center">
    <b>評価ボードの仕様</b> <!--テキスト表示-->

|  |  |
|-----------|-----------|
| FPGA | Zynq Ultra Scale+ <br> MPSoC ZCU104 |
| Logic Cell | 504,000 |
| DSP Slice  | 1,728 |
| Bram  | 11[Mb] |
| 開発環境  | Vivado 2023.1 |
</div>

</td>
</tr></table>
</div>


### 〇 PS部とPL部の役割
PS部とPL部は以下のような役割になっています。PL部ではオートエンコーダの推論計算、PS部ではハザードマップの3D表示を行っています。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/14da589a-a4f3-4b9f-8706-3033b8fcbc57" width="500">
    <br> <!--改行-->
    <b>PS部とPL部の役割</b> <!--テキスト表示-->
</div>

<br>

PS部の仕様は以下の通りです。なお、PS部のOSで使用しているPYNQ(Python Productivity for Zynq)は、AMD-Xilinx社が提供するオープンソースプロジェクトであり、Pythonを用いたアプリケーション開発やPL部とのデータ通信を行えます。

<div align="center">
    <b>PS部の仕様</b> <!--テキスト表示-->

|  |  |
|-----------|-----------|
| CPU | Cortex-A53 <br> 1.5GHz 670 CPU |
| メモリ | 2.0[Gb] |
| OS  | PYNQ　Linux <br> based on Ubuntu 22.04 |
| コンパイラ  | g++ 11.2.0 |
| 使用言語  | Python |
</div>

<br>

## 4 回路設計
### 〇 全体図
以下に設計回路の概要を示します。この回路は、「PS部-PL部接続回路」、「オートエンコーダ計算回路」に大きく分けられます。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/aa806be9-347a-4975-8128-1ca3aaa99f61" width="500">
    <br> <!--改行-->
    <b>設計回路の全体図</b> <!--テキスト表示-->
</div>

### 〇 PS部-PL部接続回路
この回路はPS部とPL部のデータのやり取りを行う回路です。PS部からPL部に標高データを入力し、計算後の危険地帯データを出力します。オートエンコーダの計算における入出力のビット幅は8192bitですが、PS部とPL部の間のデータのビット幅は64bitであるため、データ形式の調整を行っています。

#### AXI-4 通信回路
PS部とPL部の通信にはAXI-4(Advanced eXtensible Interface 4)という通信用の規格を利用しています。AXI規格は、ARM社が仕様を策定したバスプロトコルであり、開発者側は共通のインタフェースを用いることで容易に接続することができます。AXI-4は以下のような種類があります。

<div align="center">
    <b>AXI-4の種類</b> <!--テキスト表示-->

| 規格名 | バースト転送 | アドレス指定 | 制御のしやすさ |
|-----------|-----------|-----------|-----------|
| AXI-4 | 〇 | 〇 | × |
| AXI-4 Stream | 〇 | × | 〇 |
| AXI-4 Lite  | × | 〇 | 〇 |
</div>

データ転送の際には、大容量のデータの入出力を行う場合と、複数の制御信号の入出力を行う場合があります。そのため制御のしやすさを考慮すると、大容量データの扱いにはデータの連続転送（バースト転送）が行えるAXI-4 Stream、複数の制御信号の扱いにはアドレス指定で柔軟な制御が行えるAXI-4 Liteが適しています。今回、標高データと危険地帯データにはAXI-4 Stream、PS部との制御にはAXI-4 Liteを使用しました。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/71363d4b-4914-4d69-9a31-766eb0d2c88f" width="1000">
    <br> <!--改行-->
    <b>AXI-4 通信回路</b> <!--テキスト表示-->
</div>

#### 入出力回路
この回路は入力データと出力データのコントロール回路です。オートエンコーダでの処理は32×32地点×8bit = 8192bitの入出力データを処理するため、AXI-4通信回路に合わせて64bitのデータを結合、分離する役割をしています。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/50f7e387-8b16-4324-9555-5fd8c8d9f6b2" width="500">
    <br> <!--改行-->
    <b>入出力データのコントロール回路</b> <!--テキスト表示-->
</div>

#### PS部制御回路
この回路はPS Operation Statusという名称で、PS部からオートエンコーダ回路のモニタリングや制御を行う役割があります。主にモニタリングしている情報は、計算時間と計算終了セット数であり、制御情報としてPS部から、"IDLE"、"WRITE"、"READ"といった入出力の状態を転送します。

### 〇 オートエンコーダ計算回路
この回路はオートエンコーダの推論回路です。「ステートマシン」、「エンコーダ回路」、「デコーダ回路」で構成されており、ステートマシンの制御によってエンコーダとデコーダでのパイプラインを可能にしています。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/f6a30017-f32e-4f31-8280-4a04f736c672" width="500">
    <br> <!--改行-->
    <b>オートエンコーダ回路</b> <!--テキスト表示-->
</div>

#### ステートマシン
この回路ではエンコーダとデコーダの状態遷移を管理して、適切なタイミングでデータの入出力を行う役割があります。エンコーダとデコーダの計算に必要なクロック数が異なるため、エンコーダではデータを保持したり、保持しながら計算を行ったりする状態もあります。この回路によって、1セット目のデータをデコーダで計算しながら2セット目のデータをエンコーダで計算することができるため、約2倍の計算時間の高速化を行うことができます。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/b92b7ffb-ad71-4327-abf6-255ee3f295aa" width="500">
    <br> <!--改行-->
    <b>エンコーダとデコーダの状態遷移図</b> <!--テキスト表示-->
</div>

#### エンコーダ回路
この回路は標高データの次元数32×32を隠れ層の次元数32まで圧縮する役割があります。学習により取得した重みと入力データを乗算、それらを隠れ層の次元ごとに加算、バイアスを加算し、最後に活性化関数であるReLU関数を適用します。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/25a79e6d-ab9b-4bac-9086-64379c7d22fb" width="1000">
    <br> <!--改行-->
    <b>エンコーダ回路</b> <!--テキスト表示-->
</div>

#### デコーダ回路
この回路は隠れ層の次元数32から、危険地帯データの次元数32×32に再構成する役割があります。計算の流れはエンコーダ回路と同様ですが、各処理ごとに処理の回数が異なるため、並列化やパイプラインの処理も異なっています。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/bdff6152-7ad2-4a95-b7b2-224979c66e0c" width="1000">
    <br> <!--改行-->
    <b>デコーダ回路</b> <!--テキスト表示-->
</div>

### 〇 パイプライン・並列処理について
今回の設計では、主に3か所でのパイプライン化と1か所での並列化を行っています。

下の図において(a)はエンコーダにおける乗算回路のパイプライン化の様子です。miniMultiplierは1つの乗算を行う回路であり、それを32並列でおこなっています。隠れ層を1次元分構成するためには32並列の乗算を32回行う必要があるため、その部分をパイプライン化しています。これにより従来96クロックかかる処理が34クロックまで、約2.8倍削減されています。

(b)はエンコーダとデコーダのパイプライン化の様子です。大まかな処理の流れは「Multiplier(重み乗算)」⇒「Adder(加算)」⇒「Bias Adder(バイアス加算)」⇒「ReLU(ReLu関数適用)」となっており、これらの部分をパイプライン化しています。これによりエンコーダの場合は3552クロックから1043クロックに約3.4倍、デコーダの場合は10240クロックから1050クロックに約9.8倍削減されています。

(c)はオートエンコーダのパイプライン化の様子です。エンコーダの処理は1043クロック、デコーダの処理は1050クロックと、必要クロック数が異なるため、ステートマシンによりタイミング制御を行うことでパイプライン化をしています。これにより複数セットの標高データを処理する場合には約2倍の処理時間が削減されます。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/b2821d4c-5de2-4a83-aa11-32b2a2b409b8" width="400">
    <br> <!--改行-->
    <b>パイプライン化の概略図</b> <!--テキスト表示-->
</div>

## 5 実機による動作検証
### 〇 FPGA回路規模・処理速度について
設計した回路の回路規模は以下の通りです。なお、最大動作周波数は161.06MHzであり、今回は動作周波数を100MHzに設定しています。DSPは加算や乗算を行えるユニットですが、今回は乗算のみに適用しており、加算はLUTにより行われています。IPコアを利用すると、LUTとDSPを意図的に選択することができるため、さらに回路規模の最適化が行えます。

<div align="center">
    <b>設計回路の回路規模</b> <!--テキスト表示-->

| Resources | Used resources | Utilization[%] |
|-----------|-----------|-----------|
| LUT | 86,021 | 37.34 |
| LUTRAM | 789 | 0.78 |
| FF | 150,843 | 32.74 |
| BRAM | 63 | 20.19 |
| DSP | 64 | 3.70 |
| BUFG | 24 | 4.41 |
</div>

続いて、PCで計算した結果と今回作成したシステムでの計算処理速度の比較結果が以下の通りです。今回は標高データ1セットと64セットの2通りで測定しており、PCでの計算にはPythonを使用しています。結果を比較すると、1セットの場合は約20倍、64セットの場合は約614倍高速化しています。特にFPGAシステムでは、PL部でのオートエンコーダの計算時間に比べて、PS部でのPythonの入出力の時間が非常に大きいため、64セットの場合も1セットと比較して2倍程度の時間に抑えられています。

<div align="center">
    <b>計算処理速度</b> <!--テキスト表示-->

| 環境 | 1セット | 64セット |
|-----------|-----------|-----------|
| PC | 0.0430[s] | 2.5782[s] |
| FPGAシステム | 0.0022[s] | 0.0042[s] |
</div>

### 〇 災害ハザードマップ予測回路システム
最後に災害ハザードマップ予測回路システムでの出力結果を説明します。今回は「土砂災害」、「洪水」、「津波」の学習を行い、それぞれ取得した重みとバイアスから結果を取得しています。

システムの動作の流れは、次の通りです。

①PS部からPL部の回路を書き換え、実行

②PS部で標高データを64bit形式に変換し、PL部にデータを送信

③PS部とPL部で制御信号のやり取りを行いながら、PL部でオートエンコーダの計算を実行

④計算終了後に危険地帯データをPS部に送信

⑤PS部で危険地帯のデータ形式を64bitの形式からソフトウェアで利用できるように変換し、標高データと併せてハザードマップの3Dモデルを表示

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/14da589a-a4f3-4b9f-8706-3033b8fcbc57" width="500">
    <br> <!--改行-->
    <b>PS部とPL部の役割</b> <!--テキスト表示-->
</div>

#### 土砂災害
以下は土砂災害のハザードマップです。土地は緑色で表示されており、その中で危険地帯のみ赤色で表示されています。なお危険地帯は、0~255までの値のうち、少しでも危険と判断された1以上の値が反映されています。

実際に結果を見ると、標高の低い地帯が危険地帯に判断されていることが確認できます。この地域では２つの崖があり、このような地域から土砂崩れが発生する可能性があるため、大方所望の結果が出力されていることが確認できます。

<div align="center">
    <img src="https://github.com/Kanno-LSI/FPGA_Development/assets/131650927/03471c6d-66b0-4c99-b589-6ee54aa079ac" width="700">
    <br> <!--改行-->
    <b>土砂災害のハザードマップ</b> <!--テキスト表示-->
</div>

#### 洪水
続いて、洪水の結果です。この地域は図の左下の低地に氾濫平野が形成されており、洪水の危険性があります。ハザードマップにおいても、そのような低地が危険地帯として出力されています。しかし、標高が高い一部の地域でもハザードマップでは危険地帯として出力されています。これはオートエンコーダの計算精度が足りないことが原因だと考えられます。また、今回は標高データにのみ機械学習を適用した形であり、地盤の固さや水源の位置などもデータの入力要件や学習要件に加わると、より精度の良いハザードマップシステムが見込めます。残りの津波の結果については、「工夫点」の「ハザードマップの合成」をご覧ください。

<div align="center">
    <img src="https://github.com/Kanno-LSI/FPGA_Development/assets/131650927/c394591c-746e-4495-9091-8d346bfb45fd" width="700">
    <br> <!--改行-->
    <b>洪水のハザードマップ</b> <!--テキスト表示-->
</div>


## 6 工夫点
### 〇 入出力データに適した通信回路の設計
データ容量の大きい標高データと危険地帯データにはAXI-4 Streamによる連続転送（バースト転送）を行う回路、複数種類の制御信号にはAXI-4 Liteによるアドレス指定を行える回路を個別で設計しました。これにより、ハードウェア側とソフトウェア側の間で高速なデータ転送と適切な信号制御の両方を行うことができました。

### 〇 オートエンコーダのパイプライン処理
オートエンコーダに対して3重のパイプライン処理を施すことで、回路を効率良くかつ高速に処理することができました。ハードウェアの設計はソフトウェアプログラミングと比べて、ロジック単位で記述できるため、並列化に加えてパイプライン化まで施すことができます。例えば下の図のように、1回の動作でA、B、C、D、Eの処理を順番に行うとします。パイプライン化をしない場合は、1つ目の動作がすべて終わった後に2つ目の動作を行う必要があります。パイプライン化を施すことで、1つ目の動作の途中に2つ目、3つ目、、の処理を同時に行うことができます。

<div align="center">
    <img src="https://github.com/Kanno-LSI/FPGA_Development/assets/131650927/3e7a728c-63eb-49f7-a629-d18f0c7e83e7" width="300">
    <br> <!--改行-->
    <b>パイプライン処理</b> <!--テキスト表示-->
</div>

### 〇 3Dハザードマップ
ハードウェアだけでなくソフトウェアのアプリケーションにもこだわり、3Dのハザードマップによって様々な視点から危険地帯を把握できるシステムを構築しました。一般的な2Dのハザードマップは、標高の表現を等高線や色を用いて表していました。そこで、実際の地形が分かるように3Dで表示できることで危険地帯の位置をより把握しやすくなります。緊急時にリアルタイムで予測を行い、その場で3D表示を行えば土地勘が分からない人でも避難を行いやすくなります。

### 〇 ハザードマップの合成
今回のシステムでは32×32地点の区域の計算を行っていますが、実際のハザードマップはより広い地域にする必要があります。そこで、複数区域を計算してからそれらを合成することで、さらなる広い区域のハザードマップを作成することができるようにしました。以下の図は津波の予測を行い、3×3区域の合成を行ったハザードマップです。なお、標高の低い地域は海になっているため、その近辺で危険地帯が出力されており、3×3の区域においても適当な出力が得られていると言えます。

<div align="center">
    <img src="https://github.com/Kanno-LSI/LSI_Design_Contest/assets/131650927/a98ee72b-df89-44f0-9c5e-6640cc0617d4" width="500">
    <br> <!--改行-->
    <b>津波のハザードマップ</b> <!--テキスト表示-->
</div>

## 7 参考文献
[1] 千葉市役所：千葉市地震・風水害ハザードマップ, 
(https://www.city.chiba.jp/other/jf_hazardmap/map.html?lay=saigai_12)

[2] 日本経済新聞：
　「ハザードマップに記されない空白地帯　浸水履歴の確認を」（2021年6月7日掲載）, 
(https://www.nikkei.com/article/DGXZQO
MH01AK60R00C21A6000000/)

[3] 国土地理院：基盤地図情報ダウンロードサービス, 
   (https://fgd.gsi.go.jp/download/menu.php)
   
[4] 国土交通省：重ねるハザードマップ, 
   (https://disaportal.gsi.go.jp/)

[5]AMD-Xilinx: Zynq UltraScale+ MPSoC ZCU104 評価キット, 
(https://japan.xilinx.com/products/boards-and-kits/zcu104.html)

[6] PYNQ: PYTHON PRODUCTIVITY, 
(http://www.pynq.io/)

[7] Vivado Design Suite: AXI リファレンス ガイド(UG1037), 
(https://docs.xilinx.com/v/u/ja-JP/ug1037-vivado-axi-reference-guide)

## 8 ソースコード
ソースコードを /src に載せています。
フォルダ構造は以下の通りです。

<div align="center">
    <b>フォルダ構造</b> <!--テキスト表示-->

| フォルダ位置 | 内容 |
|-----------|-----------|
| /src/HDL | HDLフォルダ |
| /src/HDL/axi_stream_and_lite | 通信回路 |
| /src/HDL/autoencoder | オートエンコーダ回路 |
| /src/HDL/sim_1 | 回路シミュレーション |
|  |  |
| /src/python | Pythonフォルダ |
| /src/python/学習用 | オートエンコーダ学習 |
| /src/python/pynq動作用 | PYNQ |

</div>



# NexmonによるCSIを用いた通過検知システム
## 0. はじめに
### 0.1 Wi-Fi(CSI)センシングの動機
　IEEE 802.11規格の無線周波数Wi-Fiは, 携帯電話やラップトップPC, ゲーム機, テレビ等の様々なデバイスの通信に使用される無線LANとして普及している. 近年のWi-Fiは, デバイス通信ではなく, 人間活動認識(HAR)等の新しいセンシング手法として注目を集めている.   
　HARの既存研究として, RGB-Dカメラや加速度センサー等のウェアラブルデバイスを用いた手法が盛んに行われているが, これらは接触型センシング手法であり, 対象者によっては注察感を与えてしまい, プライバシーに関する問題に直面する可能性を否定できない.   
　Wi-Fiを用いたセンシング手法は, 従来の接触型のセンシング手法の代替となる非接触型のセンシング手法となる可能性がある. また, 今後更なる情報化に伴ってスマートデバイスが遍在化することも想定できるため, 実用化の容易性の点でも優れている.
 Wi-Fiでは, CSI(Chanel State Information)という無線送信信号の振幅と位相に関する情報で構成されたデータが送受信機間でやり取りされている. CSIデータを分析することで, Wi-Fi通信環境下における様々な要因による時間的環境変化を捉えることが可能になる.   
### 0.2 Nexmonへの注目
　CSIデータは多くのWi-Fiチップメーカーは外部からのアクセス不可能にしていることが多く, CSIデータへのアクセスは容易ではない. 加えて, CSIデータへのアクセスを可能にするためのハードウェアとソフトウェアは共に高価であり, CSIデータを収集するにあたる制限が強い.そこで[[1]](https://github.com/haradakaito/NexmonCSI/blob/main/README.md#7-%E5%8F%82%E8%80%83%E6%96%87%E7%8C%AE)の論文中に示されるような, Nexmonと呼ばれるオープンソースのCSI収集用ファームウェアパッチを用いて, 比較的安価なデバイスでCSI収集を行う. Nexmonを用いることで従来のCSIが抱えるハードウェアとソフトウェア制限の強さを解消したCSI収集システムを構築することが可能である.   
### 0.3 本研究のモチベーション
　Wi-Fi(CSI)センシングに関するサーベイ論文として[[2]](https://github.com/haradakaito/NexmonCSI/blob/main/README.md#7-%E5%8F%82%E8%80%83%E6%96%87%E7%8C%AE)がある. [2]Fig.1には, Wi-Fiセンシングの概要について示されており, Wi-Fiセンシングは3つのセクションで大別され, 各セクションが更に3種類のアプローチで構成されている. 以下に, 各セクションとアプローチを示す. 

| Section  | Approach |
| ------------- | ------------- |
| Signal Processing(信号処理) | Noise Reduction(ノイズ低減), Signal Transform(信号変換), Signal Extraction(信号抽出) |
| Algorithm(アルゴリズム)  | Modeling-Based(数式モデルベース), Learning-Based(学習ベース), Hybrid(ハイブリッド) |
| Application(アプリケーション)  | Detection(検出), Recognition(認識), Estimation(推定) |

　本研究では, Nexmonを用いて(デバイスに対して)汎用性の高いCSI収集システムを構築し, 異常検知タスクにおける有効性を網羅的に検証することを目的とする.  

## 1. 既存研究
　本研究のセクション属性は以下のようになる.  
 
- Signal Processing(信号処理)：Noise Reduction(ノイズ低減) / Signal Transform(信号変換) / Signal Extraction(信号抽出)
- Algorithm(アルゴリズム)：Learning-Besed(学習ベース)
- Application(アプリケーション)：Detection(検出)

　Signal Procesingセクションに関しては, 単一のアプローチとは限らないため, Algorithm:Learning-Based(学習ベース), Application:Detection(検出)タスクを設定している研究の動向を調査した.   
### 1.1 

  
## 2. 実験環境
### 2.1 実験デバイス  
- CSI収集デバイス：RaspberryPi 4B  
- Wi-Fiルーター：WSR-1800AX4P/N  
- Ping送信デバイス：dynabook GX83/MLE
  
### 2.2 通信条件  
- 無線規格：IEEE 802.11n / ac
- 帯域：2.4 / 5 [GHz]
- 帯域幅：20 / 40 / 80 [MHz]
- チャネル：任意

## 3. データ収集フェーズ
### 3.1 収集条件
- Ping送信間隔：10 [Ping/s]
- Ping送信方法：Ping送信プログラム(.py)
- 収集時間：18000 [Ping]
- サンプリングレート：1~10 [Sample/s]
    
### 3.2 収集環境
  
## 4. 信号処理フェーズ
### 4.1 pcap→csv変換
- 変換方法：CSI Extractor(csi_changer.py)
```  
$ cd CSI_changer
$ python csi_changer.py
```
- 入力例 ( "帯域幅:20[MHz]" で収集された "App.pcap" の "0~100パケット" をcsvファイルに変換 )
```
Pcap File Name: App
Band Width: 20
> 0-100
``` 
### 4.2 信号処理アルゴリズム
- NR(Noise Reduction)  
　・Signal Nulling 2  
　・Phase Offset 2  
　　・PDD  
　　・STO  
　・Hampel Filter 6  
　・Linear Fitting 2  
　・Least Median Squares  
　・WMA 2  
　・Density-Based Spatial Clustering  
　・Wavelet Filter 3  
　・LOF 2  
　・Median Filter  
　・Phase Difference 2  
　　・Interference Nulling  
　・Long Delay Removal  
　・LPF  
　・PCA  
　・MA 2  
　・Temporal Bias  
　・De-Correlation Filter  
　・Frequency/Temporal Smoothing  
　・Random Phase Perturbation  
　・Butterworth LPF  
　・CFO  
- ST(Signal Transform)  
　・DWT 4  
　・STFT 3  
　・FFT  
　・IFFT 2  
　・DWT  
　・DHT  
　・Linear Transform  
- SE(Signal Extraction)  
　・Interpolation 8  
　・kernel PCA  
　・Correlation Matrix  
　・PCA 8  
　・Feature Extraction 3  
　・Subcarrier Selection 4  
　　・Periodicity&SVD 1  
　・Thresholding 10  
　・BPF 2  
　・LPF 3  
　・Threshold-Based Sliding Window  
　・Trend Filter  
　・Euclidean Distance 2  
　・Signal Isolation by Skewness  
　・Duration-Based Filter  
　・Signal Compresson by CSI Amplitude Variance  
　・Butterworth BPF 2  
　・Multiple TX-RX Pairs  
　・Mahalanobis Distance  
　・EMD  
　・k-means  
　・Merging Adjacent Samples  
　・Antenna Selection 3  
　・Outlier Filter  
　・Eigen Values  
　・Normalization  
### 4.3 サブキャリア選択
参考1：https://zenn.dev/shungo_a/articles/ffbdb3614867ca  
参考2：https://blog.since2020.jp/data_analysis/time_series_kmeans/  
前処理として, 「標準化」か「正規化」を行う必要がある.  
- 時系列クラスタリング  
　・k-means  
  ・ユークリッド距離  
  ・DTW  
  ・k-shape  
  ・PCA  
  ・t-SNE  
- SSE(クラスター内誤差平方和)による評価 

### 4.4 データ分割
- ダウンサンプリング
- シーケンス単位でシャッフル
- Train：Val：Test = 6 : 2 : 2

### 4.5 ウィンドウサイズ
ウィンドウサイズは, 1~60[s]スケールになるように設定
- ウィンドウサイズ(SL:1)：1 / 5 / 10 / 20 / 30 / 40 / 50 / 60
- ウィンドウサイズ(SL:5)：2 / 10 / 20 / 40 / 60 / 80 / 100 / 120
- ウィンドウサイズ(SL:10)：10 / 50 / 100 / 200 / 300 / 400 / 500 / 600
  
## 5. 学習・評価フェーズ
### 5.1 学習器
- SVM
- RandomForest
- LSTM
- LSTM-FCN
- 1D-CNN
- ResNet
- Transformer

## 6. 今後の展望
- 個人差の検討
- 環境差の検討
- 損失関数
- ソフトラベル
- 異常検知用サブキャリア選択モデル

## 7. 参考文献
**[[1]](https://dl.acm.org/doi/10.1145/3349623.3355477)** GRINGOLI, Francesco, et al. Free your CSI: A channel state information extraction platform for modern Wi-Fi chipsets. In: Proceedings of the 13th International Workshop on Wireless Network Testbeds, Experimental Evaluation & Characterization. 2019. p. 21-28.  
**[[2]](https://dl.acm.org/doi/abs/10.1145/3310194)** MA, Yongsen; ZHOU, Gang; WANG, Shuangquan. WiFi sensing with channel state information: A survey. ACM Computing Surveys (CSUR), 2019, 52.3: 1-36.  

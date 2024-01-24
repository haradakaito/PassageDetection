# NexmonによるCSIベースの人物通過検出システムに関する研究
## 0. はじめに 
監視社会としての側面が強まる現代で，個人のプライバシーを尊重したセンシング技術の発展が喫緊の課題である．近年，Wi-Fi電波の通信媒体波及時における，チャネル状態情報(CSI)から振幅・位相情報を分析することで，非接触型人間活動認識(HAR)を行う技術が注目を集めている．  
これはカメラベースの手法に比べ，明暗の変化に強く，かつプライバシー侵害が少ない特長がある．また，赤外線ベースの手法よりも温度変化に強く，対象者の移動方向に対する脆弱性も低減している．さらには，センサの装着が不要であるため，対象者のストレスを最小限に抑えられる．今後，多様なデバイスやプラットフォームが普及していく中で，Wi-Fi電波(CSI)の利用は実用面において優れている．  
しかし，Wi-Fiチップの大半は，CSIへのアクセスが制限されており，アクセスが可能なハードウェアとソフトウェアは高価であるため，完全なCSIの取得が困難である．  
先行研究[1,2]では，オープンソースのCSI収集用ファームウェアパッチであるNexmon[3]を用いて，CSI制限に依存せず，導入コストを抑えたセンシングの可能性を示したが，実現手法やシステムアーキテクチャの性能の検証・評価が十分に実施されているとは言えない．本研究では，NexmonによるCSIベースの人物通過検出システムの実現手法を示し，その性能の検証・評価によって，NexmonによるCSIベースの人物通過検出システムにおける可能性を示す．  
![図_CSIイメージ](https://github.com/haradakaito/PassageDetection/assets/75819611/c9ebc7a3-a9cb-43da-a217-99322f136b2d)

## 1. 人物通過検出手法
NexmonによるCSIベースの人物通過検出システムの実現手法を提案する．提案手法は「データ収集」「信号処理」「学習・評価」の3フェーズから構成される．
「データ収集」フェーズでは，Nexmonを対応Wi-Fiチップを備えたデバイスに適用し，IEEE 802.11n規格以降の無線通信におけるサブキャリアの振幅情報を取得する．  
「信号処理」フェーズでは，収集した生のCSIデータに対して前処理を行う．具体的には，未使用サブキャリアを閾値によって除去する．残ったn個のサブキャリアに対して，スライディングウィンドウを作成し，hampelフィルターを適用することでノイズ除去を行う．さらに，differenceフィルターを適用し，トレンド非定常性や，スケールの差異を解消する．  
最後に，形状ベースの時系列クラスタリング手法であるk-shapeを用いて，全サブキャリアクラスタ重心を抽出し，代表値として使用することで，計算コスト・過学習を低減する．  
具体的には，ラベル平滑化とk-Foldアンサンブル学習により，k個の分類器を構築する．また，全k個の予測値は相加平均で統合され，補正処理により分類器の検出性能を向上させる．  
![図_学習アーキテクチャ](https://github.com/haradakaito/PassageDetection/assets/75819611/d8fb586b-fbc3-4255-ada5-68a214ee7de5)

## 2. 基礎評価
### 2.1. データ収集フェーズ
CSI収集用デバイスとしては，安価で入手容易なRaspberryPi4Bを使用した．また，Wi-Fi送受信機としては，IEEE 802.11n/ac規格対応のWi-Fiルーター(WSR-1800AX4P-WH)とラップトップPC(dynabook GX83/MLE)を使用した．通信条件は，送受信機間距離を1.5[m]の距離で屋内に設置し，2.4[GHz]帯の帯域幅20[MHz]で通信間隔を20[Ping/s]として通信を行った．
収集条件は，単体の通過を想定し，教師信号は「通過と非通過」の2値とし，通過速度はSlow:約0.5[m/s]，Normal:約1.0[m/s]，Fast:約2.0[m/s]の3パターンで収集を行った．  
![図_収集環境](https://github.com/haradakaito/PassageDetection/assets/75819611/b69ad1bc-46a9-4260-b83e-b59eaf6d422a)

### 2.2. 信号処理フェーズ
収集したサブキャリア数は64個であり，IEEE 802.11n規格の20[MHz]帯では，内56個が通信に使用されるため，8個の未使用サブキャリアの除去を行った．また，hampelフィルター，およびdifferenceフィルターを適用し，ノイズ除去，トレンド非定常性，スケールの差異を解消した．信号処理フェーズは，非通過区間の分散を抑え，通過区間をより明確にする効果がある．そのため，各処理を適用した際の分散(標準偏差)の変化から，信号処理フェーズの効果を定量的に評価する．  
図3に，収集された全3パターンのCSIデータに対して，信号処理フェーズのパイプラインを適用した際の，標準偏差の変化を示す．各サンプルで標準偏差は減少傾向を示し，信号処理フェーズの有効性を確認した．  
![折れ線グラフ_信号処理パイプライン](https://github.com/haradakaito/PassageDetection/assets/75819611/f0a36bce-82b9-4c33-8544-c752239b2207)

### 2.3. 学習・評価フェーズ
データの前処理後，第2章で示したアーキテクチャ(図1)に基づいて，目的変数をシーケンス内のラベル割合としてソフトな(One-Hotでない)状態で使用した．また，k-Fold(k=3)とし，アンサンブル学習モデルを構築する際の分類器は，複数の時系列モデル(1D-CNN，LSTM，LSTM- FCN，ResNet，Transformer，RandomForest)で試行した．今回は，統合された予測値の補正方法として，誤分類箇所がスパイク状に出現することから，hampelフィルターを使用した．  
各学習モデルへの入力時点数を10～30，hampelフィルターのパラメータをt=10~50，α=1.5として試行した際の，F値の分布を示す．分類器:LSTM-FCNで，Accuracy:0.985，F1-score:0.978を，最良の結果として確認した．  
![箱ひげ図_学習モデル比較](https://github.com/haradakaito/PassageDetection/assets/75819611/4e14fc27-e78a-44d8-822c-3dccc273f3ae)

![画像1](https://github.com/haradakaito/PassageDetection/assets/75819611/0cc96818-fd5b-463d-8b53-60408827326f)

## 3. 人物通過検出手法の応用
人物通過検出システムを応用し，人物通過速度分類を行う．これは，通過速度と検出区間幅に、有意な相関が存在するという仮説に基づいている．モデルの予測値から検出区間幅を得る．基礎評価で使用したテストデータ内では，合計23回(Slow:7回，Normal:7回，Fast:9回)の通過区間が存在している．得られた検出区間幅データに対して，k-means法でクラスタリングを行った際の，分類結果を示す．全通過区間で，適切な通過速度分類が可能なことを確認した．  

## 4. 考察
サンプ![画像5](https://github.com/haradakaito/PassageDetection/assets/75819611/ec7fc8cb-8a04-4ad2-87e9-f75575175a56)


## 5. おわりに
NexmonによるCSIベースの人物通過検出システムの実現手法を示し，その性能の検証・評価によって，NexmonによるCSIベースの人物通過検出システムにおける可能性を示した．第3章では，第2章の提案手法を用いて，高精度に人物通過検出を行えることを示した．また，第4章では，検出システムを応用し，通過速度分類が可能なことを示した．
今後の展望としては，さまざまな内外的要因(通過人数や通信条件，環境差など)に対し，より広範な調査を行い，他ドメインへの実用的なシステムとしての運用を検討する．  

## 参考
[1]Shahverdi, H., Nabati, M., Moshiri, F.R., et al.: Enhancing CSI-Based Human Activity Recognition by Edge Detection Techniques, Information, Vol.14.7, No.404, (2023).  
[2]Xia, Z. and Chong, S.: WiFi-based indoor passive fall detection for medical Internet of Things. Computers and Electrical Engineering, Vol.109, No.108763, (2023).  
[3]Gringoli, F., Schulz, M., Link, J., et al.: Free your CSI: A channel state information extraction platform for modern Wi-Fi chipsets, Proceedings of the 13th International Workshop on Wireless Network Testbeds, Experimental Evaluation & Characterization, pp.21-28 (2019).  

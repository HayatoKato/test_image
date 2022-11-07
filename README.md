# Nishika 「ボケ判定AIを作ろう」

運営・参加者の皆さま、ありがとうございました。
以下、解法を共有させていただきます。


### 手法概要と結果
アーキテクチャの全体概要としては以下の図の通りです
![アーキテクチャ概要]("https://github.com/HayatoKato/test_image/main/img/architecture.png")

コンペ期間中は以下の三つのモデルを検討
1. LightGBM
2. MMBT
3. Transformer

各モデルへ入力した特徴量は以下のとおり
##### 1.LightGBM
- 文章の長さ
- 「BERT/chiTra」で抽出した自然言語ベクトル
- 「Sentence-BERT」で抽出した自然言語ベクトル
  - 学習済みモデルは[sonoisa/sentence-bert-base-ja-mean-tokens-v2](https://huggingface.co/sonoisa/sentence-bert-base-ja-mean-tokens-v2)を使用
- 「CLIP_en/VIT-L14」で抽出した画像ベクトル
- tf-idfスコア
  - Henoheno Mohejiさんのディスカッション投稿を参考にtf-idf集計値(最大値、最小値、平均値、中央値、分散値)を入力
  - tf-idf計算時の分かち書きにはNEologd辞書を使用
- 文章の文末単語ラベル(end_words)
  - is_laughが1と0の場合で出現頻度に差がある文末単語をもとにラベル化
  - 「'た', 'る', '！', 'い', '」', '。', '？', それ以外」の9値ラベル
##### 2.MMBT
- 「BERT/chiTra」で抽出した自然言語ベクトル
- 「CLIP_en/VIT-L14」で抽出した画像ベクトル
##### 3.Transformer
- 「BERT/chiTra」で抽出した自然言語ベクトル
- 「CLIP_en/VIT-L14」で抽出した画像ベクトル

最終提出時には個別のモデル認識精度が一番高かったLightGBMと次点のMMBTの2つの予測結果を平均した結果を提出
| |  PublicLB  |  Private LB |
| ---- | ---- | ---- |
|  LightGBM  |  62.15  | 62.71 |
|  MMBT  |  62.17  | 63.04 |
|  Transformer  |  63.50  | 63.81 |
|  LightGBM & MMBT  |  61.82  | 62.53 |
<br>

### CV
StratifiedKFold (5fold) を適用
CVによって学習された複数のモデル (5つのモデル) の予測結果を平均することで少し精度は向上
<br>

### Data Augmentation
今回のコンペではAugmentationは適用していません。
<br>

### 画像・言語の事前学習済みモデルの選択
画像・言語の事前学習済みモデルとしてCLIPを用いることを検討   
最初は rinna Co., Ltd の日本語版CLIPを検討    (https://huggingface.co/rinna/japanese-clip-vit-b-16)
<br>

だが以下の論文から日本語版CLIPを用いない方向に検討   
Sheng Shen et.al, "How Much Can CLIP Benefit Vision-and-Language Tasks?Download PDF", ICLR 2022.   
上記論文の節「Appendix A.6 ANALYSIS ON THE CLIP TEXT ENCODER」の実験結果によれば、   
言語エンコーダーはCLIPの言語エンコーダーより、BERTの方がV&LタスクとLタスクのみの両方で高精度という報告がされていた   
この結果を元に最終的に画像・言語エンコーダーは以下の事前学習済みモデルを使用   
画像学習済みモデル : CLIP/VIT-L14 (https://github.com/openai/CLIP)   
言語学習済みモデル : chiTra (https://github.com/WorksApplications/SudachiTra)
<br>

### 言語エンコーダーの選定
東北大BERT を最初は検討していたが、未知語が多く検出された。  
そこで、形態素解析器 Sudachi を利用した単語正規化の機能をもつ Hugging Face 互換のトークナイザー "chiTra" を利用した。
chiTra の特徴として、
- 大規模テキストコーパス (NWJC) による学習で多様な表現とさまざまなドメインに対応可能  
- 形態素解析器 Sudachi を利用することで表記ゆれによる弊害を抑えられる

といった点が挙げられる。  
chiTra を利用することで、サンプルプログラムで利用されていた 東北大BERT よりも若干の精度改善を確認できたため、今回採用した。

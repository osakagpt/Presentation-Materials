# RAGとは
Retrieval Argumented Generation
ふざけんな！

「検索でプロンプトを拡張したLLMによるテキスト生成」

# キーワード
- Retriever Readerモデル
- LlamaIndex, LangChain, HeyStack
- 疎ベクトルと密ベクトル
- DPR(Dense Passage Retrieval)
- Retriever Generatorモデル

# LLMの限界を一言で言うと、、
- 学習していないことはわからない

ChatGPTは大量のオープン情報を幅広く学習しているためものすごく物知りなのは間違いないが、クローズドな情報は摂取していないためそのことについて聞かれても答えられない

## 例
- 「1年前のミーティングで取引先の○○社の人は××に反対してたっけ？」
    - ソースは社内の膨大な議事録なのでChatGPTは知らない
- 「弊社のチャットサービスのユーザー××はどんな趣味を持ってる？」
    - ソースはユーザーのチャット履歴なのでChatGPTは知らない

## LLMの限界を克服するために、
- ファインチューニング
- プロンプトエンジニアリング
の2手法がある。前者は学習させてモデルのパラメータを変化させよう。後者はパラメータはそのままでLLMに入力するプロンプトを工夫しよう。RAGは後者の手法に関連した用語である。


# Retriever Readerモデル
![](../images/QA-retriever-reader.png)

## 補足
- External Knowledgeで想定しているのは、テキストファイル、PDF、パワーポイント、データベース、API経由でのjsonなど
- これらのKnowledgeがいったん検索しやすいIndexという形で保存される
- indexは複数のコンテキストの集まり
- テキストファイルの場合は、適切なサイズで区切られた文字列をひとつのコンテキストとする
- RetrieverはqueryとIndexを受け取って、関連するノードを導出する

- readerは関連するコンテキストの中から、クエリの答えを抽出する
- つまりretrieverは関連コンテキストの範囲を絞ることでreaderの負担を減らす役割であるといえる

- Retrieverの手法はDPR(Dense Passage Retrieval)ともよばれ、retriever自体を教師あり学習で訓練することができる

# Retriever Generatorモデル
![](../images/QA-retriever-generator.png)

## 補足
- Retriever ReaderモデルのReaderと違って、Generatorはクエリの答えをゼロから生成する
- ChatGPTのプロンプトにクエリと関連コンテクストを入力すると、答えが返ってくるのをイメージするとよい
- これがRAG(検索によって拡張された生成)

- RetrieverとGeneratorを一括で訓練することができる(しかしファインチューニングしたらお手軽さという利点が失せるのでは)

# Retrieverモジュールの具体例
- LlamaIndex
- LangChain
- HeyStack(昔からあるらしい)

結局やっていることは、
- Text Embedding つまり、テキストを高次元の密ベクトルに変換すること。
- 類似度計算


# LlamaIndexを使ってみよう
[](https://docs.llamaindex.ai/en/latest/#)
LLMに食わせるプロンプトを自分用にファインチューニングするかわりにプロンプトを工夫することでファインチューニングと同様の効果を達成することを狙っている。具体的には以下のツールの集まりを指す

## Data Connectors
APIやSQL、PDFなどからデータを取得する
## Data Indexes
LLMが消費しやすく、パフォーマンスが良い中間表現でデータを構造化する
## Engines
データへの自然な言語アクセスを提供する
 - query engineは知識拡張された出力のための強力な検索インターフェース
 - chat engineはデータとの対話的なやりとりでのマルチメッセージや「やりとり」のための対話インターフェース

## どんなユースケースにふさわしい？

# 参考文献・URL
- [How to Build an Open-Domain Question Answering System?](https://lilianweng.github.io/posts/2020-10-29-odqa/)
- [大規模言語モデルの知識を補完するための Retriever の紹介](https://tech.acesinc.co.jp/entry/2023/03/31/121001)
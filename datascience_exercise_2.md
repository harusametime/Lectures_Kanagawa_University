# 演習の目的

本コンテンツは神奈川大学 情報学部 実践的データサイエンス演習受講生向けに作られたページです。

本講義は AWS の学習環境 [AWS Academy](https://aws.amazon.com/jp/training/awsacademy/) を使ってクラウドの基本を演習を通して学ぶことを目的としています。情報学部の学生の多くが、ウェブサイトの作成と公開を初期段階で学ぶことから、それに沿った形で、AWS Academy の追加コンテンツとして独自に作成しています。 コンテンツに関する問い合わせは AWS Academy ではなく github の issue にて報告をお願いします。

## 演習の概要

Python を使った情報検索の基本的な方法を学習します。Amazon SageMaker AI の Notebooks という機能を利用して Jupyter Lab 上で演習を行います。


## AWS Academy のアカウント登録

学生が AWS Academy を利用するためにはアカウントを登録する必要があります。教員が受講生をコースに追加すると「コースへの招待」というメールが届くため、それに従って学生はアカウントを登録します。

<img width=400 src="./images/account_creation.png">



## AWS Academy へのログイン

アカウント登録が終わったら、以下のページから AWS Academy へログインします。Student Login を選んでください。
https://www.awsacademy.com/vforcesite/LMS_Login

<img width=400 border=1 src="./images/login_page.png"> 

## コースの選択とモジュールの選択

左のメニューからコースを選択し、AWS Academy Machine Learning for Natural Language Processing を選択します。

<kbd><img width="443" height="404" alt="image" src="https://github.com/user-attachments/assets/da893850-9d09-4a66-b5ed-62b45361eace" /></kbd>

  
次に  Module 2 - Introduction to Natural Language Processing (NLP) から 「Lab 2.1 - Applying ML to an NLP problem」を選択します。

<kbd><img width="450" height="1006" alt="image" src="https://github.com/user-attachments/assets/be73bd77-aa4f-4758-9374-d885ff522f32" /></kbd>


## AWS 環境の起動

初回は同意事項に関するページが表示されますので、一番下までスクロールして I Agree をクリックし、同意してください。

<kbd><img width=600 src="./images/agreement.png"></kbd>

するとサンドボックスの説明画面が開きます。右上のメニューから Start Lab ボタンをクリックすると、Labをスタートすることができます。ボタンを押すと環境構築中の別画面が開きますが、1-2分たったら閉じて問題ありません。AWS ボタンを押すとAWS コンソールの画面を開くことができます。
もしAWSボタンを押してもAWSコンソール画面が開かない場合は、まだ環境構築中の可能性があるので、もう少し待ってみます。

<kbd><img width=600 src="./images/start_lab.png"></kbd>

## 演習内容

### Amazon SageMaker AI　の起動

Amazon SageMaker AIは機械学習の開発・運用を効率化するサービスです。
AWS コンソールが起動したら左上の検索ウィンドウに SageMaker AI と入力して、Amazon SageMaker AI のサービスを検索します。

<kbd><img width="525" height="271" alt="image" src="https://github.com/user-attachments/assets/6eaf2cb7-86a9-44c4-bd0c-b0188b08d65e" /></kbd>

左のメニューから Notebooks を選択します。

<kbd><img width="582" height="990" alt="image" src="https://github.com/user-attachments/assets/5173717f-6da2-4e93-8df6-8d44b25b5a8d" /></kbd>

次に右の画面から Jupyter Lab をクリックします。

<kbd><img width="598" height="506" alt="image" src="https://github.com/user-attachments/assets/4b7ef17b-43bb-4a07-a2f6-6477a8951e1c" /></kbd>

Jupyter Labを開いたらconda_pytorch_p310を選んでノートブックを作成します。

<kbd><img width="600" height="742" alt="image" src="https://github.com/user-attachments/assets/40121107-5255-450d-babb-c8203772df41" /></kbd>

### Jupter Lab での演習

#### 準備
以下はノートブックを作成しながら演習を行います。
以下のようにコードをコピーペーストして、▶ボタンを押すか、Ctrl+Enter でコードを実行できます。

<kbd><img width="668" height="167" alt="image" src="https://github.com/user-attachments/assets/76571829-c203-4f07-a2a9-ab5a4a379a94" /></kbd>

まずは以下のコードを貼り付けて実行し、生成 AI モデルをダウンロードして利用できるようにします
[Hugging Face](https://huggingface.co/)では様々なオープンソースのモデルが公開されており、transformers ライブラリから簡単にアクセスすることができます。

```
!pip install transformers==4.51.0 accelerate
```

#### 生成 AI モデルのロード

次に生成 AI モデルのダウンロードを行います。今回の演習では、Alibaba 社の Qwen3-0.6B モデルを利用します。
Qwen3-0.6B は2025年4月にリリースされた最新の軽量大規模言語モデル（LLM）でQwen3シリーズの最小モデルです。
以下のコードを実行して、 Qwen3-0.6B をロードします。

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

# モデルの指定
model_name = "Qwen/Qwen3-0.6B"

# トークナイザーとモデルのロード
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype="auto",  
    device_map="auto"
)
```

#### RAG 用途に呼び出しの関数を定義

生成 AI モデルは、ユーザからの質問に自然に回答できないことが一般的で、ユーザからの質問に加えて、様々な指示を行う必要があります。
例えば
- 日本語で回答して
- ユーザからの質問に答えて
- 与えられら文章を要約して
  
などが挙げられますが、精度を向上するためにはより詳細な指示を与える必要があります。
普段、生成AIのサービスを利用する際はこの点を気にせず使えますが、これは内部的に詳細な指示を与えていたり、規模の大きい優れたモデルを複数使っていたりするからです。

今回の演習では、RAG を行うための指示を含んだ関数を以下のように定義、準備しています。
このコードブロックを実行して、以降、この関数を使ってモデルを利用できるようにします。

```python
def rag_qwen3(context: str, question: str, max_tokens: int = 256) -> str:
    """
    RAG用関数：context と question を受け取り、日本語で回答を生成
    """
    messages = [
        {
            "role": "system",
            "content": (
                "あなたは正確で親切な日本語アシスタントです。 "
                "与えられた情報（コンテキスト）だけを使って質問に回答してください。 "
                "コンテキストにない情報は「わかりません」と答えてください。 "
                "回答は必ず日本語で、簡潔に。"
            )
        },
        {
            "role": "user",
            "content": (
                f"### コンテキスト\n"
                f"{context}\n\n"
                f"### 質問\n"
                f"{question}\n\n"
                f"上記のコンテキストに基づいて、**日本語で正確に**答えてください。"
            )
        }
    ]

    # Qwen推奨：チャットテンプレート適用
    text = tokenizer.apply_chat_template(
        messages,
        tokenize=False,
        add_generation_prompt=True,
        enable_thinking=True  # 論理的推論を強化（RAGに最適）
    )

    inputs = tokenizer(text, return_tensors="pt").to(model.device)

    with torch.no_grad():
        outputs = model.generate(
            **inputs,
            max_new_tokens=max_tokens,
            do_sample=True,
            temperature=0.3,           # RAGは正確性重視 → 低め
            top_p=0.9,
            repetition_penalty=1.1,
            pad_token_id=tokenizer.eos_token_id
        )

    response = tokenizer.decode(outputs[0], skip_special_tokens=True)
    answer = response[len(text):].strip()
    
    return answer
```

表示するとわかりますが各行の text という列にテキストデータが入っています。ある行 i のテキストデータは `df.iloc[i]['text']`でアクセス可能です。
もしキーワードが入っているか調べたい場合は `"キーワード" in df.iloc[i]['text']`で調べます。入っていれば True となります。
では、

#### 単純なキーワード一致検索

まずはキーワードの一致で検索してみます。例えば、**京都**というキーワードで検索し、最初の100文字を表示するには以下のような実装ができます（もっと便利な実装はありますが今回は割愛します）
以下では最初の100ドキュメントだけ検索し、一致箇所の前後10文字を取り出します。
```python
offset = 10
for i in range(100):
    query = "京都"
    text = df.iloc[i]["text"]
    if query in text:
        print(f"================記事インデックス{i}================")
        char_index = text.index(query)
        print(text[char_index-offset:char_index+offset])

```

いろいろ結果をみてみると、**東京都**と誤って一致しているケースが見受けられます。

#### 形態素解析の導入

形態素解析をして、東京都と京都を見分けられるようにします。以下を実行して形態素解析をインストールして試してみます。

```python
!pip install mecab-python3==1.0.10 unidic-lite==1.0.8

# 形態素解析の設定
import MeCab
tagger = MeCab.Tagger("-Owakati")

# 試してみる
result=tagger.parse("私は京都に住んでいます")
print(result)
```

形態素の前後にスペースが挿入されていることを確認できます。この`tagger.parse`の処理を先程の検索に入れてみます。結果は変わりましたか？

```python
offset = 10
for i in range(100):
    query = "京都"
    text = df.iloc[i]["text"]
    text = tagger.parse(text) #形態素解析した結果で置き換える
    if query in text:
        print(f"================記事インデックス{i}================")
        char_index = text.index(query)
        print(text[char_index-offset:char_index+offset])
````

#### 類似度検索

講義で説明したように BM25 アルゴリズムで類似度検索ができます。まずは以下を実行してライブラリをインストールします。

```python
!pip install rank_bm25==0.2.2
```

次に以下を実行して、クエリと全文書との類似度を計算し、類似度トップ5の文書を表示します。
途中の query を変更することができます。クエリにあった内容が表示されることを確認しましょう。

```python
from rank_bm25 import BM25Okapi
import MeCab
import numpy as np

# MeCabで分かち書き
tagger = MeCab.Tagger("-Owakati")
tokenized_corpus = [tagger.parse(doc).strip().split() for doc in df["text"]]

# BM25 モデル作成
bm25 = BM25Okapi(tokenized_corpus)

# クエリを分かち書きしてスコア計算
query = "東京　音楽"
tokenized_query = tagger.parse(query).strip().split()
scores = bm25.get_scores(tokenized_query)


# 上位5件のインデックスを取得
top_n = 5
top_indices = np.argsort(scores)[::-1][:top_n]

# 結果表示
print("Top 5 documents:")
for idx in top_indices:
    print(f"======== Doc {idx}: Score = {scores[idx]:.4f} =======")
    print(f"Text = {df.iloc[idx].text[:100]}")
```

BM25を計算するためには、文書中のすべての単語を調べる必要があります。
上記コードの `bm25 = BM25Okapi(tokenized_corpus)` は全文書を渡して、単語の頻度などを調べています。

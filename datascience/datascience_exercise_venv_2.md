# 演習の目的

本コンテンツは神奈川大学 情報学部 実践的データサイエンス演習受講生向けに作られたページです。

## 演習の概要

Python を使った情報検索の基本的な方法を学習します。Mac OS の環境で Jupyter notebook を使って演習を行います。

## 環境のセットアップ
**前回講義でセットアップが済んでいる方は不要です。Jupyter Lab の演習のパートに飛んでください。**

環境構築には venv を利用します。venv はPythonの仮想環境で、プロジェクトごとに独立した Python 実行環境（今回はこの演習用の環境）を作成することができます。システム全体の Python に影響を与えず、プロジェクト専用の Python とライブラリを管理できます。

まずは Mac のターミナルを開いてください。cmd+space でterminal と打てば検索できると思います。

![alt text](../images/terminal.png)

まずは演習用のディレクトリを作成します。名前はなんでも構いませんが情報検索 (Information Retrievall; IR) の文字を取って以下のようにしてみます。

```shell
mkdir ir_project
cd ir_project
```

仮想環境を作ります。利用している環境によっては python は python3 となる場合があります。

```shell
python -m venv .venv
```

仮想環境を有効化します。
```shell
source .venv/bin/activate
```
pip を最新にして必要なライブラリをいれます。以下をコピペして実行しましょう。
```shell
pip install --upgrade pip
pip install \
    pandas \
    mecab-python3==1.0.10 \
    unidic-lite==1.0.8 \
    rank_bm25==0.2.2 \
    transformers==4.51.0 \
    accelerate \
    "huggingface-hub>=0.30.0,<1.0" \
    --force-reinstall \
    sentence-transformers \
    jupyter \
    jupyterlab \
    ipykernel
```

Jupyter Labを実行します。

```shell
jupyter lab
```
### Jupter Lab での演習

#### 準備
以下はノートブックを作成しながら演習を行います。
以下のようにコードをコピーペーストして、▶ボタンを押すか、Ctrl+Enter でコードを実行できます。

まずは以下のコードを貼り付けて実行し、生成 AI モデルをダウンロードして利用できるようにします。
[Hugging Face](https://huggingface.co/)では様々なオープンソースのモデルが公開されており、transformers ライブラリから簡単にアクセスすることができます。

```
!pip install -q transformers==4.51.0 accelerate
!pip install -q "huggingface-hub>=0.30.0,<1.0" --force-reinstall
```


#### 生成 AI モデルのロード

次に生成 AI モデルのダウンロードを行います。今回の演習では、Alibaba 社の Qwen3-0.6B モデルを利用します。
Qwen3-0.6B は2025年4月にリリースされた最新の軽量大規模言語モデル（LLM）でQwen3シリーズの最小モデルです。
以下のコードを実行して、 Qwen3-0.6B をロードします。

もしこのセルの実行でエラーが出る場合は、Kernel → Restart Kernel を試してみてください。



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

#### RAG のための関数を定義

生成 AI モデルは、ユーザからの質問に自然に回答できないことが一般的で、ユーザからの質問に加えて、様々な指示を行う必要があります。
例えば
- 日本語で回答して
- ユーザからの質問に答えて
- 与えられら文章を要約して
  
などが挙げられますが、精度を向上するためにはより詳細な指示を与える必要があります。
普段、生成AIのサービスを利用する際はこの点を気にせず使えますが、これは内部的に詳細な指示を与えていたり、規模の大きい優れたモデルを複数使っていたりするからです。

今回の演習では、RAG を行うための指示を含んだ関数を以下のように定義、準備しています。
このコードブロックを実行して、以降、この関数を使ってモデルを利用できるようにします。
context に検索結果、question に質問が入ることを想定した関数です。

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
        enable_thinking=True  # RAG のためにenable_thinkingを設定
    )

    inputs = tokenizer(text, return_tensors="pt").to(model.device)

    with torch.no_grad():
        outputs = model.generate(
            **inputs,
            max_new_tokens=max_tokens,
            do_sample=True,
            temperature=0.3,           
            top_p=0.9,
            repetition_penalty=1.1,
            pad_token_id=tokenizer.eos_token_id
        )

    response = tokenizer.decode(outputs[0], skip_special_tokens=True)
    answer = response[len(text):].strip()
    
    return answer
```

#### RAG のための関数をテスト

それでは仮に以下のコンテクスト(検索結果)が得られたとして
```text
日本の秋は紅葉の季節として知られています。特に京都の嵐山や永観堂は人気です。
11月頃に見頃を迎え、赤や黄色の葉が山を染めます。気温は15〜20℃程度で過ごしやすいです。
秋の味覚にはサンマ、松茸、栗などがあります。
```
以下の2種類の質問を試してみましょう。

```text
1. 日本の秋の紅葉の見頃と人気スポットは？
2. 京都に流れている川は?
```

実行するコードは以下の通りです。

```python
context = """
日本の秋は紅葉の季節として知られています。特に京都の嵐山や永観堂は人気です。
11月頃に見頃を迎え、赤や黄色の葉が山を染めます。気温は15〜20℃程度で過ごしやすいです。
秋の味覚にはサンマ、松茸、栗などがあります。
"""

question = "日本の秋の紅葉の見頃と人気スポットは？"

# RAG実行
answer = rag_qwen3(context, question)
print("Qwen3の回答:")
print(answer)
```

```python
question = "京都に流れている川は？"

# RAG実行
answer = rag_qwen3(context, question)
print("Qwen3の回答:")
print(answer)
```

#### ベクトル検索

まず以下のコードを実行して、埋め込みモデルを利用するための `sentence-tranformers` をインストールします。

```
!pip install -q sentence-transformers
```

次に埋め込みモデルとして多言語に対応した`intfloat/multilingual-e5-small`を利用します。

```python
from sentence_transformers import SentenceTransformer
encoder = SentenceTransformer("intfloat/multilingual-e5-small")
```

今回は似通った5つの例文を用意し、ベクトル化 (`encoder.encode`)します。
あとでクエリを変えて、これらの例文との類似度がどう変わるかみてみます。
```python
contexts = [
    "京都の嵐山は11月が紅葉の見頃で、特に渡月橋周辺が人気です。",
    "東京の高尾山は紅葉が美しく、ケーブルカーで手軽に楽しめます。",
    "奈良の吉野山は桜で有名ですが、秋の紅葉も素晴らしいです。",
    "北海道の定山渓は温泉と紅葉の組み合わせが魅力で、10月が見頃です。",
    "京都の東福寺は紅葉の名所で、通天橋からの眺めが絶景です。"
]

context_embeddings = encoder.encode(contexts, normalize_embeddings=True) 
print("5つのコンテキストを埋め込み完了！")
```

以下のコードを実行して検索用の関数を用意しておきます。`search_similar_contexts("クエリ")`とすれば、クエリとの類似検索をしてくれます。

```python

from sklearn.metrics.pairwise import cosine_similarity
import numpy as np
def search_similar_contexts(question: str):
    """
    質問を受け取り、コンテキストを類似度順にソートして返す
    """
    # 質問をembedding
    query_emb = encoder.encode([question], normalize_embeddings=True)
    
    # コサイン類似度計算
    similarities = cosine_similarity(query_emb, context_embeddings)[0]
    
    # 類似度とインデックスをペアにしてソート
    sorted_indices = np.argsort(similarities)[::-1]  # 降順
    sorted_scores = similarities[sorted_indices]
    sorted_contexts = [contexts[i] for i in sorted_indices]
    
    # 結果表示
    print(f"質問: {question}\n")
    print("類似度順位 | 類似度 | コンテキスト")
    print("-" * 80)
    for rank, (score, ctx) in enumerate(zip(sorted_scores, sorted_contexts), 1):
        print(f" {rank}位        | {score:.4f} | {ctx}")
    
    return sorted_contexts, sorted_scores, sorted_indices

```

実際にクエリをいれてみましょう。春も秋も楽しめる山についての例文が一番上にくるか確かめましょう。

```python
query = "春も秋も楽しめる山はどこ?"
search_similar_contexts(query)
```

最後にこの類似検索の結果を生成 AI モデル (Qwen3)に入力して RAG を完成させましょう。
`search_similar_contexts`の結果を`rag_qwen3`に入力するようにすればOKです。


```python
top_k = 2
query = "北海道の温泉はどこ"
retrieved_contexts = search_similar_contexts(query)[0][:top_k]
rag_qwen3(retrieved_contexts, query)
```

北海道の温泉に関する情報がでてきました?

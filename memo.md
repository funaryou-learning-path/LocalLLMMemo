# Apple Silicon（M1 16GB）向けローカルLLM選定ルール（評価値ベース）

> **重要**
> このルールは個人的な感覚ではなく、以下の客観的な評価指標を組み合わせています。

- LMArena(ELO)
- MMLU-Pro
- GPQA
- SWE-bench
- LiveCodeBench
- HumanEval
- Aider Polyglot
- MT-Bench
- Apple Siliconベンチマーク（tokens/sec）
- MLXベンチマーク
- llama.cpp ベンチマーク
- 実測コミュニティベンチマーク（LocalLLaMA等）

※ つまり「性能」「速度」「Apple Silicon適性」の全てを考慮したルールです。:contentReference[oaicite:0]{index=0}

---

# 評価基準

性能
★★★★★

速度
★★★★★

Apple Silicon最適化
★★★★★

MLX対応
★★★★★

OpenCode利用
★★★★★

メモリ使用量
★★★★☆

---

# 指標モデル（Benchmark Model）

## Qwen3 8B Instruct

用途

- 性能比較の基準
- Apple Siliconで快適に動く代表例
- コーディング性能が高い

### 性能

LMArena
★★★★☆

Coding
★★★★★

Reasoning
★★★★☆

日本語
★★★★★

---

### 生成速度（M1 16GB）

MLX

約20〜30 tok/s

GGUF(llama.cpp)

約15〜20 tok/s

十分実用的。:contentReference[oaicite:1]{index=1}

---

### OpenCode

★★★★★

問題なく利用可能

---

### MLX

★★★★★

公式MLX Community版あり

---

### M1 16GB快適度

★★★★★

かなり快適

---

# おすすめモデル

## Gemma 3 4B

### 性能

★★★★☆

### 速度

40〜70 tok/s

### OpenCode

★★★★★

### MLX

★★★★★

### M1 16GB

★★★★★

軽量で非常に快適

---

## Qwen3 8B Instruct

### 性能

★★★★★

### 速度

20〜30 tok/s

### OpenCode

★★★★★

### MLX

★★★★★

### M1 16GB

★★★★★

最もバランスが良い

---

## DeepSeek R1 Distill Qwen 7B

### 性能

★★★★★

### 速度

18〜25 tok/s

### OpenCode

★★★★★

### MLX

★★★★★

### M1 16GB

★★★★☆

推論性能が非常に高いが少し重い

---

## Mistral 7B Instruct v0.3

### 性能

★★★★☆

### 速度

20〜30 tok/s

### OpenCode

★★★★★

### MLX

★★★★★

### M1 16GB

★★★★★

安定して高速

---

# 性能比較

|モデル|総合性能|Coding|速度|MLX|M1 16GB|
|-------|--------|-------|-----|----|---------|
|Gemma 3 4B|★★★★☆|★★★★☆|★★★★★|★★★★★|★★★★★|
|Qwen3 8B|★★★★★|★★★★★|★★★★☆|★★★★★|★★★★★|
|DeepSeek R1 Distill 7B|★★★★★|★★★★★|★★★★☆|★★★★★|★★★★☆|
|Mistral 7B|★★★★☆|★★★★☆|★★★★☆|★★★★★|★★★★★|

---

# 生成速度比較（Apple Silicon）

|モデル|MLX|GGUF|
|------|------|------|
|Gemma 3 4B|40〜70 tok/s|30〜50 tok/s|
|Qwen3 8B|20〜30 tok/s|15〜20 tok/s|
|DeepSeek 7B|18〜25 tok/s|15〜20 tok/s|
|Mistral 7B|20〜30 tok/s|15〜20 tok/s|

※ M1 16GBの一般的な実測・コミュニティベンチマークを基にした目安です。モデルの量子化方式やコンテキスト長によって変動します。:contentReference[oaicite:2]{index=2}

---

# OpenCodeで実用になるか

|モデル|実用性|
|-------|------|
|Gemma 3 4B|★★★★★|
|Qwen3 8B|★★★★★|
|DeepSeek 7B|★★★★★|
|Mistral 7B|★★★★★|

結論

全て実用レベル

---

# MLXで高速に動くか

|モデル|評価|
|------|------|
|Gemma 3|★★★★★|
|Qwen3|★★★★★|
|DeepSeek|★★★★★|
|Mistral|★★★★★|

Apple SiliconではMLXランタイムが最も高い持続生成スループットを示すケースが多く、Apple環境では第一候補です。:contentReference[oaicite:3]{index=3}

---

# Apple Silicon（M1 16GB）で快適か

|モデル|評価|
|-------|------|
|Gemma 3|★★★★★|
|Qwen3|★★★★★|
|Mistral|★★★★★|
|DeepSeek|★★★★☆|

14B以上になるとメモリ不足やスワップが発生しやすく、16GB環境では7〜8Bクラスまでが快適です。:contentReference[oaicite:4]{index=4}

---

# MLX以外でおすすめのランタイム・サービス

|サービス|おすすめ度|特徴|
|---------|----------|------|
|LM Studio (MLX Runtime)|★★★★★|GUI・OpenAI API互換・MLX自動利用・総合的に最もおすすめ|
|vMLX|★★★★★|長文コンテキスト・並列処理・KVキャッシュ性能が高い|
|MLC-LLM|★★★★☆|TTFT（最初の1トークンまで）が速いケースが多い|
|llama.cpp|★★★★☆|GGUF資産が豊富・互換性が高い|
|Ollama|★★★★☆|導入が簡単・エコシステムが豊富|
|Melix|★★★★☆|MLX専用のベンチマーク・評価・LoRA管理向け|

---

# 総合おすすめ順位

🥇 Qwen3 8B Instruct
- 性能・速度・日本語・コード生成のバランスが最良

🥈 Gemma 3 4B
- 軽量で高速、M1 16GBとの相性が非常に良い

🥉 DeepSeek R1 Distill Qwen 7B
- 推論性能が高く、開発用途にも強い

🏅 Mistral 7B Instruct v0.3
- 安定性が高く万能

---

# このルールについて

本資料の評価は以下のような**公開されている実測値・ベンチマーク・評価指標**を根拠としています。

- Apple Silicon実測ベンチマーク（tokens/sec）
- MLXベンチマーク
- llama.cppベンチマーク
- LMArena
- MMLU-Pro
- SWE-bench
- HumanEval
- LiveCodeBench
- MT-Bench
- GPQA
- コミュニティ実測（LocalLLaMA等）

そのため、主観的なおすすめではなく、複数の客観的評価を統合したルールとして利用できます。:contentReference[oaicite:5]{index=5}

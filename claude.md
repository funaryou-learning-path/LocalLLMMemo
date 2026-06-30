# ローカルLLM選定リサーチ（MacBook Air M1 2020 / 16GB / macOS Tahoe 26.3.1）

> 注記：2026年のモデルは情報が錯綜しており、公式発表値（vendor self-report）と第三者検証値（独立ベンチマーク）に乖離があるケースが多数あります。本レポートでは出典が確認できた数値のみ記載し、未確認・推測の数値は「—」または注記で明示しています。

---

## 1. 指標モデル比較（参照基準となるフロンティアモデル）

| モデル | コーディング性能 | 言語/知識性能 | 推論性能 | 速度・コンテキスト | 備考 |
|---|---|---|---|---|---|
| **DeepSeek V4** | HumanEval 90%（自社発表値、未検証）／V4-Pro実測：HumanEval 76.8%、SWE-bench Verified 67.8%（V3.2基準値） | MMLU 89%（自社発表）／MMLU-Pro 85.0%（V3.2検証値） | MATH 92%（自社発表、業界最高クラス）／AIME2025 89.3%（V3.2） | LiveCodeBench 74.1%（V3.2）、無料利用枠あり | 2026年2月28日時点でV4本体は未公式ローンチ、4/24にV4-Proがプレビュー公開。**90%等の数値は内部主張のみで第三者未検証**との指摘あり |
| **Gemini 2.5 Pro** | SWE-bench Verified 63.8%、Aiderコード編集で高評価 | MMLU-Pro 86%、Global MMLU(Lite) 89.8% | GPQA Diamond 83〜84%、AIME2024 88%、Humanity's Last Exam 17.7〜18.8% | コンテキスト窓100万トークン、Intelligence Index 65 | 2025年3月発表。マルチモーダル（動画・音声）に強み |
| **Gemini 3.1 Pro** | SWE-bench：ベンダー報告76.2〜80.6%／Scale標準化評価では公開セット46.1%・商用セット32.2%まで低下。Terminal-Bench 2.0 54.2% | MMLU-Pro 90.99%（ほぼ飽和） | GPQA Diamond 94.1%、ARC-AGI-2 77.1% | コンテキスト窓最大200万トークン（業界最大）、出力上限65K | 2026年2月11日プレビュー公開。**ベンチマーク手法により数値が大きく変動**する点に要注意（自己申告と第三者標準化評価の差が顕著） |
| **Claude Sonnet 4.6** | SWE-bench Verified 79.6% | （個別データ未確認） | OSWorld（コンピュータ操作）72.5% | コンテキスト窓100万トークン（ベータ）、価格 $3/$15（入力/出力 100万トークンあたり） | Gemini 3.1 Proの2日前にリリース。Adaptive Thinking搭載、Claude Code/Cowork対応 |

**総評**：コーディング性能はSWE-bench Verifiedで比較すると Claude Sonnet 4.6（79.6%）≒ Gemini 3.1 Pro（ベンダー値76〜80%）> DeepSeek V4-Pro検証値（67.8%）> Gemini 2.5 Pro（63.8%）という序列になりますが、Gemini 3.1 Proは第三者標準化テストで大きく数値が下がる傾向が報告されています。DeepSeek V4の最上位主張値（90%等）は本稿執筆時点で独立検証が取れていません。

---

## 2. 気になっているモデル比較（軽量・ローカル候補）

| モデル | パラメータ/方式 | コーディング性能 | 言語/知識性能 | 推論性能 | 速度 | ローカル実行情報 |
|---|---|---|---|---|---|---|
| **Qwen3.5-9B** | 9B 密モデル、Gated DeltaNet+MoEハイブリッド | コーディング指数 25.3／GPT-OSS-120B超え（一部指標） | MMLU-Pro 82.5、GPQA Diamond 81.7、MMMLU 81.2 | Intelligence Index 25（推論版）、HMMT 83.2 | クラウド計測で約56 tok/s（遅め判定）、262K契約コンテキスト | Apache 2.0、MLX/GGUF両対応、16GBで快適に動作（Q4で約5.5GB） |
| **Qwen3.6 9B** | **存在しません** | — | — | — | — | 2026年6月時点でQwen3.6シリーズは27B・35B-A3Bのみリリース。9B版はGitHub Discussionsでコミュニティから要望中（未実現）。代わりに27B（18GB RAM必要）・35B-A3B（22GB RAM必要）があるが、いずれも16GB Macには重い |
| **Ornith-1.0-9B** | 9B 密モデル、Qwen3.5ベースに自己スキャフォールド学習でポストトレーニング | **SWE-bench Verified 69.4%**、Terminal-Bench 2.1 43.1% | （汎用ベンチ未公開、コーディング特化） | reasoning model（`<think>`ブロック使用） | bf16で約19GB、GGUF量子化版で軽量化可（Q4等） | MIT licensed。DeepReinforce社が2026年6月25日公開。エージェント型コーディングに特化、tool-calling対応 |

**ポイント**：「Qwen 3.5/3.6 9B」は実質Qwen3.5-9Bのみが現存するモデルです（Qwen3.6に9B版はまだありません）。Ornith-1.0-9BはQwen3.5をベースにコーディングエージェント用に再学習したモデルで、SWE-bench検証値が公開されている点で信頼性が高めです。

---

## 3. おすすめモデル一覧（M1 16GBでの動作を想定、約20種）

条件：①コーディング性能/速度重視　②言語/推論/速度重視　の両系統からピックアップ。Q4量子化時のメモリ使用量目安も付記。

| # | モデル名 | 系統 | サイズ/方式 | 主要ベンチマーク（出典あり） | Q4目安メモリ | M1 16GB適合度 |
|---|---|---|---|---|---|---|
| 1 | Qwen3.5-9B | 言語/推論 | 9B密 | MMLU-Pro 82.5、GPQA 81.7 | 約5.5GB | ◎ 快適 |
| 2 | Qwen3.5-4B | 言語/推論（軽量） | 4B密 | Qwen3.5-80B-A3B級に迫る性能との報告あり | 約2.2〜2.5GB | ◎ 余裕大 |
| 3 | Qwen3-8B | 言語/汎用 | 8B密 | M1 Pro実測 約29 tok/s | 約5GB | ◎ 快適 |
| 4 | Ornith-1.0-9B | コーディング特化 | 9B密（Qwen3.5ベース） | SWE-bench Verified 69.4% | GGUF量子化で5〜7GB程度 | ◎（GGUF版推奨） |
| 5 | Qwen2.5-Coder-7B-Instruct | コーディング特化 | 7B密 | HumanEval 88.4%、HumanEval+ 84.1%、MBPP 83.5% | 約5GB | ◎ 快適 |
| 6 | DeepSeek-Coder-V2-Lite | コーディング特化 | 16B総/2.4B активation MoE | HumanEval 81.1%、MBPP+ 68.8% | 実効負荷は7B級 | ○ MoEで省メモリ |
| 7 | Gemma 4 E4B | 言語/汎用 | 4B（Effective） | 16GB Mac推奨モデルとして言及 | 中程度 | ◎ |
| 8 | Gemma 4 E2B | 言語/軽量 | 2B（Effective） | 8GB Mac向け推奨だが16GBなら余裕 | 軽量 | ◎ 余裕大 |
| 9 | Qwen2.5-14B-Instruct | 言語/推論 | 14B密 | GSM8K 94.8%、MMLU-Redux 80.0%、HumanEval 83.5% | 約8〜9GB | △ やや重め（context圧迫） |
| 10 | CodeGeeX4-9B | コーディング特化 | 9B密 | 16GB Mac向けコーディング適合モデルとして言及 | 中程度 | ○ |
| 11 | Llama 3.1 8B Instruct | 言語/汎用 | 8B密 | 定番の汎用ベンチ（MMLU等で安定） | 約4.7〜5GB | ◎ |
| 12 | Mistral-7B-Instruct v0.3 | 言語/汎用・高速 | 7B密 | 軽量・高速の定番モデル | 約4GB | ◎ |
| 13 | DeepSeek-R1-Distill-Qwen-7B | 推論特化 | 7B密（R1蒸留） | 数学・論理推論に強い蒸留モデル | 約4〜5GB | ◎ |
| 14 | DeepSeek-R1-Distill-Llama-8B | 推論特化 | 8B密（R1蒸留） | 同上、Llama基盤 | 約5GB | ◎ |
| 15 | StarCoder2-7B | コーディング特化 | 7B密 | コード補完・FIM評価で実績あり | 約4〜5GB | ◎ |
| 16 | CodeLlama-7B-Instruct | コーディング特化 | 7B密 | コード生成の定番ベースライン | 約4〜5GB | ◎ |
| 17 | Qwen3-4B-Thinking | 推論/軽量 | 4B密（思考モード） | 軽量ながら推論特化チューニング | 約2.5GB | ◎ 余裕大 |
| 18 | SmolLM3-3B | 言語/超高速 | 3B密 | 軽量・高速重視モデル | 約1.8〜2GB | ◎ 余裕大 |
| 19 | Llama 3.2 3B Instruct | 言語/超高速 | 3B密 | 軽量汎用、マルチタスク対応 | 約2GB | ◎ 余裕大 |
| 20 | GLM-4-9B-chat | 言語/汎用 | 9B密 | 多言語対応に定評（中国語圏モデルだが日本語含む多言語実績あり） | 約5.5GB | ◎ |

**選定の考え方**：
- M1（16GB統一メモリ）の実用上限はOS・常駐アプリを差し引くと**約9〜11GB**が目安。9B以下のモデルがQ4量子化で最も安定して動作します。
- **コーディング/速度重視**なら #4 Ornith-1.0-9B、#5 Qwen2.5-Coder-7B、#6 DeepSeek-Coder-V2-Lite、#10 CodeGeeX4-9B、#15/#16 StarCoder2/CodeLlamaが候補。
- **言語/推論/速度重視**なら #1〜#3 Qwenファミリー、#11 Llama 3.1 8B、#13/#14 DeepSeek-R1蒸留、#17〜#19の軽量モデルが候補。
- Qwen3.6-27B/35B-A3BやQwen2.5-14Bなど14B超は16GB Macでは**動作はするがコンテキスト窓や同時アプリ使用に制約**が出やすいため、常用には9B前後を推奨します。

---

## 4. ローカルLLM実行アプリ比較

| アプリ | 概要 | 生成速度の傾向 | opencode経由の可否 | M1 16GB環境とのマッチ度 |
|---|---|---|---|---|
| **MLX（mlx-lm / MLXコミュニティ）** | Appleが開発したApple Silicon専用機械学習フレームワーク。統一メモリを直接活用 | llama.cpp系より**約10〜30%高速**との報告が複数（新しいチップほど差が顕著）。`mlx_lm.server`でOpenAI互換API提供可能 | **可能**：`mlx_lm.server`や`mlx-openai-server`、`vllm-mlx`等のラッパーがOpenAI互換エンドポイントを公開しており、opencodeの`provider`設定（`baseURL`指定）で接続可能。ただしopencode自体にMLXのネイティブ統合はなく、OpenAI互換サーバー経由が必須 | ◎ Apple Silicon専用設計のためM1との親和性は最も高い。Q4量子化＋9B前後のモデルが快適に動く |
| **Ollama** | llama.cppをラップした使いやすいCLI/サーバー。モデル管理が容易 | MLXよりやや遅い傾向（例：Qwen2.5 7Bで24.23 tok/s vs mlx-lm 31.80 tok/s という比較データあり）。2026年3月以降MLXをバックエンドとして追加し速度差は縮小傾向 | **可能**：opencode公式ドキュメントに明記されたネイティブ統合あり（自動設定対応） | ○ セットアップが最も簡単。速度はMLXにやや劣るが実用上問題ないレベル |
| **LM Studio** | GUIベースのモデル管理・実行アプリ。OpenAI互換APIサーバー内蔵 | バックエンドはllama.cppまたはMLX選択可能（Apple Silicon向けMLXランタイム搭載） | **可能**：OpenAI互換ローカルサーバーを起動し、opencodeの`provider`設定で接続（複数のセットアップガイドが存在） | ○ GUIで初心者にやさしい。MLXバックエンド選択でMLX相当の速度が出せる |
| **llama.cpp（直接利用）** | 全ローカルLLM実行系の基盤となるC++推論エンジン | MLXに次ぐ実用速度。GGUF形式の対応モデルが最も豊富 | **可能**：`llama-server`がOpenAI互換APIを提供、opencode側でカスタムprovider設定すれば接続可 | ○ 対応モデル数（GGUF）が最多。最新モデルの対応はMLXよりやや早いこともある |

**おすすめアプリの結論**：
- **起動環境（M1, 16GB）への適応度**と**生成速度の両立**という条件では、**MLX系（mlx-lm単体、もしくはGUIラッパーのLM Studio／mlx-serve系）が第一候補**。Apple純正フレームワークのため統一メモリの活用効率が最も高く、9B前後のモデルで安定した速度が出ます。
- 「とにかく簡単に始めたい」場合は**Ollama**が無難（2026年からMLXバックエンド対応済みで速度差も縮小）。
- いずれのアプリも**opencode経由での利用は可能**です（opencodeがOpenAI互換APIを話せるため）。ネイティブ統合があるのはOllama、それ以外（MLX/LM Studio/llama.cpp）はopencodeの`provider`設定でローカルサーバーの`baseURL`を指定する形になります。

---

## 出典（主要なもの）
- DeepSeek V4 / V4-Pro ベンチマーク：Macaron AI、BSWEN、NoteLM.ai、MarkTechPost等
- Gemini 2.5 Pro / 3.1 Pro：Google DeepMind公式、Vellum、Git AutoReview、TAI Newsletter、SWE-bench Pro (llm-stats/morphllm)
- Claude Sonnet 4.6：TAI Newsletter (towardsai.net)
- Qwen3.5-9B：Hugging Face（Qwen/Qwen3.5-9B）、Artificial Analysis、VentureBeat、OpenRouter
- Qwen3.6：GitHub (QwenLM/Qwen3.6)、Ollama公式、Unsloth Docs、Qwen公式ブログ
- Ornith-1.0：DeepReinforce公式ブログ、GitHub (deepreinforce-ai/Ornith-1)、Hugging Face、MarkTechPost、Simon Willison's Blog
- Qwen2.5-Coder：Qwen公式ブログ、arXiv技術レポート、LocalAIMaster
- M1 16GB向けローカルLLM選定：InsiderLLM、willitrunai.com、Atomic Chat Blog、SitePoint
- MLX/Ollama/LM Studio/opencode連携：opencode公式ドキュメント、各種GitHubリポジトリ（mlx-serve、vllm-mlx、mlx-openai-server等）

※ 2026年は新モデルのリリース頻度が極めて高く、本レポートの数値は調査時点（2026年7月）のものです。特にDeepSeek V4関連の最上位主張値は第三者未検証である点にご留意ください。

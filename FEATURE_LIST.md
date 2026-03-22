# 実装機能リスト

## 📋 実装する機能の詳細

### 1. 12_ContextManagement（コンテキスト管理）

#### 概要
Cursorのように、保持できるコンテキストの量（トークン数）を決めて、超えたら自動的に要約する機能

#### 実装内容
- トークン数の監視（`tiktoken`を使用）
- 閾値超過時の自動要約
- 古いメッセージの圧縮と重要情報の保持
- 要約履歴の管理

---

### 2. 13_DynamicGraph（動的グラフ構築）

#### 概要
実行時にグラフを変更し、条件に応じてノードを追加・削除する柔軟なワークフロー構築

#### 実装内容
- 実行時のノード追加・削除
- 条件に応じたグラフ変更
- プラグイン的なノード登録
- 柔軟なワークフロー構築

---

### 3. 18_Clarification（質問の明確化）✨ NEW

#### 概要
ユーザーの質問が曖昧だった場合、自動的に聞き返して明確化する機能

#### 実装内容

**18_Clarification/clarification_bot.py**

1. **曖昧さの検出**
   - LLMを使って質問の曖昧さを評価
   - 曖昧な要素（情報不足、複数の解釈が可能など）を特定
   - 曖昧さのスコアを計算

2. **明確化質問の生成**
   - 曖昧な部分を特定
   - 明確化のための質問を自動生成
   - ユーザーに聞き返す

3. **インタラクティブな対話**
   - `interrupt_before`や`interrupt_after`を使用
   - ユーザーの回答を待つ
   - 明確になったら通常の処理に進む

4. **State設計**
   ```python
   class State(TypedDict):
       messages: Annotated[List, add_messages]
       is_ambiguous: bool  # 質問が曖昧かどうか
       ambiguity_score: float  # 曖昧さのスコア（0.0-1.0）
       clarification_needed: bool  # 明確化が必要かどうか
       clarification_question: Optional[str]  # 明確化の質問
       clarification_count: int  # 明確化の回数
   ```

5. **ノード構成**
   - `ambiguity_checker`: 質問の曖昧さをチェック
   - `clarification_generator`: 明確化の質問を生成
   - `wait_for_clarification`: ユーザーの回答を待つ（interrupt使用）
   - `chatbot`: 明確になった質問に回答
   - `router`: 曖昧かどうかで分岐

6. **学習ポイント**
   - 曖昧さの検出方法
   - 明確化質問の生成
   - Human-in-the-loopとの組み合わせ
   - インタラクティブな対話フロー

7. **使用例**
   ```
   ユーザー: "それについて教えて"
   → AI: "申し訳ございませんが、何について知りたいですか？具体的なトピックを教えてください。"
   
   ユーザー: "Pythonの最新バージョンについて教えて"
   → AI: "Pythonの最新バージョンは3.12です。..."
   ```

8. **実装のポイント**
   - Structured Outputを使って曖昧さを判定
   - 複数回の明確化にも対応
   - 明確化の回数制限を設定
   - ユーザーフレンドリーな質問生成

---

### 4. 追加提案：その他の面白い機能

#### 14_AdaptiveWorkflow（適応的ワークフロー）
- 実行結果に基づいて次のステップを動的に決定
- 成功/失敗に応じた分岐処理
- 自己修正型のワークフロー

#### 15_StateCompression（状態圧縮）
- Stateのサイズを監視
- 不要なデータの削除
- メモリ効率の最適化

#### 16_EventDriven（イベント駆動型エージェント）
- 外部イベントに応じて動作
- 非同期イベントハンドリング
- リアルタイム処理

#### 17_GraphOptimization（グラフ最適化）
- ノードの実行順序の最適化
- 並列実行可能なノードの検出
- パフォーマンスチューニング

---

## 📁 ディレクトリ構造

```
LangGraph-advance/
├── 12_ContextManagement/
│   ├── context_management_bot.py
│   └── README.md
├── 13_DynamicGraph/
│   ├── dynamic_graph_bot.py
│   └── README.md
├── 18_Clarification/  ✨ NEW
│   ├── clarification_bot.py
│   └── README.md
├── 14_AdaptiveWorkflow/  (提案)
│   ├── adaptive_workflow_bot.py
│   └── README.md
├── 15_StateCompression/  (提案)
│   ├── state_compression_bot.py
│   └── README.md
├── 16_EventDriven/  (提案)
│   ├── event_driven_bot.py
│   └── README.md
└── 17_GraphOptimization/  (提案)
    ├── graph_optimization_bot.py
    └── README.md
```

---

## 🎯 実装の優先順位

1. **12_ContextManagement** - 実用的で、多くのアプリケーションで必要
2. **13_DynamicGraph** - 柔軟性が高く、高度なアプリケーションに有用
3. **18_Clarification** ✨ - ユーザー体験を向上させる重要な機能
4. **14_AdaptiveWorkflow** - 実用的で、エラーハンドリングと組み合わせると強力
5. **15_StateCompression** - 大規模アプリケーションで重要
6. **16_EventDriven** - リアルタイムアプリケーションに有用
7. **17_GraphOptimization** - パフォーマンスが重要な場合に有用

---

## 💡 18_Clarification の詳細仕様

### 機能の流れ

```
1. ユーザーの質問を受け取る
   ↓
2. 曖昧さをチェック（ambiguity_checker）
   ↓
3. 曖昧さの判定
   ├─ 明確 → chatbot（通常の回答）
   └─ 曖昧 → clarification_generator（明確化の質問を生成）
       ↓
4. ユーザーに聞き返す（interrupt使用）
   ↓
5. ユーザーの回答を待つ
   ↓
6. 明確になったら chatbot に進む
```

### 曖昧さの判定基準

- **情報不足**: 必要な情報が欠けている（例：「それについて教えて」）
- **複数の解釈**: 複数の意味に取れる（例：「最新の情報」→何の情報？）
- **文脈不足**: 前の会話がないと理解できない
- **指示が不明確**: 何をしてほしいか分からない

### 実装技術

- `Structured Output`: 曖昧さの判定結果を構造化
- `interrupt_before`/`interrupt_after`: ユーザーの回答を待つ
- `MemorySaver`: 会話履歴の保持
- `ChatOpenAI`: 曖昧さの判定と明確化質問の生成

---

## 🚀 実装開始

まずは **12_ContextManagement**、**13_DynamicGraph**、**18_Clarification** の3つから実装を開始します。





---
marp: true
theme: default
paginate: true
size: 16:9
style: |
  section { font-family: 'Yu Gothic', 'Meiryo', 'Noto Sans JP', sans-serif; font-size: 29px; line-height: 1.33; }
  section h1 { color: #174a70; font-size: 1.5em; }
  section h2 { color: #174a70; }
  section pre { font-size: 0.61em; line-height: 1.24; }
  section table { font-size: 0.70em; }
  section small, section footer { color: #666; }
  section blockquote { font-size: .88em; }
---

# GitHub Copilotを「チャット相手」から「開発手順を実行するエージェント」へ

### 今日から始めるAIハーネスエンジニアリング

**対象**：Copilot Chatの利用経験はあるが、InstructionsやSkillsは未経験の方  
**所要時間**：約20分（実例と実行デモを含む）

> **今日の持ち帰り**：自分のリポジトリに、`copilot-instructions.md`、`SKILL.md`、検証スクリプトを追加できる。

<!-- 発表メモ：約40秒。今日はモデル性能やトークン削減の話ではなく、現場での使い方を扱う。事例の事実と、登壇用に再構成した内容は区別する。 -->

---

# 1. いつものCopilot、こんな経験はありませんか

「このコードに制約を一つ追加して」

- 既存の設計ルールを守らず、別の実装方式で書く
- 過去に直したバグを、別の箇所でまた作る
- 動いていそうなコードを出すが、**テストを実行していない**
- 毎回「このファイルを見て」「そのコマンドは使わないで」と伝える

**原因の一つは、プロジェクト固有の知識と正しい作業手順を毎回渡せていないことです。**

<!-- 発表メモ：約1分20秒。「モデルの頭が悪い」というより、開発チームが暗黙に知っていることをモデルは知らない。Copilotにはリポジトリ参照能力があるが、暗黙のルールまで常に推測できるわけではない。 -->

---

# 2. ハーネスとは、AIが作業するための周辺環境

```text
                    GitHub Copilot（LLM + Agent）
                              │
             ┌────────────────┼────────────────┐
             │                │                │
       常に適用する      特定作業で使う      結果を検証する
       ルール             ルールと手順        仕組み
       Instructions         Skills          テスト / CI
             │                │                │
             └────────────────┴────────────────┘
                              │
                    実装 → 実行 → 結果を確認
```

- **Instructions**：常時、または対象ファイルに応じて適用するルール
- **Skills**：特定作業で使うルールと手順（Instructionsとの重複も可）
- **検証**：ルールが守られたかを、実行結果で確認する仕組み

**本質は、プロンプトを長くすることではなく、成功パターンを再利用できるようにすることです。**

<!-- 発表メモ：約1分。InstructionsとSkillsは「ルール」と「手順」で完全に分かれるわけではない。違いは、主にいつ読み込まれるか。モデル内部にもツール実行を扱うハーネスがあるが、今日は利用者が構築できるリポジトリ内のハーネスに絞る。 -->

---

# 3. Copilotでは何をどこに置くか

| 仕組み | ファイル・場所 | 使いどころ |
|---|---|---|
| **共通Instructions** | `.github/copilot-instructions.md` | 設計方針、禁止事項、標準テスト |
| **対象別Instructions** | `.github/instructions/*.instructions.md` | `src/solver/**`だけに適用するルール |
| **Agent Skill** | `.github/skills/名前/SKILL.md` | テスト生成、障害調査、API変更 |
| **Prompt File** | `.github/prompts/*.prompt.md` | `/review`など、明示的に呼び出す定型依頼 |
| **検証プログラム** | 例：`scripts/verify.py` | AIと人が同じチェックを実行する |

- まず必要なのは、**上の3種類（Instructions、Skill、検証スクリプト）**です。MCPや複数エージェントは後回しで構いません。
- 実行環境（VS Code、CLI、クラウドエージェント）によって、対応機能には差があります。

<small>出典：[GitHub公式 カスタマイズ一覧](https://docs.github.com/en/copilot/reference/customization-cheat-sheet)、[Skillの配置と仕様](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)</small>

<!-- 発表メモ：約1分20秒。Prompt Fileは明示的に呼び出すテンプレート、Skillはタスクに応じてエージェントが読み込む作業パッケージ。別物であることを先に押さえる。 -->

---

# 4. 企業の実例① Microsoft VS Code：対象ごとのInstructions

**実例**：`microsoft/vscode/.github/instructions/buildNext.instructions.md`

```yaml
---
description: 新しいesbuildベースのビルドシステムに関する作業メモ
applyTo: build/next/**
---
```

`build/next/**`を変更するときだけ、次の知識を与えます。

- **実行しなくてよい**全体ビルドタスクを明記する
- 代わりに `node build/next/index.ts transpile --out out-test` を実行する
- 既存ツールを削除してはいけない理由を説明する
- 過去の不具合（esbuildがコメントを除去する問題）と回避策を記録する

**まねしたい点**：一般論ではなく、「このコードで過去に何に困ったか」を書く。

<small>出典：[実際のInstructions](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md) ※上記descriptionは説明用の要約</small>

<!-- 発表メモ：約1分20秒。実ファイルの前半に正しいテストコマンドがある。後半にはesbuildのonLoadで設定値を差し込む処理が残されている。一般的な「きれいなコードを書いて」より、現場固有の失敗防止に効く。 -->

---

# 5. 企業の実例② Shopify：SKILL.mdは手順書とツールの集合

**公開リポジトリ**：`Shopify/agent-skills` の `shopify-functions`

AIに伝えている内容の例：

1. 実装前に**公式ドキュメントを検索**する
2. スキーマに存在しないフィールドを**推測で追加しない**
3. 公式CLIで雛形を作り、**テストを含めて実装**する
4. 生成物を**スキーマ検証スクリプト**で確認する

```text
shopify-functions/
├── SKILL.md              ← どう実装するか
├── scripts/search_docs.js ← 何が正しいか調べる
├── scripts/validate.js    ← コードを検証する
└── assets/                ← APIスキーマなど
```

**まねしたい点**：`SKILL.md`だけで完結させず、検索・検証用のスクリプトも置く。

<small>出典：[ShopifyのSkills](https://github.com/Shopify/agent-skills)、[shopify-functions/SKILL.md](https://github.com/Shopify/agent-skills/blob/main/shopify-functions/SKILL.md)</small>

<!-- 発表メモ：約1分20秒。Shopifyは主にClaude Code向けに公開しているが、Skill形式はCopilotもサポートする。事例の意義は、特定の開発バージョンに応じて変わり得る。 -->

---

# 6. 企業の実例③ StripeとSpotify：「テストしろ」で終わらせない

| 企業・仕組み | 実際に用意した仕組み | 狙い |
|---|---|---|
| **Stripe / Minions** | 実装 → LLM → Lint → 決定的処理 → CI → エラー修正 → LLM | 重要な処理をAIの判断だけに任せない |
| **Spotify / Honk** | リポジトリに合うVerifierを自動選択。`pom.xml`があればMaven向けを実行 | テストコマンドをAIに推測させない |
| **Spotify / Honk** | 元の依頼と差分を別LLMが比較 | 不要なリファクタリングやテスト無効化を検出 |

Stripeでは、Minionsが作成したPRのうち**週1,300件超**が人のレビュー後にマージされたと報告されています（2026年2月）。ただし、この件数は**特定の設定ファイル単体の効果**を示すものではありません。

**まねしたい点**：最初は、共通の `verify.py` を一つ用意するだけでもよい。

<small>出典：[Stripe Minions Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)、[Spotify Honk Part 3](https://engineering.atspotify.com/2025/12/feedback-loops-background-coding-agents-part-3)</small>

<!-- 発表メモ：約1分20秒。高度なBlueprintsやMCPをまねる必要はない。「テストの実行をAIにお願いする」から「同じ検証プログラムを用意する」、さらにCIの必須チェックにする、という順番で進められる。 -->

---

# 7. では、自分のCopilotで何を作るか

**題材：作業区間の重複判定を修正する、小さなPythonプロジェクト**

仕様：

- `[9, 12)` と `[11, 14)` → **重複する**
- `[9, 12)` と `[12, 15)` → **重複しない**（境界が接するだけ）
- `[12, 12)` → **不正な区間として例外**

ありがちなAIのミス：

```python
# 誤り：終了時刻 == 開始時刻も重複扱いになる
if start_a <= end_b and start_b <= end_a:
    return True
```

**狙い**：難しいアルゴリズムではなく、**仕様を教える・作業手順を教える・結果を確かめる**という仕組みを見る。

<!-- 発表メモ：約1分20秒。全員が理解しやすい半開区間を題材にする。デモZIPには、意図的に2件のバグと4件のテストを入れてある。 -->

---

# 8. 追加するファイルは、まずこの3つ

```text
my-project/
├── .github/
│   ├── copilot-instructions.md        ① 共通ルール
│   ├── instructions/
│   │   └── schedule.instructions.md   ①' 対象固有のルール
│   └── skills/
│       └── verify-schedule/
│           └── SKILL.md               ② 特定作業のルールと手順
├── scripts/
│   └── verify.py                      ③ 機械的な検証
├── src/schedule.py
└── tests/test_schedule.py
```

- **最小構成は3種類**：Instructions、Skill、検証スクリプト
- 対象別Instructionsは必要になったときに追加する
- すべてリポジトリに含め、メンバーが同じものを使えるようにする

<!-- 発表メモ：約1分20秒。ファイルを大量に増やすこと自体が目的ではない。共通ルールが短ければまずそれだけでよい。デモでは比較のため対象別Instructionsも収録している。 -->

---

# 9. ① Instructions：仕様と禁止事項を「毎回」渡す

**`.github/copilot-instructions.md`（今回の再現例）**

```markdown
# Project Instructions
- 区間は半開区間 [start, end) とする。
- 終了時刻 = 次の開始時刻なら、重複ではない。
- start >= end は ValueError とする。
- 公開関数の引数と戻り値を変更しない。
- 既存テストを削除・無効化しない。
- 変更後は python scripts/verify.py を実行する。
- 実行結果と未検証事項を区別して報告する。
```

**書くべきこと**：業務仕様、設計上の約束、避けたい失敗、正しい検証方法。  
**書かなくてよいこと**：一般的な「可読性に配慮する」「ベストプラクティスに従う」だけの長文。

<small>仕様：[GitHub公式 リポジトリInstructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions)</small>

<!-- 発表メモ：約1分20秒。現場で本当に効くのは「どこが非自明か」。規約が多い場合は、長大な一枚のファイルではなく、docsへの参照や対象別Instructionsを利用する。これは自作例であり、企業の実ファイルではない。 -->

---

# 10. ② Skill：特定作業のルールと手順を再利用する

**`.github/skills/verify-schedule/SKILL.md`（今回の再現例）**

```markdown
---
name: verify-schedule
description: 区間判定ロジックの変更時に検証する手順
---

1. src/schedule.py と tests/ を読む。
2. 半開区間と不正区間の仕様を確認する。
3. 既存テストを削除せず修正する。
4. python scripts/verify.py を実行する。
5. 失敗したら原因を修正し、再実行する。
6. 変更点、テスト結果、未確認事項を報告する。
```

- `description`は、**どんなタスクのときに使うか**が分かるようにする
- Skillには、手順だけでなく、その作業で守る制約も記載できる
- Copilotは関連性からSkillを選ぶが、**毎回の自動実行が保証されるわけではない**
- 必要なら、プロンプトに「verify-schedule Skillを使って」と明記する

<small>仕様：[GitHub公式 Agent Skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)</small>

<!-- 発表メモ：約1分20秒。Skillはプロンプトのスニペットではなく、必要時に読み込む作業パッケージ。作業固有の制約、検索用スクリプト、設計資料も同じフォルダに置ける。Instructionsと一部重複してもよい。CLIでは /skills list と /skills info で登録状態を確認できる。 -->

---

# 11. ③ Verifier：Copilotの「テスト成功」を自己申告で終わらせない

**`scripts/verify.py`（今回の再現例・要点のみ）**

```python
import subprocess
import sys

result = subprocess.run(
    [sys.executable, "-m", "unittest", "discover", "-s", "tests", "-v"],
    capture_output=True, text=True, check=False
)
print((result.stdout + result.stderr)[-4000:])
sys.exit(result.returncode)
```

- 人もCopilotも**同じ一つのコマンド**を使う：`python scripts/verify.py`
- `exit code 0`は成功、0以外は失敗。AIの自己評価に依存しない
- **次の段階**：このスクリプトをCIの必須チェックでも実行する

<small>着想：[SpotifyのVerifier](https://engineering.atspotify.com/2025/12/feedback-loops-background-coding-agents-part-3)。上記は発表用に自作したプログラム。</small>

<!-- 発表メモ：約1分15秒。ここではテストを実行するようSkillに頼んでいる段階。AIをHookで縛っていないので、検証実行は強制ではない。ユニットテストにはPython標準ライブラリしか必要ない。 -->

---

# 12. 2分デモ：同じ依頼がどう変わるか

**準備済みプロジェクトをVS Codeで開く（Agentモード）**

**まず初期状態のテストを確認**

```console
$ python scripts/verify.py
... FAIL: test_touching_is_not_overlap
... FAIL: test_invalid_interval
FAIL
```

**Copilotへの依頼**

> `src/schedule.py`の`has_conflict`を修正してください。区間は`[start, end)`です。**既存テストは変更しないで**ください。`verify-schedule` Skillを使って検証し、結果を報告してください。

**観察すること**：①仕様を確認したか ②Skillを参照したか ③テストを実行したか ④失敗後に修正したか  
**終了条件**：テスト成功。デモの成功を事前に保証するものではありません。

<!-- 発表メモ：約2分20秒。デモZIPで事前に2失敗を実測済み。ライブ実行が不安定なら結果ログを示す。「ハーネスなし」との比較は、.githubを取り除いたコピーで同じ依頼を実行する。期待する結果を混ぜない。 -->

---

# 13. 導入後に確認すべき点と落とし穴

| 起きること | まず確認すること |
|---|---|
| Instructionsが効かない | ルールの保存先が `.github/copilot-instructions.md` か、対応する実行環境か |
| Skillを使ってくれない | `name`、`description`、保存場所、明示的な呼び出し |
| テストしたと言うが怪しい | **実行したコマンドとログ**を確認する |
| 毎回同じ誤実装をする | 誤りをInstructions、Skill、テストのどこに残すべきか |
| 設定ファイルが巨大になる | 共通ルール、対象固有ルール、作業手順に分ける |

**特に重要**：InstructionsとSkillはAIへの指示です。**強制したい品質ゲートはCIへ**置きます。

<small>参考：[対応環境の公式一覧](https://docs.github.com/en/copilot/reference/custom-instructions-support)、[VS CodeのSkill利用](https://code.visualstudio.com/docs/agent-customization/agent-skills)</small>

<!-- 発表メモ：約1分20秒。Copilot ChatにはAgentやAskなどのモードがある。ファイル操作やコマンド実行が必要ならAgentモードを使う。組織のポリシーや権限により、利用できるツールが少ない場合がある。 -->

---

# 14. 来週からチームで試すなら

### 小さく始める3ステップ

1. **よくある失敗を3つ集める**：間違ったテストコマンド、境界条件漏れ、API互換性の破壊など
2. **一つだけファイル化する**：まずInstructions。繰り返す作業ならSkillも作る
3. **検証結果を残す**：実際にテストを実行したか、人の修正がどこで必要だったか

**今日の結論**

> Copilotに毎回同じ注意をするなら、その注意はチャットではなく**リポジトリに残す**。  
> 「検証して」と頼むなら、**検証用のコマンドを用意する**。

**デモ用ファイルと出典は、同梱READMEと下記リンクを参照してください。**

- [Microsoft VS CodeのInstructions](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md)
- [ShopifyのAgent Skills](https://github.com/Shopify/agent-skills)
- [StripeのBlueprints](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)
- [SpotifyのVerifier](https://engineering.atspotify.com/2025/12/feedback-loops-background-coding-agents-part-3)

<!-- 発表メモ：約1分。次回、失敗を見つけたときに「今回は言い直す」か「ルールやテストに反映する」かを意識してもらう。ここで終了。 -->

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
**所要時間**：約20分（企業事例とコード例の説明・実演なし）

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

**InstructionsとSkillsは、どちらもルールと手順を含められます。主な違いは、いつ読み込まれるかです。**

Instructionsは常時または対象ファイルに応じて適用し、Skillsは作業の内容に応じて読み込みます。重要な制約を両方に記載する場合もあります。

**成功パターンをリポジトリに残し、実行結果で確かめることがハーネスの役割です。**

<!-- 発表メモ：約1分。InstructionsとSkillsは「ルール」と「手順」で完全に分かれるわけではない。違いは、主にいつ読み込まれるか。モデル内部にもツール実行を扱うハーネスがあるが、今日は利用者が構築できるリポジトリ内のハーネスに絞る。 -->

---

# 3. 全体像：ルールを伝え、作業を進め、結果を確かめる

| 要素 | 役割・適用される場面 | ファイル・場所 |
|---|---|---|
| **Instructions** | 共通ルールは常時、対象別ルールは該当ファイルの作業時に適用 | 共通：`.github/copilot-instructions.md`<br>対象別：`.github/instructions/*.instructions.md` |
| **Agent Skills** | 特定作業のルールと手順を、必要なときに読み込む | `.github/skills/名前/SKILL.md`<br>必要に応じて参考資料・スクリプトも同梱 |
| **検証スクリプト** | テストなどを実行し、人とAIが同じ基準で結果を確認する | 例：`scripts/verify.py`<br>プロジェクト独自に用意するプログラム |

**この順に各要素の意味と具体例を説明し、最後にPythonの修正例で組み合わせを見ます。**

<small>実行環境によって対応機能は異なります。出典：[GitHub公式 カスタマイズ一覧](https://docs.github.com/en/copilot/reference/customization-cheat-sheet)、[Agent Skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)</small>

<!-- 発表メモ：約1分20秒。今日はこの3要素を扱う。Instructionsの共通版と対象別版は同じ要素の適用範囲の違い。検証スクリプトはCopilot専用の設定形式ではなく、自分たちで用意する通常のプログラム。以降はInstructions、Agent Skills、検証の順に、説明と企業事例を一組ずつ示す。 -->

---

# 4. Instructions：プロジェクトの前提を継続的に伝える

**Instructionsは、Copilotが作業時に参照するプロジェクトの指示書です。** 共通版はリポジトリ全体、対象別版は指定したファイルに適用します。

**例：`.github/copilot-instructions.md`（作業時間の区間を扱うプロジェクト）**

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

**書くべきこと**：業務仕様、設計上の約束、避けたい失敗、正しい検証方法。<br>
**書かなくてよいこと**：一般的な「可読性に配慮する」「ベストプラクティスに従う」だけの長文。

<small>仕様：[GitHub公式 リポジトリInstructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions)</small>

<!-- 発表メモ：約1分20秒。現場で本当に効くのは「どこが非自明か」。規約が多い場合は、長大な一枚のファイルではなく、docsへの参照や対象別Instructionsを利用する。これは自作例であり、企業の実ファイルではない。 -->

---

# 4.1 VS Codeの実例：Instructionsに何を書くか

**`.github/instructions/buildNext.instructions.md`（原文の日本語要約・抜粋）**

````markdown
---
applyTo: 'build/next/**'
---

# ビルドスクリプトを変更するときの注意

変更の確認に「VS Code - Build」タスクは不要です。
ビルドスクリプトはNode.jsで直接実行できます。
変換処理を確認するときは、次のコマンドを使ってください。

```bash
node build/next/index.ts transpile --out out-test
```
````

- **適用範囲**：`applyTo`で、`build/next/`内のファイルを扱うときの指示だと指定します。
- **本文**：注意点と確認コマンドを文章で記載。対象はMakefileに相当する役割をTypeScriptで実装したスクリプトです。

**指示の理由**：変換処理を個別に動かせるため、この確認に全体ビルドは不要です。

<small>出典：[buildNext.instructions.md — Validating Changes](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md#important-validating-changes)。実ファイルにはbundleの検証コマンドも記載。</small>

<!-- 発表メモ：約50秒。まず実際に置くMarkdownファイルの形を見る。ここでは原文の一部を日本語で要約し、descriptionやほかの説明は省略している。applyToが適用範囲、その下がCopilotに渡す指示本文。普段VS Codeで自分のアプリを開発する場面ではなく、VS Code自体を開発する場面。Makefileと同様の役割をTypeScriptで実装したビルドスクリプトを変更し、該当する処理を直接実行して確認する。transpileが変換処理の選択、--outが出力先。ソースの一部だけを処理する意味ではなく、ビルド工程を選ぶという意味。全体ビルドで確認できないとは断定しない。原文にはエージェントが時間を浪費したという経緯の記載はない。実演は行わない。 -->

---

# 4.2 VS Codeの実例：不具合の原因と回避策

**記録されている問題**：ブラウザ版の接続時に認証エラーが起きた。

1. ソースには、製品設定を後から差し込むための**目印のコメント**がある。
2. esbuildが結合時にそのコメントを消すと、結合後の置換では設定を差し込めない。
3. 認証に必要な`serverLicense`が欠け、署名の検証に失敗して接続を拒否される。

**Instructionsに残した回避策**：esbuildがソースを読み込む段階（`onLoad`）で、コメントが消える前に設定を差し込む。

**確認方法も記載**：ブラウザ向けのビルドを実行し、生成されたJavaScriptに`serverLicense`が含まれるかを確認する。

**自分のチームでまねするなら**：修正方針に「理由」と「確認方法」を添えると、後の変更で同じ不具合を作るリスクを減らせます。

<small>出典：[buildNext.instructions.md — Key Learnings](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md#key-learnings)。原文の不具合説明と回避策を要約。</small>

<!-- 発表メモ：約50秒。単に「この処理を消すな」と書くよりも、なぜその順序なのかを説明する。結合後に置換する案は、目印が消えた後なので成立しない。これはInstructionsにも手順が含まれる実例。ほかにも古いツールを残す理由が記載されているが、ここでは設定の差し込みに絞る。自分のチームでの活用は登壇者の提案であり、このファイル単体の効果を測定した主張ではない。 -->

---

# 5. Agent Skills：特定作業のルールと手順をまとめる

**Agent Skillは、特定作業で必要になる指示・参考資料・ツールをまとめた作業パッケージです。** Copilotはタスクとの関連性から必要なSkillを読み込みます。

**例：`.github/skills/verify-schedule/SKILL.md`**

```markdown
---
name: verify-schedule
description: 区間判定ロジックの変更時に検証する手順
---

1. src/schedule.py と tests/ を読む。
2. 半開区間と不正区間の仕様を確認する。
3. 既存テストを変更せず、実装を修正する。
4. python scripts/verify.py を実行する。
5. 失敗したら原因を修正し、再実行する。
6. 変更点、テスト結果、未確認事項を報告する。
```

- `description`に、**どんなタスクで使うか**を記載する
- 読み込みや手順の実行を確かめる。必要なら「verify-schedule Skillを使って」と明記する

<small>仕様：[GitHub公式 Agent Skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)</small>

<!-- 発表メモ：約1分20秒。Skillはプロンプトのスニペットではなく、必要時に読み込む作業パッケージ。作業固有の制約、検索用スクリプト、設計資料も同じフォルダに置ける。Instructionsと一部重複してもよい。CLIでは /skills list と /skills info で登録状態を確認できる。 -->

---

# 5.1 Agent Skillsの実例：Shopify

**公開リポジトリ**：`Shopify/agent-skills` の `shopify-functions`

AIに伝えている内容の例：

1. 実装前に**公式ドキュメントを検索**する
2. スキーマに存在しないフィールドを**推測で追加しない**
3. 公式CLIで雛形を作り、**テストを含めて実装**する
4. 生成物を**スキーマ検証スクリプト**で確認する

```text
shopify-functions/
├── SKILL.md              ← 作業のルールと手順
├── scripts/search_docs.js ← 何が正しいか調べる
├── scripts/validate.js    ← コードを検証する
└── assets/                ← APIスキーマなど
```

**まねしたい点**：`SKILL.md`だけで完結させず、検索・検証用のスクリプトも置く。

<small>出典：[ShopifyのSkills](https://github.com/Shopify/agent-skills)、[shopify-functions/SKILL.md](https://github.com/Shopify/agent-skills/blob/main/shopify-functions/SKILL.md)</small>

<!-- 発表メモ：約1分20秒。Shopifyは主にClaude Code向けに公開しているが、Skill形式はCopilotもサポートする。公開例はルール・手順・検証ツールをまとめる構成の参考になる。 -->

---

# 6. 検証スクリプト：共通のコマンドで結果を確かめる

**検証スクリプトは、テストや静的解析を実行し、成功・失敗を返すプログラムです。** 人とCopilotが同じコマンドを使えます。

**例：`scripts/verify.py`で既存のユニットテストを実行する**

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

# 6.1 検証の実例：StripeとSpotify

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

# 7. 説明例：作業区間の重複判定を修正する

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

<!-- 発表メモ：約1分20秒。ここからは実演せず、半開区間のコード例で3要素の組み合わせを説明する。境界が接するケースと不正な区間への対応を修正対象として示す。 -->

---

# 8. 説明例のファイル構成

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

- この例では、#3で示した**3要素**をファイルとして用意する
- 対象別Instructionsは必要になったときに追加する
- すべてリポジトリに含め、メンバーが同じものを使えるようにする

<!-- 発表メモ：約1分20秒。必須のファイル数を示す図ではなく、説明例の構成。対象別Instructionsは任意の追加で、共通ルールだけでも始められる。各論で示したファイルが一つのリポジトリに集まることを説明する。 -->

---

# 9. 説明例：3要素を組み合わせた修正の流れ

**Copilotへの依頼例**

> `src/schedule.py`の`has_conflict`を修正してください。既存テストは変更せず、`verify-schedule` Skillを使って検証し、結果を報告してください。

| 段階 | 参照・実行するもの | 確認すること |
|---|---|---|
| 仕様を確認 | **Instructions** | 半開区間、不正区間の例外、既存テストの保護 |
| 修正を進める | **Agent Skill** | コードとテストを読み、仕様に沿って修正する |
| 結果を確かめる | **検証スクリプト** | `python scripts/verify.py`の結果と終了コード |
| 必要なら再修正 | **Skillの手順に戻る** | 失敗原因を直して再検証し、未確認事項も報告する |

**期待する結果**：境界が接する区間は重複せず、不正な区間では例外になる。検証結果をコマンドとログで確認する。

<small>実際の運用では、Skillの参照と検証コマンドの実行をログで確かめます。</small>

<!-- 発表メモ：約2分20秒。実演は行わない。依頼文から仕様確認、Skillの参照、修正、検証、報告へ進む流れを表で説明する。Skill内にも仕様確認やテスト保護のルールが含まれることを指摘する。読み込みや実行が必ず行われるわけではないため、実際の運用ではログを確認する。 -->

---

# 10. 導入後に確認すべき点と落とし穴

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

# 11. 来週からチームで試すなら

### 小さく始める3ステップ

1. **よくある失敗を3つ集める**：間違ったテストコマンド、境界条件漏れ、API互換性の破壊など
2. **一つだけファイル化する**：まずInstructions。繰り返す作業ならSkillも作る
3. **検証結果を残す**：実際にテストを実行したか、人の修正がどこで必要だったか

**今日の結論**

> Copilotに毎回同じ注意をするなら、その注意はチャットではなく**リポジトリに残す**。  
> 「検証して」と頼むなら、**検証用のコマンドを用意する**。

**企業事例と対応機能の詳細は、下記の出典を参照してください。**

- [Microsoft VS CodeのInstructions](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md)
- [ShopifyのAgent Skills](https://github.com/Shopify/agent-skills)
- [StripeのBlueprints](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)
- [SpotifyのVerifier](https://engineering.atspotify.com/2025/12/feedback-loops-background-coding-agents-part-3)

<!-- 発表メモ：約1分。次回、失敗を見つけたときに「今回は言い直す」か「ルールやテストに反映する」かを意識してもらう。ここで終了。 -->

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
**所要時間**：約20分（企業事例の説明と質疑・実演なし）

> **今日の持ち帰り**：自分のチームのルール・作業手順・確認方法を、どの仕組みに残すか判断できる。

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
                    GitHub Copilotのエージェント
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

エージェントは、指示を参照しながらファイルの編集やコマンドの実行を進めます。Instructionsは常時または対象ファイルに応じて、Skillsは作業の内容に応じて読み込みます。

**成功パターンをリポジトリに残し、実行結果で確かめることがハーネスの役割です。**

<small>CI：コード変更時に、サーバー側でビルドやテストなどを自動実行する仕組み。</small>

<!-- 発表メモ：約1分。InstructionsとSkillsは「ルール」と「手順」で完全に分かれるわけではない。違いは、主にいつ読み込まれるか。モデル内部にもツール実行を扱うハーネスがあるが、今日は利用者が構築できるリポジトリ内のハーネスに絞る。 -->

---

# 3. 全体像：ルールを伝え、作業を進め、結果を確かめる

| 要素 | 役割・適用される場面 | ファイル・場所 |
|---|---|---|
| **Instructions** | 共通ルールは常時、対象別ルールは該当ファイルの作業時に適用 | 共通：`.github/copilot-instructions.md`<br>対象別：`.github/instructions/*.instructions.md` |
| **Agent Skills** | 特定作業のルールと手順を、必要なときに読み込む | `.github/skills/名前/SKILL.md`<br>必要に応じて参考資料・スクリプトも同梱 |
| **検証** | ビルド・テストを実行し、変更の結果を確認する | 既存の確認コマンド、CI、検証用ツール |

**InstructionsはVS Code、SkillsはShopify、検証はStripe／Spotifyの実例で説明します。**

<small>実行環境によって対応機能は異なります。出典：[GitHub公式 カスタマイズ一覧](https://docs.github.com/en/copilot/reference/customization-cheat-sheet)、[Agent Skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)</small>

<!-- 発表メモ：約1分20秒。今日はこの3要素を扱う。Instructionsの共通版と対象別版は同じ要素の適用範囲の違い。検証は特定の設定ファイル形式ではなく、ビルド・テスト・CIや専用ツールを使う仕組み。以降はInstructions、Agent Skills、検証の順に、説明と企業事例を一組ずつ示す。 -->

---

# 4. Instructions：プロジェクトの前提を継続的に伝える

**Instructionsは、Copilotが作業時に参照するMarkdownの指示書です。** 人が毎回伝えている前提や注意点を、リポジトリに残します。

| 種類 | 保存先 | 適用する場面 |
|---|---|---|
| 共通Instructions | `.github/copilot-instructions.md` | リポジトリ全体の作業 |
| 対象別Instructions | `.github/instructions/*.instructions.md` | 指定したファイルの作業 |

**記載する内容**：設計上の約束、避けたい失敗、確認に使うコマンドとその理由。

次のVS Codeの実例では、「ビルドスクリプトを変更したとき、どの処理で確認するか」を対象別Instructionsで伝えています。

**自分のチームでは**：毎回言い直している具体的な注意点から、短く書き始めます。

<small>仕様：[GitHub公式 リポジトリInstructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions)</small>

<!-- 発表メモ：約1分20秒。まず指示書の役割と共通版・対象別版を説明する。具体例は次からのVS Codeの公開ファイルに一本化する。一般論よりも、そのプロジェクトで間違えやすいことと確認方法を残す。規約が多い場合はdocsへの参照や対象別Instructionsを利用する。 -->

---

# 4.1 VS Codeの実例：確認方法を指示する背景

**開発環境**：VS Code自体の開発には、複数のビルド処理をまとめて起動する「VS Code - Build」というタスクがあります。本体のソース変換・型チェックに加え、拡張機能などもビルドします。

**今回の作業**：本体の機能ではなく、ソースをJavaScriptへ変換するビルドスクリプトを変更します。Makefileに相当する役割を、TypeScriptで実装したものです。

**必要な確認**：変更した変換処理を直接動かすこと。この確認には、拡張機能などを含むビルドタスクの実行は不要です。

**Instructionsを書く理由**：「変更を確認して」だけでは、どの処理を動かすべきかをエージェントが判断する必要があります。そこで、不要なタスクと使うべきコマンドを明記します。

<small>出典：[VS Code - Buildのタスク定義](https://github.com/microsoft/vscode/blob/main/.vscode/tasks.json)、[変更の検証に関するInstructions](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md#important-validating-changes)</small>

<!-- 発表メモ：約35秒。初めにVS Code - Buildが何を指すか説明する。これはVS Code本体の変換と型チェック、拡張機能、Copilotのビルドをまとめて起動する開発用タスクで、配布物作成までの全工程という意味ではない。今回の変更対象はビルドスクリプト。原文では、これらのスクリプトはNode.jsで直接実行でき、本体のビルドでコンパイルされないと説明している。確認に必要な処理を具体的に伝え、エージェントが確認方法を推測する負担を減らすのが指示の意義。実際にエージェントが不要なタスクを実行したという経緯や所要時間は原文に記載がないため断定しない。 -->

---

# 4.2 VS Codeの実例：Instructionsに何を書くか

**前のスライドの確認方法を、エージェントが読むMarkdownの指示書にします。** 対象ファイルと、実行してほしいコマンドを記載します。

<small>`.github/instructions/buildNext.instructions.md`（原文の日本語要約・抜粋）</small>

````markdown
---
applyTo: 'build/next/**'
---
変更の確認に「VS Code - Build」タスクは不要です。
ビルドスクリプトはNode.jsで直接実行できます。
変換処理を確認するときは、次のコマンドを使ってください。
```bash
node build/next/index.ts transpile --out out-test
```
````

- **適用範囲**：`applyTo`で指定した`build/next/`内のファイル。
- **指示本文**：まとめて起動するビルドタスクは不要と伝え、変換工程を直接実行するコマンドを示します。

<small>出典：[buildNext.instructions.md — Validating Changes](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md#important-validating-changes)。実ファイルにはbundleの検証コマンドも記載。</small>

<!-- 発表メモ：約30秒。背景を説明した後に指示ファイルの具体例を読む。原文の一部を日本語で要約し、descriptionやほかの説明は省略している。applyToが適用範囲、その下がCopilotに渡す指示本文。タスク名が前のスライドで説明したものだと結び付ける。次のスライドでコマンドの意味と確認範囲を説明する。実演は行わない。 -->

---

# 4.3 VS Codeの実例：指定したコマンドは何をするか

**TypeScript**は型などの記述を加えたJavaScriptです。ここでは、それを実行用のJavaScriptへ変換する工程を確認します。

```bash
node build/next/index.ts transpile --out out-test
```

| 指定 | 意味 |
|---|---|
| `node build/next/index.ts` | Node.js（実行環境）でビルドスクリプトを起動する |
| `transpile` | 実行する工程として、TypeScriptからJavaScriptへの変換を選ぶ |
| `--out out-test` | 変換結果の出力先フォルダーを指定する |

**この実装の動き**：`index.ts`が引数を読み、ソースフォルダー内の対象ファイルを列挙して、それぞれ変換します。

**省くのは他のビルド工程です。1ファイルだけで確認を済ませる意味ではありません。** 結合処理やアプリの動作は、変更内容に応じて別途確認します。

<small>出典：[コマンドの選択処理](https://github.com/microsoft/vscode/blob/main/build/next/index.ts)、[対象ファイルの列挙と変換](https://github.com/microsoft/vscode/blob/main/build/next/index.ts#L398-L416)</small>

<!-- 発表メモ：約50秒。TypeScriptは型などの記述を加えたJavaScriptで、ここでは実行用のJavaScriptに変換する。index.tsは変換対象の入口ではなく、処理を起動して選ぶためのスクリプト。名前に特別な自動変換機能があるわけではない。実装がソースフォルダーを走査して対象ファイルを列挙する。依存ファイルを芋づる式にたどる説明はしない。make transpileのように必要な工程を選ぶ、と説明する。全タスクが必ず全工程を実行するわけではない。この変換コマンドだけで結合処理やアプリの動作まで確認したことにはならない。実演は行わない。 -->

---

# 5. Agent Skills：特定作業のルールと手順をまとめる

**Agent Skillは、特定作業で必要になる指示・参考資料・ツールをまとめた作業パッケージです。** Copilotはタスクとの関連性から必要なSkillを読み込みます。

**入口は`SKILL.md`**です。Copilotでは、リポジトリの`.github/skills/名前/SKILL.md`などに配置します。

| 記載箇所 | 内容 |
|---|---|
| 冒頭の`name` | Skillの名前 |
| 冒頭の`description` | どんな作業で使うか |
| 本文 | 守るルール、作業手順、参照先、使うコマンド |

次のShopifyの実例では、機能の実装に必要な制約と、調査から動作確認までの手順を一つのSkillにまとめています。

<small>仕様：[GitHub公式 Agent Skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)。読み込みや手順の実行は、実際のログで確認します。</small>

<!-- 発表メモ：約1分20秒。Skillの入口と本文の構成を説明した後、Shopifyの背景、具体的なファイルの順に進む。作業固有の制約、参考資料やスクリプトをまとめられ、Instructionsと一部重複してもよい。必要なら使うSkillを依頼文に明記する。 -->

---

# 5.1 Agent Skillsの実例：Shopify

**作業背景**：ShopifyはECサイトの基盤です。Shopify Functionsでは、割引などの処理を開発者が追加できます。例えば「購入条件に応じた割引を実装して」とAIに依頼する場面です。

**必要な前提**：使えるデータ項目と型は、Shopifyの「スキーマ」で決まっています。項目を推測して実装すると、存在しないデータを使う誤りにつながります。

そこで、`shopify-functions/SKILL.md`に次のルールと手順を記載しています（原文の日本語要約）。

1. 実装前に**公式ドキュメントを検索**する
2. スキーマに存在しないフィールドを**推測で追加しない**
3. 公式コマンドで雛形を作り、**テストを含めて実装**する
4. ビルドし、入力データの例を与えて**動作を確認**する

**まねしたい点**：作業順序に加え、「何を根拠に実装するか」と「どう確認するか」をSkillに記載する。

<small>出典：[ShopifyのSkills](https://github.com/Shopify/agent-skills)、[shopify-functions/SKILL.md](https://github.com/Shopify/agent-skills/blob/main/shopify-functions/SKILL.md)</small>

<!-- 発表メモ：約1分20秒。割引の実装依頼は説明用の場面で、特定の企業内タスクを再現したものではない。スキーマは使えるデータ項目と型の定義。「割引を作って」と頼むだけではShopify固有の制約を十分に渡せないため、公式資料を調べてから実装し、確認まで行う手順をSkillに残す。公開Skillは主にClaude Codeなど向けで、Copilot専用の実例ではない。 -->

---

# 5.2 Shopifyの実例：SKILL.mdに何を書くか

**`Shopify/agent-skills/shopify-functions/SKILL.md`の日本語要約・抜粋**

```markdown
---
name: shopify-functions
description: Shopifyの割引や購入時の処理をカスタマイズする機能の実装
---

- 公式ドキュメントの関連例を、実装前に検索する。
- 利用するデータ項目はスキーマで確認し、存在しない項目を作らない。
- 新しい機能の雛形は、公式CLIで生成する。
- 実装にはテストを含め、ビルドと入力例による動作確認を行う。
- ユーザーに代わってデプロイしない。
```

**含まれているもの**：調査・実装・検証の手順と、データ項目や公開操作に関する制約。

**読み取れる点**：Skillにも「守ること」を記載します。作業の進め方と一緒に残せます。

<small>出典：[実際のSKILL.md](https://github.com/Shopify/agent-skills/blob/main/shopify-functions/SKILL.md)。descriptionと本文は要約。CLIはコマンドで操作するツール、デプロイは実装を利用環境へ反映する操作。公開SkillはCopilot専用ではありません。</small>

<!-- 発表メモ：約40秒。これは公開Skillの内容を日本語で短く要約した抜粋で、実ファイルの全文ではない。最初に調査し、仕様に存在する項目を使い、公式コマンドで雛形を作り、確認まで進める。最後のデプロイ禁止は、手順だけでなく作業の制約もSkillに含める具体例。 -->

---

# 6. 検証：実行結果で変更を確かめる

**InstructionsやSkillsで確認方法を伝えたら、実際にビルドやテストを実行し、結果を確かめます。**

「テストに成功しました」という報告だけでなく、次を確認します。

- **何を実行したか**：コマンドや検証用ツールの実行ログ
- **何が分かったか**：成功・失敗、エラーの内容
- **何が未確認か**：実行していないテストや、チェック対象外の動作

**終了コード**は、プログラムが返す数値です。多くのコマンドでは0を成功、0以外を失敗として扱います。成功しても、チェックした範囲を超える正しさは保証されません。

**次の実例**：Stripeは確認を作業の流れに組み込み、Spotifyはプロジェクトに合う検証を共通の入口から実行します。

<!-- 発表メモ：約1分。検証の具体的な仕組みは次の2社の実例で説明する。指示の記載、実行した事実、検証できた範囲を区別する。 -->

---

# 6.1 Stripeの実例：確認処理を必ず実行する

**作業背景**：決済サービスのStripeでは、社内エージェント「Minions」が、人の付き添いなしでコードを変更します。変更内容は、取り込む前に人がレビューします。

**課題**：AIに「最後にチェックして」と指示するだけでは、実行するかどうかもAIの判断に依存します。

**仕組み**：コードで定義した作業の流れに、AIが実装・修正する工程と、決まったチェックを実行する工程を組み込みます。

1. AIが依頼された変更を実装する
2. 形式チェックを、AIへの依頼を介さずプログラムで実行する
3. CIで関連テストを実行し、必要ならAIに修正を戻す

**まねしたい点**：重要な確認は、指示書に書くだけでなく、実行の仕組みに組み込む。

<small>出典：[Stripe Minions Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)。作業の流れは原文の要約。</small>

<!-- 発表メモ：約40秒。原文のBlueprintsはコードで定義したワークフロー。形式チェックの実行はLLMを呼ばずにコードで行う。AIの柔軟な実装と、決まった確認処理を組み合わせる例。原文ではローカルのチェックとCIを使い分け、CIの再試行回数にも上限を設けている。確認すれば全ての不具合を防げるという意味ではない。参考情報として、Stripeは週1,300件超のMinions作成PR（コードの変更提案）が人のレビュー後に取り込まれると報告している（2026年2月）。特定の設定ファイル単体の効果を示す件数ではない。 -->

---

# 6.2 Spotifyの実例：プロジェクトに合う検証を選ぶ

**作業背景**：Spotifyの社内エージェント「Honk」は、多数のプロジェクトのコードを変更します。プロジェクトによってビルドやテストの方法が異なります。

**課題**：AIが確認コマンドを選べない、またはテストに通っても依頼外の変更を加える場合があります。

**仕組み**：Verifier（検証処理）が、設定ファイルから実行するチェックを選びます。例えばMavenという主にJava向けのビルドツールでは、設定ファイルが`pom.xml`です。

1. `pom.xml`を見つけたら、Maven向けの検証処理を選ぶ
2. ビルドやテストを実行し、結果をAIに返す。失敗したら修正を促す
3. 別のAIが依頼文と変更内容を比較し、不要な修正やテスト無効化も確認する

**まねしたい点**：確認方法を共通の入口にまとめ、AIにコマンドの推測をさせない。

<small>出典：[Spotify Honk Part 3](https://engineering.atspotify.com/2025/12/feedback-loops-background-coding-agents-part-3)。別AIによる判定も、正しさの完全な保証ではありません。</small>

<!-- 発表メモ：約40秒。先にプロジェクトごとに確認方法が異なることを説明し、それからMavenとpom.xmlを紹介する。検証は設定ファイルを手掛かりに選ばれ、エージェントは共通の検証用ツールを呼ぶ。原文ではPRを作る前にも検証を必須実行している。別AIの比較はビルドやテストとは別の確認で、依頼の範囲を逸脱していないかを見る。 -->

---

# 7. 実例から自分のチームに取り入れること

| 実例 | 取り入れたい考え方 | 自分のチームで確認すること |
|---|---|---|
| VS Code | 対象ごとに、確認コマンドと理由をInstructionsに残す | 変更箇所に合う確認方法は何か |
| Shopify | 作業固有の制約と手順をSkillにまとめる | 調査・実装・検証のどこで間違えやすいか |
| Stripe | 重要な確認を、実行の仕組みに組み込む | AIの判断に任せず実行したいチェックは何か |
| Spotify | 確認方法を、共通の入口にまとめる | AIや人が確認コマンドを迷わず使えるか |

**各社の例は、それぞれの環境に合わせた仕組みです。** 自分のプロジェクトの確認方法や制約を基に、必要な部分から取り入れます。

<!-- 発表メモ：約1分。4社の仕組みをそのまま一つに組み合わせる説明ではない。残すべき知識、繰り返す手順、確実に実行したい確認を自分のチームで選ぶための整理。 -->

---

# 8. 導入後に確認すべき点と落とし穴

| 起きること | まず確認すること |
|---|---|
| Instructionsが効かない | ルールの保存先が `.github/copilot-instructions.md` か、対応する実行環境か |
| Skillを使ってくれない | `name`、`description`、保存場所、明示的な呼び出し |
| テストしたと言うが怪しい | **実行したコマンドとログ**を確認する |
| 毎回同じ誤実装をする | 誤りをInstructions、Skill、テストのどこに残すべきか |
| 設定ファイルが巨大になる | 共通ルール、対象固有ルール、作業手順に分ける |

**実行を必須にしたいテストはCIへ**置き、失敗した変更は取り込めない設定にします。InstructionsやSkillに記載するだけでは、実行は保証されません。

<small>参考：[対応環境の公式一覧](https://docs.github.com/en/copilot/reference/custom-instructions-support)、[VS CodeのSkill利用](https://code.visualstudio.com/docs/agent-customization/agent-skills)</small>

<!-- 発表メモ：約1分20秒。Copilot ChatにはAgentやAskなどのモードがある。ファイル操作やコマンド実行が必要ならAgentモードを使う。組織のポリシーや権限により、利用できるツールが少ない場合がある。 -->

---

# 9. 来週からチームで試すなら

### 小さく始める3ステップ

1. **よくある失敗を3つ集める**：間違った確認コマンド、存在しないデータ項目の利用、テスト実行漏れなど
2. **一つだけファイル化する**：まずInstructions。繰り返す作業ならSkillも作る
3. **検証結果を残す**：実際にテストを実行したか、人の修正がどこで必要だったか

**今日の結論**

> Copilotに毎回同じ注意をするなら、その注意はチャットではなく**リポジトリに残す**。  
> 「検証して」と頼むなら、**確認に使うコマンドやツールを決める**。

**企業事例と対応機能の詳細は、下記の出典を参照してください。**

- [Microsoft VS CodeのInstructions](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md)
- [ShopifyのAgent Skills](https://github.com/Shopify/agent-skills)
- [StripeのBlueprints](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)
- [SpotifyのVerifier](https://engineering.atspotify.com/2025/12/feedback-loops-background-coding-agents-part-3)

<!-- 発表メモ：約1分。次回、失敗を見つけたときに「今回は言い直す」か「ルールやテストに反映する」かを意識してもらう。説明はここで終了し、残りは質疑に使う。 -->

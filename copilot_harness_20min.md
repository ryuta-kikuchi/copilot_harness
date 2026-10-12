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

# 1. ルールや作業手順を共有しない場合の課題

Copilotに「このコードに制約を追加して」と依頼したとき、次のような問題が起こることがあります。

- 既存の設計ルールに合わない実装をする
- 過去に直したものと同じ種類のバグを、別の箇所で作る
- 動きそうなコードを提示するが、**テストは実行していない**
- 「このファイルを見て」「そのコマンドは使わないで」と毎回伝える必要がある

**プロジェクト固有の知識や確認手順が、指示として共有されていないことが一因です。**

<!-- 発表メモ：約1分20秒。チームが暗黙に知っている設計ルールや確認方法を、Copilotが常に推測できるとは限らない。毎回チャットで補足している注意点を、再利用できる指示や手順として残す必要がある。次のスライドで、そのための周辺環境をハーネスとして説明する。 -->

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

**VS Codeの公開実例で、Instructions・Skills・変更後の検証を説明します。**

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

次のVS Codeの実例では、まず**全体に適用する共通Instructions**を示し、その後に**ビルドスクリプト向けの対象別Instructions**を見ます。

**自分のチームでは**：毎回言い直している具体的な注意点から、短く書き始めます。

<small>仕様：[GitHub公式 リポジトリInstructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions)</small>

<!-- 発表メモ：約1分20秒。まず指示書の役割と共通版・対象別版を説明する。具体例は次からのVS Codeの公開ファイルに一本化する。一般論よりも、そのプロジェクトで間違えやすいことと確認方法を残す。規約が多い場合はdocsへの参照や対象別Instructionsを利用する。 -->

---

# 4.1 共通Instructionsの実例：VS Code

**背景**：VS Codeでは、複数の機能を開発しても設計や書き方をそろえる必要があります。共通の前提を、`.github/copilot-instructions.md`に記載しています。

**ファイルに含まれる内容**：フォルダーの役割、設計方針、コードの規約、テスト・ビルドの選び方。

```markdown
# VS Code Copilot Instructions
- インデントにはスペースではなくタブを使う。
- 関数名や変数名はcamelCase（例：loadFile）にする。
- 新しく実装する前に、既存の共通関数や実装パターンを探す。
- テストを追加するときは、既存の書き方を確認する。
- 変更の範囲とリスクに合わせて、必要な検証を選ぶ。
```

**「共通」は、そのリポジトリ全体に適用するという意味です。** チームの規約と、毎回伝えたい作業上の注意を残します。

<small>出典：[VS Codeの共通Instructions](https://github.com/microsoft/vscode/blob/main/.github/copilot-instructions.md)。本文は原文の日本語要約・抜粋。規約はVS Codeプロジェクトのものです。</small>

<!-- 発表メモ：約50秒。共通Instructionsの具体例として公開ファイルの内容を紹介する。インデントや命名の規約、既存コードとテストの再利用、変更に応じた検証の方針はリポジトリ全体で参照したい前提。原文にはフォルダーの役割や設計の説明も含まれる。camelCaseは先頭を小文字にし、単語の区切りを大文字で表す命名方式。ここでの共通は全世界のプロジェクトに共通の正解という意味ではなく、VS Code内での共通方針。次は特定の作業にだけ必要な確認コマンドを対象別ファイルに置く例。 -->

---

# 4.2 対象別Instructionsの実例：ビルド処理の確認

**開発環境**：VS Code自体の開発には、複数のビルド処理をまとめて起動する「VS Code - Build」というタスクがあります。本体のソース変換・型チェックに加え、拡張機能などもビルドします。

**今回の作業**：本体の機能ではなく、ソースをJavaScriptへ変換するビルドスクリプトを変更します。Makefileに相当する役割を、TypeScriptで実装したものです。

**必要な確認**：変更した変換処理を直接動かすこと。この確認には、拡張機能などを含むビルドタスクの実行は不要です。

**Instructionsを書く理由**：「変更を確認して」だけでは、どの処理を動かすべきかをエージェントが判断する必要があります。そこで、不要なタスクと使うべきコマンドを明記します。

<small>出典：[VS Code - Buildのタスク定義](https://github.com/microsoft/vscode/blob/main/.vscode/tasks.json)、[変更の検証に関するInstructions](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md#important-validating-changes)</small>

<!-- 発表メモ：約35秒。初めにVS Code - Buildが何を指すか説明する。これはVS Code本体の変換と型チェック、拡張機能、Copilotのビルドをまとめて起動する開発用タスクで、配布物作成までの全工程という意味ではない。今回の変更対象はビルドスクリプト。原文では、これらのスクリプトはNode.jsで直接実行でき、本体のビルドでコンパイルされないと説明している。確認に必要な処理を具体的に伝え、エージェントが確認方法を推測する負担を減らすのが指示の意義。実際にエージェントが不要なタスクを実行したという経緯や所要時間は原文に記載がないため断定しない。 -->

---

# 4.2.1 対象別Instructionsのファイル内容

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

# 4.2.2 指定した確認コマンドの動き

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

次のVS Codeの実例では、画面の配色を実装・修正するときに必要なルールと具体的な実装方法を、一つのSkillにまとめています。

<small>仕様：[GitHub公式 Agent Skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)。読み込みや手順の実行は、実際のログで確認します。</small>

<!-- 発表メモ：約1分20秒。Skillの入口と本文の構成を説明した後、VS Codeの配色作業の背景、具体的なファイルの順に進む。作業固有の制約や実装方法をまとめられ、Instructionsと一部重複してもよい。必要なら使うSkillを依頼文に明記する。 -->

---

# 5.1 VS CodeのSkill実例：配色とテーマ対応

**背景**：VS Codeでは、利用者が画面を明るい表示や暗い表示に切り替えられます。この配色設定を「テーマ」と呼びます。

**起こる問題**：新しいボタンやパネルの背景を白で固定すると、暗いテーマに切り替えても、その部分だけ白いままになります。

**VS Codeの仕組み**：部品ごとの色をテーマ側で定義しています。画面を作るコードがその色を使えば、選ばれたテーマに合わせて配色が変わります。

**Skillの役割**：`ux-theming`には、この仕組みの使い方と、見やすい表示を保つための注意点が書かれています。**画面の配色を実装・修正する作業で参照します。**

**依頼例**：「このパネルの背景を、明るい表示・暗い表示のどちらにも対応するように修正して」

<small>出典：[ux-theming/SKILL.md](https://github.com/microsoft/vscode/blob/main/.github/skills/ux-theming/SKILL.md)。白い背景と依頼文は、用途を説明するための例です。</small>

<!-- 発表メモ：約1分20秒。背景、起こる問題、VS Codeの仕組み、Skillの役割、依頼例の順に説明する。テーマは利用者が選ぶ配色設定。固定の色を直接書くと切り替わらないため、テーマが部品ごとに用意する色を使う。全ての部品に同じ背景色を使う意味ではない。実ファイルはこの使い方に加え、キーボード操作の対象の表示や高コントラストへの対応も説明する。次のスライドで具体的なSkillの記載を示す。 -->

---

# 5.2 VS Codeの実例：SKILL.mdに何を書くか

**`.github/skills/ux-theming/SKILL.md`の日本語要約・抜粋**

```markdown
---
name: ux-theming
description: VS Codeの配色・テーマ対応・フォーカス表示を実装するときに使う
---

- 画面部品には、テーマが提供する色や既存の標準スタイルを使う。
- 新しい色を登録するときは、明暗と高コントラストの4種類に対応する。
- キーボード操作の対象を示す枠にも、テーマの色を使う。
- 高コントラストでは、透明な背景を避け、境界を分かるようにする。
- 色の登録場所や標準スタイルの実装例を参照する。
```

**含まれているもの**：色の登録・適用方法、キーボード操作時の表示、高コントラストへの対応、参照すべき実装ファイル。

**読み取れる点**：Skillは作業手順だけでなく、その作業で守るルールや実装知識もまとめられます。

<small>出典：[実際のSKILL.md](https://github.com/microsoft/vscode/blob/main/.github/skills/ux-theming/SKILL.md)。descriptionと本文は要約。フォーカス表示は、キーボード操作の対象を示す表示です。</small>

<!-- 発表メモ：約40秒。これは公開Skillの内容を日本語で短く要約した抜粋で、実ファイルの全文ではない。4種類はlight、dark、hcDark、hcLight。実ファイルには色の登録コード、CSSでの参照例、部品の標準スタイル、フォーカス表示の実装例、参照ファイル一覧がある。テストを必ず実行するためのSkillというより、配色作業で必要なルールと実装方法をまとめたSkill。InstructionsとSkillsの内容が重なり得ることも説明する。 -->

---

# 6. Copilotに変更後の確認まで依頼する

**作業背景**：前の例で、VS Codeのパネルをテーマに対応させる修正を依頼しました。修正したコードが、意図した動作をするか確認する必要があります。

**Copilotのエージェントモードへの依頼例**

> このパネルの配色を、`ux-theming` Skillを参考に修正してください。共通Instructionsに従って必要な確認を実行し、実行内容・結果・未確認事項を報告してください。

**役割のつながり**

- **Instructions**：共通の規約と、確認方法を選ぶ方針を伝える
- **Skill**：配色の実装に必要なルールと方法を伝える
- **検証**：変更に合うチェックを実行し、結果を確かめる

<small>依頼文は公開ファイルを基にした説明例です。実演や、実際の修正・検証結果の提示は行いません。</small>

<!-- 発表メモ：約1分。VS Code自身のコードをCopilotで修正する場面を説明する。ここまでの配色例を続け、コードの修正と確認を一つの依頼で伝える。Skillの読み込みや確認の実行は保証されるわけではなく、実際の参照やログを確認する。組織の設定や権限によってコマンド実行に承認が必要な場合がある。 -->

---

# 6.1 VS Codeの実例：変更に合う確認方法を選ぶ

**共通Instructionsの検証方針**：大規模なビルドや型チェックには時間・資源がかかるため、変更範囲とリスクに応じて必要な確認を選びます。

**公開ファイルの指示（日本語要約）**

1. すでに実行中のビルドなどがあれば、まずその結果を確認する
2. 変更した動作を確認できる、対象を絞ったテストを優先する
3. 広い範囲の変更や型・ビルド設定の変更では、必要に応じて型チェックやビルドを行う

**具体的な確認コマンドの例（Windows）**

```console
scripts\test.bat --grep "<対象テスト名>"
```

`test.bat`は既存のテストを実行するスクリプト、`--grep`は対象をテスト名で絞る指定です。対象名は変更内容に合わせて選びます。

<small>出典：[共通Instructionsの検証方針](https://github.com/microsoft/vscode/blob/main/.github/copilot-instructions.md#validating-typescript-changes)、[テスト実行スクリプト](https://github.com/microsoft/vscode/blob/main/scripts/test.bat)。コマンドの対象名は説明用の仮置きです。</small>

<!-- 発表メモ：約40秒。4章は指示書に何を書くか、ここはその指示に従って確認方法を選ぶ話。型チェックは型の整合性を調べる処理で、画面の見やすさの確認ではない。テスト名を絞る指定は共通Instructionsに記載がある。何でもこのコマンドだけで十分とは説明しない。対象テストが存在するか、変更を確認できるかを確かめる。必要な依存関係や実行用の出力が未準備なら準備も必要になる。実演は行わない。 -->

---

# 6.2 配色の例：何を確認できたかを分けて報告する

**背景**：テストが通っても、明るい表示・暗い表示で文字や枠が見やすいかを確認できたとは限りません。

**Copilotの報告で確認すること**

| 項目 | 確認する内容 |
|---|---|
| 実行内容 | どのタスクやコマンドを実行し、何を対象にしたか |
| 実行結果 | ログにエラーがないか、対象テストが実際に実行されたか |
| 画面の確認 | テーマを切り替えた表示を確認したか。未確認ならその旨を報告する |

**人が確認する場面もあります。** 表示確認が実行できない場合は、利用者がVS Codeを起動してテーマを切り替え、文字や境界が見えるかを確認します。

**終了コード**はプログラムが返す数値で、多くのコマンドでは0が成功です。ただし、成功したチェックの範囲と、まだ確認していないことを区別します。

<small>参照：[配色・高コントラスト対応のSkill](https://github.com/microsoft/vscode/blob/main/.github/skills/ux-theming/SKILL.md)。表示確認と報告の整理は、この実例を基にした運用上の提案です。</small>

<!-- 発表メモ：約40秒。実行ログで事実を確認する。0件のテストが実行されただけでも成功表示になるツールがあるので対象も見る。画面を確認していない場合は「テスト成功・表示は未確認」と分けて報告する。実際の画面では明暗に加え、Skillが扱う高コントラストで文字や枠が見えるかも確認する。実行や表示確認を実施したという結果はこの資料では示していない。 -->

---

# 7. VS Codeの実例から自分のチームに取り入れること

| 要素 | VS Codeの実例 | 自分のチームで決めること |
|---|---|---|
| 共通Instructions | リポジトリ全体の規約と検証方針 | 毎回伝えている前提や注意点 |
| 対象別Instructions | ビルドスクリプト向けの確認コマンド | 特定のファイルで必要になる指示 |
| Skill | 配色作業のルール・実装方法・参照先 | 作業ごとに参照したい知識 |
| 検証 | 変更範囲に合うチェックと結果の確認 | 実行するコマンドと、人が確認する項目 |

**自分のプロジェクトにあるルールや確認方法から始めます。** Copilotに渡す指示と、実際の検証結果をセットで残します。

<!-- 発表メモ：約1分。共通Instructions、対象別Instructions、Skill、検証の役割を同じリポジトリの実例で整理する。VS Code固有のコマンドや配色ルールをそのまま他のプロジェクトに持ち込むのではなく、自分のチームに相当する前提や確認方法を探す。 -->

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

1. **よくある失敗を3つ集める**：間違った確認コマンド、テーマ変更で見えない配色、テスト実行漏れなど
2. **一つだけファイル化する**：まずInstructions。繰り返す作業ならSkillも作る
3. **検証結果を残す**：実際にテストを実行したか、人の修正がどこで必要だったか

**今日の結論**

> Copilotに毎回同じ注意をするなら、その注意はチャットではなく**リポジトリに残す**。  
> 「検証して」と頼むなら、**確認に使うコマンドやツールを決める**。

**VS Codeの公開ファイルと対応機能の詳細は、下記の出典を参照してください。**

- [Microsoft VS CodeのInstructions](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md)
- [Microsoft VS Codeの共通Instructions](https://github.com/microsoft/vscode/blob/main/.github/copilot-instructions.md)
- [Microsoft VS Codeの配色Skill](https://github.com/microsoft/vscode/blob/main/.github/skills/ux-theming/SKILL.md)
- [VS CodeのAgent Skillsの使い方](https://code.visualstudio.com/docs/agent-customization/agent-skills)

<!-- 発表メモ：約1分。次回、失敗を見つけたときに「今回は言い直す」か「ルールやテストに反映する」かを意識してもらう。説明はここで終了し、残りは質疑に使う。 -->

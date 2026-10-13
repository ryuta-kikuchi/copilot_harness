# 1. Copilotへの指示をチャットだけで伝える場合の課題

Copilotにコードの修正を依頼すると、次のような問題が起こることがあります。

- 既存の設計ルールに合わない実装をする
- 「このファイルを見て」「そのコマンドは使わないで」と、チャットごとに伝え直す必要がある
- 以前説明して解消したはずの誤解や認識のずれが、別の依頼でも繰り返される
- コードを修正しても、テストを実行せずに作業を終える

**一因は、プロジェクト固有のルールや確認手順が、チャットをまたいで参照できる形で残されていないことです。**

<!-- 発表メモ：約1分20秒。チームが暗黙に知っているルールを毎回説明する負担に加え、一度説明して解消した認識のずれが別の依頼でも再発する点を取り上げる。チャットでの補足を再利用できる指示や手順に残すと、以後の作業でも参照できる。次の章で、モデルとハーネスの役割分担、その仕組みをプロジェクトに合わせて整える考え方を説明する。 -->

# 2. ハーネスとは、LLMをエージェントとして動かす基盤

**ハーネスは、LLMに必要な情報やツールを提供し、LLMが要求した操作の実行と作業状態の管理を担うソフトウェア基盤です。**

LLM（大規模言語モデル）は、与えられた情報を基に推論し、回答文やツール呼び出しの要求を生成します。ハーネスがその要求に応じてツールを実行し、結果をLLMに返すことで、コードを調べ、修正し、結果を確認するエージェントとして作業できます。

| ハーネスの役割 | 具体的に行うこと |
|---|---|
| コンテキスト管理 | 指示、ソースコード、会話履歴などをLLMに渡す |
| ツール管理 | ファイル操作、ターミナル、外部サービス連携などの操作手段を提供する |
| 実行制御 | LLMの要求した操作を実行し、結果を返すやり取りを繰り返す |
| 状態管理 | 会話履歴、作業状況、ファイル変更などを追跡する |
| 権限管理 | 操作を実行してよいか確認し、必要なら利用者の承認を求める |

例えば、LLMが「テストを実行する」と要求すると、ハーネスが権限を確認してコマンドを実行し、結果を返します。LLMはその結果を見て、修正を続けるか、作業を終えるか判断します。

**LLMが何をするか判断し、ハーネスがその判断を実際の作業につなげる、という役割分担です。**

出典：[VS Code公式 ハーネスの役割と作業の流れ](https://code.visualstudio.com/docs/agents/concepts/agent-harnesses)。テスト実行の流れは、公式の説明を具体化した例です。

<!-- 発表メモ：約1分。まずLLMは回答文や操作の要求を生成し、ハーネスは情報やツールを提供して、その操作を実行・管理する役割だと説明する。この組み合わせで、調査・修正・確認を進めるエージェントになる。コンテキストはモデルに渡す情報で、ツールは実際の操作手段。テスト実行の例では、LLMがコマンド実行を要求し、ハーネスが権限を確認して結果を返し、LLMが次の行動を判断する。ハーネスはこのやり取りと作業状態を管理するソフトウェアで、OSやハードウェアなど周囲にあるもの全てを指すわけではない。 -->

# 2.1 InstructionsやSkillsとハーネスの関係

**InstructionsやSkillsは、ハーネスをプロジェクトに合わせてカスタマイズするための構成要素です。**

Copilotには、モデルとツールを連携させるハーネスの仕組みがあらかじめ用意されています。利用者は、そこにプロジェクトのルールや作業手順、追加の確認処理を設定できます。

この勉強会では、まず次の3機能を扱います。

- **Instructions**：プロジェクトの前提やルールを伝える
- **Skills**：特定作業のルール・手順・参考資料をまとめる
- **Hooks**：決めたイベントで、処理を自動実行する

InstructionsとSkillsは、どちらもルールと手順を含められます。主な違いは、いつ読み込まれるかです。Hooksは、処理の起動をLLMの判断に委ねずに組み込めます。

MCPで外部ツールを追加したり、Custom Agentsで役割ごとの指示やツールを設定したりする方法もあります。Prompt Filesでは、レビュー依頼などの定型文を保存して呼び出せます。

出典：[VS Code公式 カスタマイズの考え方](https://code.visualstudio.com/docs/agents/concepts/customization)

<!-- 発表メモ：約35秒。Copilotには利用者が設定を追加する前からハーネスがある。提供されているハーネスの仕組みと、利用者が追加する構成要素を区別する。InstructionsやSkillsはモデルに渡す規約・作業知識を用意し、Hooksは決めたイベントで処理を起動する。この勉強会では3機能を主に扱い、MCP・Custom Agents・Prompt Filesは、関連機能として用途だけをここで紹介する。Hooksの起動も、処理の成功やコードの正しさを保証するものではない。 -->

# 2.2 ハーネスエンジニアリングとは、作業の仕組みを設計・改善すること

**ハーネスエンジニアリングは、LLMエージェントが適切に作業できるように、指示・ツール・実行環境・検証方法などを設計し、改善する取り組みです。**

例えば、繰り返し起きる誤実装をInstructionsに注意点として残す、作業固有の知識をSkillにまとめる、終了前にLintを実行するHookを用意する、といった改善が含まれます。エージェントが参照できる仕様書や、実行しやすいテスト環境を整えることも、この取り組みの一部です。

OpenAIの記事では、エージェントが参照できるように開発知識をリポジトリに整理し、Lintやテストで問題を検出して修正につなげる環境を整備した実例が紹介されています。

**利用者も、既存のハーネスの設定や作業環境を改善することで実践できます。** この勉強会では、その入口としてInstructions・Skills・Hooksの整備を扱います。

参考：[OpenAI Harness engineering](https://openai.com/index/harness-engineering/)。上の定義は、記事の取り組みをこの勉強会向けに整理したものです。

<!-- 発表メモ：約40秒。ハーネスはエージェントを動かす基盤、ハーネスエンジニアリングはその基盤や周辺の作業環境を設計・改善する取り組み。Copilotの場合、ハーネス本体の実装は提供元が担い、利用者は規約、作業知識、ツール連携、検証方法などを整備できる。InstructionsやSkillsの整備はその一例で、全体ではない。OpenAIの記事はCodexでの開発経験であり、Copilotで同じ成果が出ると保証するものではない。ここでは、知識を参照できる形で残し、問題を検出して修正できる仕組みを作るという考え方を参考にする。次の章では、実際に何をどこに設定するかを整理する。 -->

# 3. 全体像：何をどこに設定するか

| 機能 | 設定するもの | リポジトリ内の場所 |
|---|---|---|
| **Instructions** | 共通の規約、対象ごとの注意点、検証方針 | 共通：`.github/copilot-instructions.md`<br>対象別：`.github/instructions/*.instructions.md` |
| **Agent Skills** | 特定作業のルール、手順、参考資料 | `.github/skills/名前/SKILL.md`など |
| **Hooks** | イベントと、その時に実行するコマンド | `.github/hooks/*.json`など<br>実行するスクリプトも用意する |

**検証方法を伝えるならInstructionsやSkills、決まったタイミングで処理を実行するならHooksです。**

InstructionsとSkillsはVS Codeの公開リポジトリ、HooksはVS Code公式ドキュメントの設定例で説明します。

出典：[カスタマイズの全体像](https://code.visualstudio.com/docs/agents/concepts/customization)、[Hooksの設定](https://code.visualstudio.com/docs/agent-customization/hooks)。対応機能や設定形式は実行環境で異なります。

<!-- 発表メモ：約1分20秒。検証を独立した第三の設定と呼ばず、InstructionsやSkillsに検証方針を残し、必要ならHooksで確認処理を起動する関係を説明する。HooksはVS Codeの公開リポジトリで実運用されている例ではなく、公式ドキュメントの例を扱う。 -->

# 4. Instructions：プロジェクトの前提を継続的に伝える

**Instructionsは、Copilotが作業時に参照するMarkdownの指示書です。** 人が毎回伝えている前提や注意点を、リポジトリに残します。

| 種類 | 保存先 | 適用する場面 |
|---|---|---|
| 共通Instructions | `.github/copilot-instructions.md` | リポジトリ全体の作業 |
| 対象別Instructions | `.github/instructions/*.instructions.md` | 指定したファイルの作業 |

**記載する内容**：設計上の約束、避けたい失敗、確認に使うコマンドとその理由。

次のVS Codeの実例では、まず**全体に適用する共通Instructions**を示し、その後に**ビルドスクリプト向けの対象別Instructions**を見ます。

**自分のチームでは**：毎回言い直している具体的な注意点から、短く書き始めます。

仕様：[GitHub公式 リポジトリInstructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions)

<!-- 発表メモ：約1分20秒。まず指示書の役割と共通版・対象別版を説明する。具体例は次からのVS Codeの公開ファイルに一本化する。一般論よりも、そのプロジェクトで間違えやすいことと確認方法を残す。規約が多い場合はdocsへの参照や対象別Instructionsを利用する。 -->

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
- 実行中のビルドがあれば、まず既存の結果を確認する。
- 変更に関連する既存のテストを選び、実行する。
```

**「共通」は、そのリポジトリ全体に適用するという意味です。** コードの規約と、変更後の確認方法を同じ指示書に残せます。

出典：[VS Codeの共通Instructions](https://github.com/microsoft/vscode/blob/main/.github/copilot-instructions.md)。本文は原文の日本語要約・抜粋。

<!-- 発表メモ：約1分20秒。共通Instructionsにはコードの規約と検証方針の両方を書く。検証は独立した設定機能ではなく、この指示書に残す内容の一つ。原文にはフォルダーの役割や設計の説明も含まれる。camelCaseは先頭を小文字にし、単語の区切りを大文字で表す命名方式。テストはコードの動作を確認するプログラムで、変更した機能に関連するものを選んで実行する。広い範囲や型・ビルド設定の変更では、必要に応じて型チェックやビルドも行う。共通はVS Code内の共通方針で、次は対象別の確認方法へ進む。実演は行わない。 -->

# 4.2 対象別Instructionsの実例：ビルド処理の確認

**開発環境**：VS Code自体の開発には、複数のビルド処理をまとめて起動する「VS Code - Build」というタスクがあります。本体のソース変換・型チェックに加え、拡張機能などもビルドします。

**今回の作業**：本体の機能ではなく、ソースをJavaScriptへ変換するビルドスクリプトを変更します。Makefileに相当する役割を、TypeScriptで実装したものです。

**必要な確認**：変更した変換処理を直接動かすこと。この確認には、拡張機能などを含むビルドタスクの実行は不要です。

**Instructionsを書く理由**：「変更を確認して」だけでは、どの処理を動かすべきかをエージェントが判断する必要があります。そこで、不要なタスクと使うべきコマンドを明記します。

出典：[VS Code - Buildのタスク定義](https://github.com/microsoft/vscode/blob/main/.vscode/tasks.json)、[変更の検証に関するInstructions](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md#important-validating-changes)

<!-- 発表メモ：約35秒。初めにVS Code - Buildが何を指すか説明する。これはVS Code本体の変換と型チェック、拡張機能、Copilotのビルドをまとめて起動する開発用タスクで、配布物作成までの全工程という意味ではない。今回の変更対象はビルドスクリプト。原文では、これらのスクリプトはNode.jsで直接実行でき、本体のビルドでコンパイルされないと説明している。確認に必要な処理を具体的に伝え、エージェントが確認方法を推測する負担を減らすのが指示の意義。実際にエージェントが不要なタスクを実行したという経緯や所要時間は原文に記載がないため断定しない。 -->

# 4.2.1 対象別Instructionsのファイル内容

**前のスライドの確認方法を、エージェントが読むMarkdownの指示書にします。** 対象ファイルと、実行してほしいコマンドを記載します。

`.github/instructions/buildNext.instructions.md`（原文の日本語要約・抜粋）

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

出典：[buildNext.instructions.md — Validating Changes](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md#important-validating-changes)。実ファイルにはbundleの検証コマンドも記載。

<!-- 発表メモ：約30秒。背景を説明した後に指示ファイルの具体例を読む。原文の一部を日本語で要約し、descriptionやほかの説明は省略している。applyToが適用範囲、その下がCopilotに渡す指示本文。タスク名が前のスライドで説明したものだと結び付ける。次のスライドでコマンドの意味と確認範囲を説明する。実演は行わない。 -->

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

出典：[コマンドの選択処理](https://github.com/microsoft/vscode/blob/main/build/next/index.ts)、[対象ファイルの列挙と変換](https://github.com/microsoft/vscode/blob/main/build/next/index.ts#L398-L416)

<!-- 発表メモ：約50秒。TypeScriptは型などの記述を加えたJavaScriptで、ここでは実行用のJavaScriptに変換する。index.tsは変換対象の入口ではなく、処理を起動して選ぶためのスクリプト。名前に特別な自動変換機能があるわけではない。実装がソースフォルダーを走査して対象ファイルを列挙する。依存ファイルを芋づる式にたどる説明はしない。make transpileのように必要な工程を選ぶ、と説明する。全タスクが必ず全工程を実行するわけではない。この変換コマンドだけで結合処理やアプリの動作まで確認したことにはならない。実演は行わない。 -->

# 5. Agent Skills：特定作業のルールと手順をまとめる

**Agent Skillは、特定作業で必要になる指示・参考資料・ツールをまとめた作業パッケージです。** Copilotはタスクとの関連性から必要なSkillを読み込みます。

Copilotでは、リポジトリの`.github/skills/名前/SKILL.md`などに配置します。

通常は依頼内容と`name`・`description`から関連するSkillを選び、本文を読み込みます。Skill名を毎回指定する必要はありません。必要なら`/Skill名`で明示的に呼び出せます。

次のVS Codeの実例では、画面の配色を実装・修正するときに必要なルールと具体的な実装方法を、一つのSkillにまとめています。

仕様：[GitHub公式 Agent Skills](https://code.visualstudio.com/docs/agent-customization/agent-skills)。読み込みや手順の実行は、実際のログで確認します。

<!-- 発表メモ：約1分20秒。Skillの入口と本文の構成を説明した後、VS Codeの配色作業の背景、具体的なファイルの順に進む。作業固有の制約や実装方法をまとめられ、Instructionsと一部重複してもよい。必要なら使うSkillを依頼文に明記する。 -->

# 5.1 VS CodeのSkill実例：配色とテーマ対応

**背景**：VS Codeでは、利用者が画面を明るい表示や暗い表示に切り替えられます。この配色設定を「テーマ」と呼びます。

**起こる問題**：新しいボタンやパネルの背景を白で固定すると、暗いテーマに切り替えても、その部分だけ白いままになります。

**VS Codeの仕組み**：部品ごとの色をテーマ側で定義しています。画面を作るコードがその色を使えば、選ばれたテーマに合わせて配色が変わります。

**Skillの役割**：`ux-theming`には、この仕組みの使い方と、見やすい表示を保つための注意点が書かれています。**画面の配色を実装・修正する作業で参照します。**

**依頼例**：「このパネルの背景を、明るい表示・暗い表示のどちらにも対応するように修正して」

出典：[ux-theming/SKILL.md](https://github.com/microsoft/vscode/blob/main/.github/skills/ux-theming/SKILL.md)。白い背景と依頼文は、用途を説明するための例です。

<!-- 発表メモ：約1分20秒。背景、起こる問題、VS Codeの仕組み、Skillの役割、依頼例の順に説明する。テーマは利用者が選ぶ配色設定。固定の色を直接書くと切り替わらないため、テーマが部品ごとに用意する色を使う。全ての部品に同じ背景色を使う意味ではない。実ファイルはこの使い方に加え、キーボード操作の対象の表示や高コントラストへの対応も説明する。次のスライドで具体的なSkillの記載を示す。 -->

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

出典：[実際のSKILL.md](https://github.com/microsoft/vscode/blob/main/.github/skills/ux-theming/SKILL.md)。descriptionと本文は要約。フォーカス表示は、キーボード操作の対象を示す表示です。

<!-- 発表メモ：約40秒。これは公開Skillの内容を日本語で短く要約した抜粋で、実ファイルの全文ではない。4種類はlight、dark、hcDark、hcLight。実ファイルには色の登録コード、CSSでの参照例、部品の標準スタイル、フォーカス表示の実装例、参照ファイル一覧がある。テストを必ず実行するためのSkillというより、配色作業で必要なルールと実装方法をまとめたSkill。InstructionsとSkillsの内容が重なり得ることも説明する。 -->

# 6. Hooks：イベントをきっかけに処理を起動する

**Hookは、エージェントの作業中に決められた出来事が起きると、用意したプログラムを動かす仕組みです。** この出来事を「イベント」と呼びます。

- **イベント**：ツールの実行前・正常終了後、作業を終える直前などの節目
- **Hookの設定**：どのイベントで、どのコマンドを起動するか
- **実行するプログラム**：確認や整形を行い、必要なら結果をVS Codeへ伝える

> 例：Copilotが作業を終える前に、コードのチェックを自動実行する。

**イベントを検知してプログラムを起動するのは、VS Code側の実行環境です。** チェックの起動を、AIモデルの判断に委ねずに組み込めます。

この章はVS Code拡張機能側で動くLocalを例に説明します。[Hooksの公式説明](https://code.visualstudio.com/docs/agent-customization/hooks)。実行先によって設定・入出力の仕様が異なります。

<!-- 発表メモ：約35秒。作業の最後にコードのチェックを自動実行する用途を示す。イベントは実行環境が検知する節目、設定はタイミングとコマンド、プログラムは実際の処理を担う。InstructionsやSkillsに確認方法を残すことに加えて、チェックの起動をHookとして設定できる。実行環境が起動することと、チェックの合格は区別する。ここで説明するのはLocalの仕様で、別の実行先にはそのまま適用しない。 -->

# 6.1 Hookが働くイベントの代表例

**作業全体の節目と、一つ一つの操作の節目があります。**

| イベント | いつ起きるか | Hookの用途例 |
|---|---|---|
| `SessionStart` | 最初の依頼でセッションが始まる時 | 現在のプロジェクト情報を初期情報として渡す |
| `PreToolUse` | 各ツールを呼び出す直前 | 操作の内容を確認し、対象の操作を禁止する |
| `PostToolUse` | ツール呼び出しが正常に終わった後 | 編集したファイルの書式を整える |
| `Stop` | エージェントが今回の処理を終える直前 | コードをチェックし、失敗なら修正を要求する |

**ツールは、ファイルの編集やコマンド実行などに使う操作機能です。** 今回は、作業終了前の`Stop`を使います。

VS Code Localの代表例。Stopはチャット全体の終了とは限りません。[イベントの公式一覧](https://code.visualstudio.com/docs/agent-customization/hooks#local-hook-lifecycle-events)

<!-- 発表メモ：約40秒。SessionStartでは、その時点のプロジェクト情報などを追加できる。固定の規約はInstructionsに書く。PostToolUseはツール呼び出しが正常に終わった後に起きるが、ターミナルで実行したテストの合格を意味するとは限らない。整形例では編集ツールと対象ファイルを判定する処理が必要。Stopは今回のエージェント実行を終える前で、チャットを閉じるタイミングではない。ツールの詳細な一覧は今回の例には不要なので省き、Stopの具体例へ進む。 -->

# 6.2 実用例：作業終了前にLintを自動実行する

**困りごと**：コードの修正後に、チェックを実行しないまま「完了」と報告することがある。毎回「最後に確認して」と伝える手間を減らしたい。

**Lint**は、コードを実行せずに、書き方の問題や規約違反などを検出するチェックです。ここでは、既存の`npm run lint`で短時間のチェックができるプロジェクトを想定します。

1. **Copilot**が修正を終え、作業を終了しようとする。
2. **VS Code**が`Stop`のHookを起動する。
3. **Hookのプログラム**が`npm run lint`を実行する。
4. **成功なら終了。初回の失敗なら、エラー内容をCopilotへ戻して修正を要求する。**
5. **修正後も再チェックし、失敗なら理由を利用者に示して自動修正を打ち切る。**

**自分のチームでは、既存の軽いチェックを終了前に組み込むところから試せます。**

[Stopの公式仕様](https://code.visualstudio.com/docs/agents/reference/hooks-reference#stop)を使った運用例です。Lintコマンドはプロジェクトに合わせて用意します。

<!-- 発表メモ：約45秒。毎回の確認依頼を減らし、修正直後に問題を見つける利点から説明する。npm run lintはpackage.jsonに定義済みのlintスクリプトを実行するコマンドで、Copilot固有の機能ではない。Hookのプログラムが直接コマンドを起動するので、モデルがチェック用のツールを選ばなくても実行できる。全てのプロジェクトにlintコマンドがあるわけではない。既存の静的チェックや型チェックなど、そのプロジェクトに合った短時間の確認に置き換える。CIは共有コードに取り込む条件、今回のHookは作業中に修正を促すための確認で、併用できる。再開後の終了前にもチェックを実行し、成功かどうかを実行結果で確認する。修正後も失敗する場合は、エラー内容を利用者に示して自動修正を停止する。 -->

# 6.3 Hookの設定：いつ・何を起動するか

**`.github/hooks/`に、設定ファイル`lint.json`と処理本体`check_lint.py`を用意します。**

以下は`lint.json`の内容です。JSONは、項目名と値で情報を表す形式です。

```json
{
  "hooks": {
    "Stop": [{
      "type": "command",
      "command": "python -X utf8 .github/hooks/check_lint.py",
      "cwd": ".",
      "timeout": 15
    }]
  }
}
```

**`Stop`が起動するタイミング、`command`が起動するコマンドです。** `cwd`は実行する場所で、ここではリポジトリのルート。`timeout`は待つ時間の上限（秒）です。

Python 3が必要です。`-X utf8`はUTF-8で入出力する指定。Localで`chat.useHooks`が有効か、ワークスペースが信頼されているかを確認します。[設定の公式仕様](https://code.visualstudio.com/docs/agents/reference/hooks-reference#configuration-format)

<!-- 発表メモ：約35秒。まず利用者が用意する設定と処理本体を示す。このJSONはVS Codeが読む設定ファイルで、いつ何を起動するかを指定する。この後、VS Codeからプログラムへ渡す入力と、プログラムから返す結果を順に説明する。どちらもJSONだが、項目名や構造はそれぞれの仕様で決まっている。pythonはPythonの実行環境を起動するコマンドで、check_lint.pyは利用者が用意するプログラム。pythonがPython 3を起動する環境を想定する。python3やpyを使う環境ではコマンドを合わせる。-X utf8で日本語を含む入出力をUTF-8で扱う。cwdをリポジトリのルートに指定し、そこで既存のnpm run lintを実行する。HookのプログラムにはPythonを使い、チェックにはプロジェクトの既存コマンドを使う。Hookの上限15秒に対し、6.6・6.7のプログラムはチェックを10秒で打ち切る。所要時間に合わせて両方を調整する。この資料ではコード例を示すだけで、有効なHookをリポジトリに追加しない。 -->

# 6.4 VS CodeからHookへ渡される情報

**VS Codeは、Hookの起動時にイベント情報をJSONで渡します。** 受け取る経路は標準入力（`stdin`）です。以下は`Stop`の入力の抜粋です。

```json
{
  "hook_event_name": "Stop",
  "stop_hook_active": false
}
```

`hook_event_name`はイベント名です。`stop_hook_active`は、**前のStop Hookが終了を止めたため、Copilotが作業を続けているか**を示します。値はVS Codeが付けます。**どちらの場合もLintを実行し、失敗時の対応を変えます。**

| 値 | 状態 | Lint失敗時の処理 |
|---|---|---|
| `false` | 通常の終了前 | Copilotに修正を要求する |
| `true` | Stop Hookの要求で作業を続けている | 未解消の問題を示し、自動修正を停止する |

Pythonでは`json.load(sys.stdin)`で辞書`event`に読み込み、`event.get("stop_hook_active")`でこの項目の値を取得します。

JSONの`false`／`true`は、Pythonでは`False`／`True`になります。入力の項目名と型は[公式仕様](https://code.visualstudio.com/docs/agents/reference/hooks-reference#stop)で定義されています。

<!-- 発表メモ：約45秒。Hookの設定ファイルとは別に、呼び出すたびにVS Codeがイベント情報を標準入力へ渡す。ここでは共通項目のhook_event_nameと、Stop固有のstop_hook_activeを抜粋した。実際の入力にはtimestampや、任意のcwd、session_id、transcript_pathなどもある。stop_hook_activeはLintの合否やHook機能の有効・無効を示す値ではない。前回のStop Hookの継続要求によって作業を続けているかを示す。通常はfalseで、終了を止められた後の継続中はtrue。この値は修正やLintが成功したことを示さないため、どちらの場合もチェックを実行する。失敗した場合はfalseなら修正要求、trueなら未解消の問題を通知して自動修正を停止する。修正要求を1回に制限しつつ、再確認を省かない。VS Codeが渡すJSONをjson.loadでPythonの辞書へ変換し、その辞書をeventという変数に保存する。eventはプログラム側で付けた変数名。getは辞書から指定した項目の値を読む操作で、JSONのfalseとtrueはPythonのFalseとTrueになる。入力のhook_event_nameと次の出力のhookEventNameは仕様上の表記が異なるため、それぞれの形式に従う。 -->

# 6.5 失敗したとき、VS Codeへ何を返すか

**初回の失敗：終了を止め、Copilotに修正を要求する。** `decision: "block"`が継続の指定、`reason`がCopilotに渡す理由です。

```json
{
  "hookSpecificOutput": {
    "hookEventName": "Stop",
    "decision": "block",
    "reason": "Lintに失敗しました。修正して再確認してください。"
  }
}
```

**再開後も失敗：未解消の問題を利用者に示し、自動修正を打ち切る。** `continue: false`で作業を停止し、`stopReason`を利用者に示します。

```json
{
  "continue": false,
  "stopReason": "Lintが成功しませんでした。自動修正を停止します。"
}
```

VS Codeが定めた形式のJSONを標準出力へ返します。実装では理由に実行ログも添えます。[Stopの結果](https://code.visualstudio.com/docs/agents/reference/hooks-reference#stop)、[共通の出力仕様](https://code.visualstudio.com/docs/agents/reference/hooks-reference#common-output)

<!-- 発表メモ：約55秒。初回の失敗はStop固有のhookSpecificOutputを返す。hookEventNameは結果がStopに対応することを示す。decisionのblockは今回の終了を止め、reasonをモデルへ渡して作業を続けさせる指定。再開後に再確認しても失敗する場合は、共通出力のcontinueをfalseにしてエージェントの実行を停止する。stopReasonは利用者に示す理由で、成功したものとして終了させず、失敗を明示して人に判断を戻す。両方ともJSONを標準出力へ出した後、Hookプログラム自体は正常終了する。終了コード0はHookの結果を正しく返したという意味で、Lint合格とは別。初回と再開後の区別は入力のstop_hook_activeを使い、自動の修正要求を1回に制限する。成功時は何も返さず終了する。 -->

# 6.6 Pythonの実装：入力とLintの実行

**`check_lint.py`の前半です。再開後も、毎回Lintを実行します。**

```python
import json
import subprocess
import sys
event = json.load(sys.stdin)  # 入力JSONを辞書eventとして読む
try:  # Lintを実行し、結果を受け取る（上限10秒）
    result = subprocess.run("npm run lint", shell=True, capture_output=True,
                            text=True, timeout=10)
    # 終了コード0なら成功。失敗なら実行結果を理由に使う
    error = None if result.returncode == 0 else result.stdout + result.stderr
except subprocess.TimeoutExpired:
    error = "Lintが10秒以内に完了しませんでした。"
if error is None:
    sys.exit(0)  # Lintが成功したので、継続要求なしで終了する
```

**失敗や時間切れの場合は、次のスライドの処理へ進みます。** `stop_hook_active`だけで、チェックを省くことはありません。

Pythonの標準ライブラリだけを使用します。[subprocess.runの仕様](https://docs.python.org/3/library/subprocess.html#subprocess.run)

<!-- 発表メモ：約40秒。前半はイベント情報を受け取り、既存のnpm run lintを実行する処理。stop_hook_activeがTrueでも必ずチェックし、合否は実行結果で判断する。shell=Trueは固定のコマンドをシェル経由で実行し、Windowsのnpmにも対応する。入力値をコマンド文字列へ組み込まない。capture_output=Trueで標準出力と標準エラー出力を受け取り、text=Trueで文字列として扱う。returncodeの0はLintコマンドの成功。このときだけerrorをNoneにして、Hookから何も要求せずsys.exit(0)で終了する。失敗ならログを、10秒の時間切れならその理由をerrorに入れ、次のスライドの後半へ進む。前半と後半は同じcheck_lint.pyに続けて保存する。 -->

# 6.7 Pythonの実装：失敗時の結果を返す

**同じ`check_lint.py`の後半です。前半のコードの直後に続けます。**

```python
if event.get("stop_hook_active"):  # 再開後も失敗なら、自動修正を打ち切る
    output = {
        "continue": False,
        "stopReason": "Lintが成功しませんでした。自動修正を停止します。\n"
                      + error
    }
else:  # 初回の失敗なら、Copilotに修正を要求する
    output = {"hookSpecificOutput": {
        "hookEventName": "Stop", "decision": "block",
        "reason": "Lintに失敗しました。修正して再確認してください。\n" + error
    }}
print(json.dumps(output, ensure_ascii=False))  # 結果をVS Codeへ渡す
```

**どちらも実行ログを添えて返します。** 修正要求を繰り返し続けず、解消できなかった場合は利用者に判断を戻します。

Hookプログラムは結果を返して正常終了します。VS CodeはJSONの内容に従って動作します。[入力・出力仕様](https://code.visualstudio.com/docs/agents/reference/hooks-reference)

<!-- 発表メモ：約40秒。この後半に到達した時点でLintは失敗または時間切れになっている。stop_hook_activeがFalseなら初回の失敗として修正を要求し、Trueならすでに終了を止められて作業を続けた後なので自動修正を停止する。停止した理由と実行ログを利用者に示すため、無言で終了させることはない。出力が空の失敗でも、理由の文を含むJSONを必ず返す。PythonのFalseはjson.dumpsによってJSONのfalseになり、ensure_ascii=Falseで日本語をそのまま出力する。6.3のUTF-8設定と組み合わせる。printで標準出力へ渡した後、プログラムは末尾に到達して正常終了する。Lintの合否とHookプログラムの終了コードを区別する。前半と後半を同じファイルに続けて保存する。 -->

# 7. 来週からチームで試すなら

### 小さく始める3ステップ

1. **よくある失敗を3つ集める**：間違った確認コマンド、テーマ変更で見えない配色、テスト実行漏れなど
2. **適した機能に残す**：共通ルールはInstructions、作業固有の知識はSkills、イベントでの実行はHooks
3. **検証結果を残す**：実際にテストを実行したか、人の修正がどこで必要だったか

**今日の結論**

> Copilotに毎回同じ注意をするなら、その注意はチャットではなく**リポジトリに残す**。  
> 確認方法はInstructions／Skillsに残し、イベントで自動実行する処理は**Hooksに設定する**。

**主な出典**

- [Microsoft VS CodeのInstructions](https://github.com/microsoft/vscode/blob/main/.github/instructions/buildNext.instructions.md)
- [Microsoft VS Codeの共通Instructions](https://github.com/microsoft/vscode/blob/main/.github/copilot-instructions.md)
- [Microsoft VS Codeの配色Skill](https://github.com/microsoft/vscode/blob/main/.github/skills/ux-theming/SKILL.md)
- [VS CodeのAgent Skillsの使い方](https://code.visualstudio.com/docs/agent-customization/agent-skills)
- [VS CodeのHooksの設定](https://code.visualstudio.com/docs/agent-customization/hooks)

<!-- 発表メモ：約1分。次回、失敗を見つけたときに「今回は言い直す」か「ルールやテストに反映する」かを意識してもらう。説明はここで終了し、残りは質疑に使う。 -->

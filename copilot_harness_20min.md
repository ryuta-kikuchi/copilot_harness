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

Copilotにコードの修正を依頼すると、次のような問題が起こることがあります。

- 既存の設計ルールに合わない実装をする
- 「このファイルを見て」「そのコマンドは使わないで」と、チャットごとに伝え直す必要がある
- 以前説明して解消したはずの誤解や認識のずれが、別の依頼でも繰り返される
- コードを修正しても、テストを実行せずに作業を終える

**一因は、プロジェクト固有のルールや確認手順が、チャットをまたいで参照できる形で残されていないことです。**

<!-- 発表メモ：約1分20秒。チームが暗黙に知っているルールを毎回説明する負担に加え、一度説明して解消した認識のずれが別の依頼でも再発する点を取り上げる。チャットでの補足を再利用できる指示や手順に残すと、以後の作業でも参照できる。次のスライドで、そのための周辺環境をハーネスとして説明する。 -->

---

# 2. ハーネスとは、AIが作業するための周辺環境

**ハーネスは、AIモデルの周囲で、指示の読み込み・ツール実行・権限・作業の進行を扱う仕組みです。** Copilotが提供する仕組みを、利用者が自分のプロジェクトに合わせて設定します。

この勉強会では、まず次の3機能を扱います。

- **Instructions**：プロジェクトの前提やルールを伝える
- **Skills**：特定作業のルール・手順・参考資料をまとめる
- **Hooks**：決めたイベントで、処理を自動実行する

**InstructionsとSkillsは、どちらもルールと手順を含められます。主な違いは、いつ読み込まれるかです。**

Hooksは、設定したイベントで処理を起動します。「その処理を実行するか」をLLMの判断に委ねない点が異なります。

<small>出典：[VS Codeのカスタマイズの考え方](https://code.visualstudio.com/docs/agents/concepts/customization)。ハーネスの設定は、この3機能以外にもあります。</small>

<!-- 発表メモ：約1分。ハーネス全体はInstructionsとSkillsを書くことだけではない。ここでは利用者が設定する入口として3機能を選ぶ。指示をモデルが解釈することと、イベントで処理が起動することを区別する。Hooksの処理自体の成功や、コードの正しさを保証する意味ではない。 -->

---

# 3. 全体像：何をどこに設定するか

| 機能 | 設定するもの | リポジトリ内の場所 |
|---|---|---|
| **Instructions** | 共通の規約、対象ごとの注意点、検証方針 | 共通：`.github/copilot-instructions.md`<br>対象別：`.github/instructions/*.instructions.md` |
| **Agent Skills** | 特定作業のルール、手順、参考資料 | `.github/skills/名前/SKILL.md`など |
| **Hooks** | イベントと、その時に実行するコマンド | `.github/hooks/*.json`など<br>実行するスクリプトも用意する |

**検証方法を伝えるならInstructionsやSkills、決まったタイミングで処理を実行するならHooksです。**

InstructionsとSkillsはVS Codeの公開リポジトリ、HooksはVS Code公式ドキュメントの設定例で説明します。

<small>出典：[カスタマイズの全体像](https://code.visualstudio.com/docs/agents/concepts/customization)、[Hooksの設定](https://code.visualstudio.com/docs/agent-customization/hooks)。対応機能や設定形式は実行環境で異なります。</small>

<!-- 発表メモ：約1分20秒。検証を独立した第三の設定と呼ばず、InstructionsやSkillsに検証方針を残し、必要ならHooksで確認処理を起動する関係を説明する。HooksはVS Codeの公開リポジトリで実運用されている例ではなく、公式ドキュメントの例を扱う。 -->

---

# 3.1 ほかのカスタマイズ機能

| 機能 | 使う場面 |
|---|---|
| **Prompt Files** | レビュー依頼などの定型文を保存し、明示的に呼び出す |
| **Custom Agents** | レビュー役など、役割と使えるツールをまとめて設定する |
| **MCP** | 外部システムを操作するツールを追加する。例：社内Issue管理システムから仕様情報を取得する |

**まず困っていることに合う機能から始めます。** プロジェクトの前提不足ならInstructions、作業固有の知識ならSkills、イベントでの実行ならHooksを選びます。

<small>出典：[VS Codeのカスタマイズ一覧](https://code.visualstudio.com/docs/agents/concepts/customization)。関連する設定をまとめて配布するPluginsもあります。</small>

<!-- 発表メモ：約30秒。機能の存在と用途だけを紹介する。MCPは外部のツールを使えるようにする接続の仕組みで、作業手順を記載するSkillとは役割が異なる。全てを最初から追加する必要はない。 -->

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
- 実行中のビルドがあれば、まず既存の結果を確認する。
- 変更に関連する既存のテストを選び、実行する。
```

**「共通」は、そのリポジトリ全体に適用するという意味です。** コードの規約と、変更後の確認方法を同じ指示書に残せます。

<small>出典：[VS Codeの共通Instructions](https://github.com/microsoft/vscode/blob/main/.github/copilot-instructions.md)。本文は原文の日本語要約・抜粋。</small>

<!-- 発表メモ：約1分20秒。共通Instructionsにはコードの規約と検証方針の両方を書く。検証は独立した設定機能ではなく、この指示書に残す内容の一つ。原文にはフォルダーの役割や設計の説明も含まれる。camelCaseは先頭を小文字にし、単語の区切りを大文字で表す命名方式。テストはコードの動作を確認するプログラムで、変更した機能に関連するものを選んで実行する。広い範囲や型・ビルド設定の変更では、必要に応じて型チェックやビルドも行う。共通はVS Code内の共通方針で、次は対象別の確認方法へ進む。実演は行わない。 -->

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

Copilotでは、リポジトリの`.github/skills/名前/SKILL.md`などに配置します。

通常は依頼内容と`name`・`description`から関連するSkillを選び、本文を読み込みます。Skill名を毎回指定する必要はありません。必要なら`/Skill名`で明示的に呼び出せます。

次のVS Codeの実例では、画面の配色を実装・修正するときに必要なルールと具体的な実装方法を、一つのSkillにまとめています。

<small>仕様：[GitHub公式 Agent Skills](https://code.visualstudio.com/docs/agent-customization/agent-skills)。読み込みや手順の実行は、実際のログで確認します。</small>

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

# 6. Hook：イベントをきっかけに処理を起動する

**イベントは、エージェントの作業中に起きる、決められた出来事です。** 例えば、次のような節目があります。

- 利用者が依頼を送信した時
- エージェントが操作を実行する直前・直後
- エージェントが今回の処理を終える直前

**Hookでは「どのイベントで、どのプログラムを動かすか」を設定します。** 設定したイベントが起きると、実行環境がプログラムを起動します。

> 例：依頼が送信された時に、受付日時を記録するプログラムを動かす。

**イベントが実行のきっかけ、Hookがその時に追加する処理の設定です。** 次にイベントの代表例を見て、その中の「ツール実行前・後」を詳しく説明します。

<small>この章はVS Code拡張機能側で動くLocalを例に説明します。[Hookの公式説明](https://code.visualstudio.com/docs/agent-customization/hooks)</small>

<!-- 発表メモ：約35秒。イベントとHookの関係から話を始める。イベントは実行環境が検知する決められた出来事で、利用者が依頼を送る、操作を始める、今回の実行を終えるなどの節目がある。利用者は、そのイベントで起動するプログラムをHookとして設定する。例として依頼送信時に日時を記録する処理を挙げる。イベントの前後という言い方だけでは、すべてのイベントに前後の組があるように聞こえるため、イベントが起きた時に処理を動かすと説明する。ツール自体は操作の機能であり、ツール呼び出しの直前・正常完了後がイベントになる。ここではツールの詳細を先に展開せず、次のイベント一覧からツール実行前後の話へ進む。記録などの追加処理は利用者が用意するプログラムに実装する。Hook設定だけで処理本体が生成されるわけではない。 -->

---

# 6.1 Hookが働くイベントの代表例

**作業全体の節目と、一つ一つの操作の節目があります。**

| イベント | いつ起きるか | Hookの用途例 |
|---|---|---|
| `SessionStart` | 最初の依頼でセッションが始まる時 | プロジェクトの前提を初期情報として渡す |
| `UserPromptSubmit` | 利用者が依頼を送信した時 | 依頼を受け付けた日時を記録する |
| `PreToolUse` | 各ツールを呼び出す直前 | 操作を記録する、実行前に内容を確認する |
| `PostToolUse` | ツール呼び出しが正常に終わった後 | 結果を確認する、編集したコードを整形する |
| `Stop` | エージェントが今回の処理を終える直前 | 未確認のテストがあれば継続を要求する |

**ツールは、ファイル操作やコマンド実行などに使う機能です。** 次に、ツールを使って作業を進める仕組みと、その種類を見ます。

<small>VS Code Localの代表例。Stopはチャット全体の終了とは限りません。[イベントの公式一覧](https://code.visualstudio.com/docs/agent-customization/hooks#local-hook-lifecycle-events)</small>

<!-- 発表メモ：約40秒。ツール呼び出しだけがHookのタイミングではないと説明する。SessionStartは最初の依頼でセッションが始まるタイミングで、例えばプロジェクト名や実行環境などを初期情報として追加できる。UserPromptSubmitは依頼送信時。PreToolUseとPostToolUseはツール呼び出しごとの前後で、PostToolUseは呼び出しが正常に完了した場合に起きる。ツール呼び出しの正常終了と、その中で実行したテストの合格は別なので、結果の内容を確認する。Stopは現在のエージェント実行を終える前であり、チャットを閉じる時ではない。Stopで必要な確認が未完了なら理由を返して継続を要求することもできるが、stop_hook_activeなどによる繰り返し制御が必要。検証結果の確認には、単にツールを呼んだ記録ではなく実際の結果を使う。用途例は公式仕様を基にした提案で、設定・処理本体を用意する必要がある。ほかにPreCompactやSubagentStart、SubagentStopもあるが、ここでは代表例に絞る。 -->

---

# 6.2 ツールでファイルを修正する仕組み

**依頼例：「この関数のバグを直して」**

修正案を考えるだけでは、保存されているファイルは変わりません。**ファイルを実際に書き換える操作が必要です。**

1. **CopilotのAIモデル**が、渡されたコードを基に修正内容を考える。
2. **Copilot**が、「この部分を置き換える」という操作を要求する。
3. **VS Code側の仕組み**が、その要求を受けて操作を進める。

**ツールとは、Copilotが作業に使う操作機能です。** ファイルを読む・書き換える・コマンドを実行する、といった機能を指します。

次に、使うツールの代表例を見ます。

<small>この章はVS Code拡張機能側で動くLocalを例に説明します。[ツールの仕組み](https://code.visualstudio.com/docs/agents/concepts/tools)</small>

<!-- 発表メモ：約40秒。利用者がCopilotにバグ修正を頼む場面から始める。AIモデルが修正案を出すことと、実際のファイルが書き換わることを区別する。モデルは使うツールと渡す内容を選び、実行環境が操作を進め、結果をモデルへ戻す。ツールは操作の機能で、ファイルそのものや画面上の工具アイコンではない。この章ではVS Code Localの実行環境を「VS Code側の仕組み」と呼ぶ。権限の確認などを経て実行されるので、要求すれば無条件に操作できるという意味ではない。次はツールの代表例を見た後、ツール実行前に記録処理を挟む例へ進む。参照：https://code.visualstudio.com/docs/agents/concepts/tools 。 -->

---

# 6.2.1 Copilotが使うツールの代表例

**ツールは、情報を集める操作にも、変更を加える操作にも使います。**

| ツールの仕事 | 具体的にすること |
|---|---|
| プロジェクト内の検索 | 関連する関数や、呼び出している箇所を探す |
| ファイルの読み込み | ソースコードやREADME、設定ファイルの内容を取得する |
| ファイルの編集 | コードを置き換える、新しいファイルを作る |
| ターミナルでのコマンド実行 | テスト、ビルド、Gitのコマンドを実行する |
| 外部サービスとの連携（MCPなど） | 社内Issueから仕様を取得する、Issueの状態を更新する |

例えば**テストも、ターミナルのツールでコマンドを実行して進められます。** 次は、それぞれの操作でHookをどう使うかを見ます。

<small>表は操作の分類です。実際のツール名や利用できる機能は環境で異なります。[ツールの公式説明](https://code.visualstudio.com/docs/agents/concepts/tools)</small>

<!-- 発表メモ：約35秒。ツールの代表的な仕事を具体例で示す。読み込みや検索のように情報を得る操作も、編集やコマンド実行のように環境を変える操作もツールに含まれる。テスト実行やGit操作には、例えばターミナルのツールで対応するコマンドを実行する方法がある。「テスト実行」という名前の独立したHookイベントを想定する必要はない。表は実際のツール名の一覧ではなく用途による分類で、一つのツールで複数の仕事ができる場合もある。MCPは外部サービスのツールを追加する接続の仕組みで、使うにはその連携が設定されている必要がある。ターミナルのコマンドでファイルを書き換える場合もあるので、ファイル編集ツールだけを見れば全変更が分かるという意味ではない。 -->

---

# 6.3 イベントとツールを組み合わせるHookの例

Instructions／Skillsはモデルに行動を促します。Hookでは**実行環境がタイミングを検知し、処理を起動**します。

| イベント | 対象のツール・操作 | 起動する処理の例 |
|---|---|---|
| `PreToolUse` | ファイル読み込み・検索 | 参照先や検索条件を記録し、調査の経路を残す |
| `PreToolUse` | ターミナルのコマンド実行 | 削除を含む操作なら、実行前に承認を求める |
| `PostToolUse` | ファイル編集 | 変更したファイルだけ、インデントなどの書式を整える |
| `PreToolUse` | MCPで外部Issueを更新 | 更新先と変更内容を確認し、承認を求める |

**イベントで「いつ」を決め、ツール名と操作の指定内容（引数）で対象を絞ります。** 次からは、ツール実行前にログを残す例の仕組みと設定を見ます。

<small>Local用の[用途例（公式仕様）](https://code.visualstudio.com/docs/agents/reference/hooks-reference)。実際のツール名と引数を確認して実装します。実行先ごとに[仕様](https://code.visualstudio.com/docs/agent-customization/hooks#choose-the-hook-implementation-for-your-session)が異なります。HooksはPreview（2026年10月13日時点）。</small>

<!-- 発表メモ：約40秒。具体的なイベントと対象の操作を組み合わせる。PreToolUseはファイル編集専用ではなく各ツールの実行前に起きる。同じイベントでも、Hookプログラムがtool_nameとtool_inputを確認して処理を分ける。ファイルの読み込みや検索は参照先・検索条件を記録して後から調査の経路を追えるようにする。この後のログ例は日時とツール名だけを記録するので、参照先まで残すには引数を読む処理を追加する。ターミナルでは、実行するコマンドの内容を調べ、削除など対象の操作ならPreToolUseのpermissionDecision:askで承認を求める。コマンド名だけを単純な文字列比較で見る実装では複雑なコマンドを取りこぼすので、実運用では対象を具体的に定義する。ファイル編集のPostToolUseでは、編集ツールであることと対象ファイルを確認し、そのファイルだけ既存の整形コマンドで書式をそろえる。検索や読み込み後に全体を整形する設定にはしない。MCPもツールなので、例えばIssueの取得は通し、更新などの書き込み時だけ承認を求める使い方ができる。各例は用途の提案で、実際のツール名や引数の形式はデバッグログなどで確認する。条件に該当しない呼び出しでは追加処理をしない。今回の設定例とイベント名はVS Code Local用で、Hooks機能はPreview（2026年10月13日時点）。 -->

---

# 6.4 Hookを挟む位置：操作の前に記録する

**今回の目的**：いつ・どのツールを使おうとしたかを、後から確認したい。そのために残す作業記録のファイルを**ログ**と呼びます。

**Hookは、決めたタイミングで、用意したプログラムを起動する仕組みです。** 今回は、ツールを使う直前に記録用プログラムを動かします。

1. **Copilot**が、ファイルを書き換えるツールを使おうとする。
2. **VS Code**が、ツール実行前というタイミングを検知する。
3. **VS Code**が、Hookの設定に従って記録用プログラムを起動する。
4. **記録用プログラム**が、日時とツール名をログに書く。
5. **VS Code**が、元のファイル書き換え操作へ進む。

**元の仕事はバグ修正です。その途中に、記録する処理を追加しています。**

<small>出典：[VS Code公式のログ記録例](https://code.visualstudio.com/docs/agent-customization/hooks#create-your-first-local-hook)</small>

<!-- 発表メモ：約45秒。何を自動化したいかを先に説明する。ログは後から見るための作業記録で、チャットの返答とは別のファイルに保存する。元の操作に入る前に、ログ記録という追加の処理を挟む。Hookの設定には実行するタイミングと起動するプログラムを指定する。Copilotは元の作業のツールを選ぶが、ログ記録プログラムを動かすかどうかはVS Code側の設定で決まる。この例のプログラムが正常に完了した後、通常の権限確認などを経て元の操作へ進む。ログ記録が操作の成功を保証するとは説明しない。 -->

---

# 6.5 利用者が用意する設定とプログラム

| ファイル | 役割 |
|---|---|
| `audit.json` | VS Codeが読む設定。「いつ・何を起動するか」を指定する |
| `log-tool-use.cjs` | 起動されるプログラム。実際にログを書く処理を実装する |

保存先は`.github/hooks/`。設定を項目名と値で表す形式をJSONと呼びます。

```json
{
  "hooks": {
    "PreToolUse": [
      { "type": "command", "command": "node .github/hooks/log-tool-use.cjs" }
    ]
  }
}
```

`PreToolUse`＝ツールを使う前。`command`＝起動するコマンド。

<small>設定はVS Code Local用。[公式の設定例](https://code.visualstudio.com/docs/agent-customization/hooks#create-your-first-local-hook)</small>

<!-- 発表メモ：約35秒。Hook設定と処理本体の2つが必要な理由を説明する。毎朝9時に仕事を実行する予約と、仕事の中身を用意する関係に近い。Hookでは時計の時刻ではなくツール実行前などの出来事がきっかけになる。JSONは項目名と値で設定を記述する形式。hooksの中にPreToolUseというタイミングを指定し、その配列に実行する処理を登録する。typeのcommandは処理の種類で、次のcommandは実行するコマンドそのもの。nodeはNode.jsを起動するコマンドで、その後ろのパスが実行するプログラムのファイル。cjsはNode.jsで実行するJavaScriptファイルの形式。audit.jsonもlog-tool-use.cjsも利用者が用意する。設定ファイルを作るだけでログを書く処理が生成されるわけではない。実際に使うときはNode.jsが必要で、VS Codeの実行先をLocalにし、chat.useHooksが有効でワークスペースが信頼された環境に置く。この資料ではHook自体を有効化しない。 -->

---

# 6.6 記録用プログラムが行うこと

**`log-tool-use.cjs`をNode.js（JavaScriptの実行環境）で動かします。**

```javascript
const fs = require('node:fs');
const input = fs.readFileSync(0, 'utf8');
const event = JSON.parse(input);
const line = event.timestamp + ' ' + event.tool_name + '\n';
fs.appendFileSync('.github/hooks/tool-use.log', line);
```

1. **受け取る**：`0`は標準入力。VS Codeからデータを受け取る入口です。
2. **取り出す**：JSON形式の文字列をデータに変え、日時とツール名を参照する。
3. **保存する**：2項目を空白でつなぎ、`tool-use.log`の末尾へ1行追加する。`\n`は改行。

**日時とツール名はVS Codeが渡します。** 毎回手入力する必要はありません。

<small>出典：[公式の記録プログラム](https://code.visualstudio.com/docs/agent-customization/hooks#create-your-first-local-hook)を、同じ記録処理になるよう短く書き換えた例。</small>

<!-- 発表メモ：約35秒。コードの文法より、受け取る・取り出す・保存するという3つの仕事を説明する。VS Codeはツール呼び出しを扱う実行環境なので、今回どのツールを使おうとしているかを把握し、その情報をプログラムに渡せる。記録プログラムがチャットからツール名を推測するわけではない。fsはファイルの読み書きをするNode.jsの標準機能。readFileSyncの0は標準入力を指定する番号で、utf8は文字を読む方式。JSON.parseで入力の文字列を項目ごとに参照できるデータに変え、timestampが日時、tool_nameがツール名を参照する。appendFileSyncは既存の記録を消さず末尾へ追加し、ファイルがなければ作成する。公式例はイベントで標準入力を読み取るが、ここでは同期読み取りに短く書き換えた。正しいLocalのJSON入力が渡され、リポジトリを作業ディレクトリとして実行する前提。エラー処理は省略している。 -->

---

# 6.7 渡される情報と、実際に残る記録

**VS Codeから記録用プログラムへ渡す情報の例**

```json
{
  "timestamp": "2026-10-13T10:00:00+09:00",
  "tool_name": "example_edit_tool"
}
```

`timestamp`＝日時、`tool_name`＝ツール名。空白でつないで1行にします。

**`.github/hooks/tool-use.log`に追加される記録**

```text
2026-10-13T10:00:00+09:00 example_edit_tool
```

ツールを呼ぶたびに1行追加されます。ログで**呼び出し前の日時とツール名**を確認できます。操作の成功は別に確認します。

<small>入力は2項目を抜粋。日時とツール名は説明用の架空例。[Localの入力仕様](https://code.visualstudio.com/docs/agents/reference/hooks-reference)</small>

<!-- 発表メモ：約35秒。1回の操作を入力と出力で確認する。日時とツール名がVS Codeから渡され、前のコードのevent.timestampとevent.tool_nameで読まれて、空白でつながった1行になる。実際の入力にはイベント名やツールへの引数などもあるが、記録に使う2項目に絞っている。example_edit_toolは説明用の架空名で、利用できる実際のツール名ではない。実際の名前は環境のログなどで確認する。日時も架空値で、Hookを実行した結果ではない。ツールを使おうとするたびこの処理が起動し、既存の記録に行が追加される。操作前に書くので、その後ツールが失敗しても記録は残る。ツールを使おうとした記録と成功した証拠を区別する。 -->

---

# 7. 自分のチームでは何を設定するか

| 必要なこと | 設定するもの |
|---|---|
| 全体の規約や検証方針を毎回伝えたい | 共通Instructions |
| 特定のファイルで必要な注意点を残したい | 対象別Instructions |
| 作業固有のルール・実装方法・参照先をまとめたい | Skill |
| イベントが起きたら処理を実行したい | Hookの設定と実行スクリプト |

**検証の役割分担**：確認方法はInstructions／Skillsに記載し、自動実行が必要ならHooksを組み合わせます。実行ログと未確認事項は、利用者が確認します。

<small>VS Codeのプロジェクト規約は公開ファイル、Hooksは公式ドキュメントの設定例を参照しています。</small>

<!-- 発表メモ：約1分。利用者が設定する具体物を整理する。InstructionsやSkillsに書くことと、Hookで起動することを分ける。Hooksはカスタマイズ機能の一つで、テストだけの機能ではない。プロジェクトに既存のCIがあるなら、その確認も併用する。 -->

---

# 8. 導入後に確認すべき点と落とし穴

| 起きること | まず確認すること |
|---|---|
| Instructionsが効かない | ルールの保存先が `.github/copilot-instructions.md` か、対応する実行環境か |
| Skillを使ってくれない | `name`、`description`、保存場所、明示的な呼び出し |
| テストしたと言うが怪しい | **実行したコマンドとログ**を確認する |
| Hookが動かない | 実行先、設定形式、スクリプトの場所、実行ログを確認する |
| 毎回同じ誤実装をする | 誤りをInstructions、Skill、テストのどこに残すべきか |
| 設定ファイルが巨大になる | 共通ルール、対象固有ルール、作業手順に分ける |

Instructions／Skillsへの記載だけで実行は保証されません。Hooksでチェックを起動する場合も、結果を確認します。共有コードに取り込む条件は、CIでも設定できます。

<small>参考：[対応環境の公式一覧](https://docs.github.com/en/copilot/reference/custom-instructions-support)、[VS CodeのSkill利用](https://code.visualstudio.com/docs/agent-customization/agent-skills)、[Hooksの設定](https://code.visualstudio.com/docs/agent-customization/hooks)</small>

<!-- 発表メモ：約1分20秒。Copilot ChatにはAgentやAskなどのモードがある。ファイル操作やコマンド実行が必要ならAgentモードを使う。組織のポリシーや権限により、利用できるツールが少ない場合がある。 -->

---

# 8.1 配色の例：実行結果と未確認事項を分ける

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

# 9. 来週からチームで試すなら

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

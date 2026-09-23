# Kiro IDE で Agent フックと Agent Skills の設定 🛠️

## 概要

Kiro IDE に Agent フックと Agent Skills を設定して使用します。

**所要時間**: 約10〜15分

**前提条件**:
- ラボ 2 の環境で Kiro にサインができていること

---
## 準備

1. まず、docstring（JSDoc）がない Node.js ファイルを作成します。チャット欄に以下を入力します:

```
以下の内容で calculator.js を作成してください（コメントは付けないでください）:

function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

function divide(a, b) {
  if (b === 0) {
    throw new Error("0で割ることはできません");
  }
  return a / b;
}

module.exports = { add, subtract, multiply, divide };
```

2. `calculator.js` が作成されたことを確認します

---
## Agent フックの設定

ここでは、Kiro が Node.js ファイルを作成・保存したときに、ログファイルに記録を残すフックを設定します。ファイルに書き出すことで、フックが実際に発火したことを目で確認できます。

1. チャット欄に以下を入力して送信します:

```
Agent フックを作成してください。

内容:
- 名前: Node.js ファイル変更ログ
- トリガー: PostFileSave（ファイル保存後）
- マッチャー: .js ファイルのみ対象
- アクション: コマンドで、現在日時を hook-log.txt に追記する（PowerShell の Add-Content を使用）

コマンド例: powershell -Command "Add-Content -Path 'hook-log.txt' -Value ('JavaScript file saved at ' + (Get-Date -Format 'yyyy-MM-dd HH:mm:ss'))"
```

2. Kiro がフックファイル（`.kiro/hooks/` 配下）を作成するのを確認します
3. Kiro の左側で Kiro のアイコンをクリックし、「**AGENT HOOKS**」セクションにフックが表示されることを確認します

> 💡 **ポイント**: フックは `.kiro/hooks/` フォルダに JSON ファイルとして保存されます。トリガー（いつ実行するか）とアクション（何を実行するか）を定義します。

---

## フックの動作確認

フックはエージェント（Kiro）がファイルを操作したときに発火するため、Kiro にファイルの修正を依頼して動作を確認します。

1. チャット欄に以下を入力して送信します:

```
calculator.js にコメントを1行追加してください。内容は「// フックのテスト」としてください。
```

2. Kiro が `calculator.js` を修正するのを確認します
3. エクスプローラーでワークスペース内に `hook-log.txt` が作成されていることを確認します
4. `hook-log.txt` を開き、フックが実行された日時のログが記録されていることを確認します

> 💡 **ポイント**: フックは Kiro（エージェント）のファイル操作をトリガーに実行されます。今回はログファイルへの記録で発火を確認しましたが、実際の開発ではリント実行やテスト実行の自動化に活用できます。ユーザーの手動保存では発火しない点に注意してください。

---

## Agent Skills の設定と確認

Agent Skills を使って、Kiro に特定の作業パターンを教え、再利用可能な手順として呼び出す方法を体験します。

---

### スキルの設定

ここでは、Node.js ファイルのドキュメントコメント（JSDoc）を生成するスキルを作成します。

> 💡 **Agent Skills とステアリングの違い**: ステアリングはプロジェクトのルール・規約を定義する単一の Markdown ファイルです。Agent Skills は再利用可能なワークフロー（手順）を定義するもので、フォルダ単位で管理し、[agentskills.io](https://agentskills.io) のオープン標準に準拠しています。

スキルのフォルダ構成:
```
.kiro/skills/
└── jsdoc-comment/         ← スキル名のフォルダ
    └── SKILL.md           ← スキル定義ファイル（必須）
```

1. チャット欄に以下を入力して送信します:

````
以下の内容でスキルを作成してください。

フォルダとファイル: .kiro/skills/jsdoc-comment/SKILL.md
内容:

---
name: jsdoc-comment
description: JavaScript の関数やクラスに日本語の JSDoc コメントを自動生成する。docstring やドキュメントコメントの追加、ドキュメント整備を依頼されたときに使用。
---

## 手順

1. 指定された JavaScript ファイルを読み込む
2. JSDoc コメントが未記載の関数・クラスを特定する
3. 各関数・クラスに以下の形式で JSDoc コメントを追加する

## JSDoc のフォーマット

日本語で記述する:

```javascript
/**
 * 合計金額を計算する。
 *
 * 税率を適用して最終的な合計金額を算出します。
 *
 * @param {number} price - 商品の価格（税抜き）
 * @param {number} taxRate - 税率（例: 0.1 は10%）
 * @returns {number} 税込みの合計金額
 * @throws {Error} price が負の値の場合
 */
function calculateTotal(price, taxRate) {
  // ...
}
```


## ルール

- コメントは日本語で記述する
- 既に JSDoc コメントがある関数は上書きしない
- 引数や戻り値の型が推測できる場合は `@param`・`@returns` に記載する
- `@throws` は該当する場合のみ記載する
````

2. Kiro がスキルフォルダとファイルを作成するのを確認します
3. Kiro の左側で Kiro のアイコンをクリックし、「**AGENT STEERING & SKILLS**」セクションで `jsdoc-comment` スキルが表示されることを確認します

> 💡 **ポイント**: Agent Skills は `name` と `description` をフロントマターに定義します。`description` の内容をもとに、Kiro がユーザーのリクエストに合致する Agent Skills を自動的にマッチングして呼び出します。また、チャットで `/jsdoc-comment` と入力して明示的に呼び出すこともできます。

---

### Agent Skills の動作確認


1. チャット欄に以下を入力して送信し、Agent Skills を呼び出します:

```
calculator.js の全ての関数に JSDoc コメントを追加してください。
```

   - Kiro が `jsdoc-comment` スキルの `description` にマッチし、このスキルが自動的にアクティベートされます
   - または、明示的に `/jsdoc-comment` と入力して呼び出すこともできます

2. Kiro がこのスキルの手順に従い、各関数に日本語の JSDoc コメントを追加するのを確認します
3. `calculator.js` を開き、JSDoc コメントが追加されていることを確認します

> 💡 **ポイント**: Agent Skills は agentskills.io のオープン標準に準拠しているため、チーム内での共有はもちろん、他の AI ツールとの互換性もあります。再利用可能なワークフローをスキルとして定義し、チーム全員で同じ品質の作業を実行できます。


---

## 振り返り

このハンズオンで体験した Kiro の機能をまとめます:

| 機能 | 体験した内容 | 活用シーン |
|------|------|------|
| Agent フック | ファイル保存時の自動構文チェック | CI/CD・品質管理の自動化 |
| Agent Skills | JSDoc コメントの自動生成 | チーム標準の作業パターン共有 |


---

## FAQ

| 質問 | 回答 |
|------|------|
| Agent フックと Agent Skills の違いは？ | フックはイベント駆動で自動実行されます。スキルはユーザーが明示的に呼び出すテンプレートです |
| Agent Skills はチーム共有できる？ | はい。`.kiro/skills/` フォルダごと Git で管理すればチーム全員で共有できます |

---

## トラブルシューティング

| 症状 | 対処 |
|------|------|
| Agent フック が動作しない | `.kiro/hooks/` 内の JSON ファイルの構文を確認。Kiro を再起動してみる。|
| Agent Skills が表示されない | `.kiro/skills/` 内のファイルパスとフロントマターの `inclusion` 設定を確認 |


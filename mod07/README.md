#### Agent フックの設定

ここでは、Kiro が Python ファイルを作成・保存したときに、ログファイルに記録を残すフックを設定します。ファイルに書き出すことで、フックが実際に発火したことを目で確認できます。

1. チャット欄に以下を入力して送信します:

```
Agent フックを作成してください。

内容:
- 名前: Python ファイル変更ログ
- トリガー: PostFileSave（ファイル保存後）
- マッチャー: .py ファイルのみ対象
- アクション: コマンドで、現在日時を hook-log.txt に追記する（PowerShell の Add-Content を使用）

コマンド例: powershell -Command "Add-Content -Path 'hook-log.txt' -Value ('Python file saved at ' + (Get-Date -Format 'yyyy-MM-dd HH:mm:ss'))"
```

2. Kiro がフックファイル（`.kiro/hooks/` 配下）を作成するのを確認します
3. Kiro の左側で Kiro のアイコンをクリックし、「**AGENT HOOKS**」セクションにフックが表示されることを確認します

> 💡 **ポイント**: フックは `.kiro/hooks/` フォルダに JSON ファイルとして保存されます。トリガー（いつ実行するか）とアクション（何を実行するか）を定義します。

---

#### フックの動作確認

フックはエージェント（Kiro）がファイルを操作したときに発火するため、Kiro にファイルの修正を依頼して動作を確認します。

1. チャット欄に以下を入力して送信します:

```
hello.py にコメントを1行追加してください。内容は「# フックのテスト」としてください。
```

2. Kiro が `hello.py` を修正するのを確認します
3. エクスプローラーでワークスペース内に `hook-log.txt` が作成されていることを確認します
4. `hook-log.txt` を開き、フックが実行された日時のログが記録されていることを確認します

> 💡 **ポイント**: フックは Kiro（エージェント）のファイル操作をトリガーに実行されます。今回はログファイルへの記録で発火を確認しましたが、実際の開発ではリント実行やテスト実行の自動化に活用できます。ユーザーの手動保存では発火しない点に注意してください。

---

### Agent Skills の設定と確認

Agent Skills を使って、Kiro に特定の作業パターンを教え、再利用可能な手順として呼び出す方法を体験します。

---

#### 5-1. スキルの設定

ここでは、Python ファイルのドキュメント文字列（docstring）を生成するスキルを作成します。

> 💡 **Agent Skills とステアリングの違い**: ステアリングはプロジェクトのルール・規約を定義する単一の Markdown ファイルです。Agent Skills は再利用可能なワークフロー（手順）を定義するもので、フォルダ単位で管理し、[agentskills.io](https://agentskills.io) のオープン標準に準拠しています。

スキルのフォルダ構成:
```
.kiro/skills/
└── python-docstring/      ← スキル名のフォルダ
    └── SKILL.md           ← スキル定義ファイル（必須）
```

1. チャット欄に以下を入力して送信します:

````
以下の内容でスキルを作成してください。

フォルダとファイル: .kiro/skills/python-docstring/SKILL.md
内容:

---
name: python-docstring
description: Python の関数やクラスに Google スタイルの日本語 docstring を自動生成する。docstring の追加やドキュメント整備を依頼されたときに使用。
---

## 手順

1. 指定された Python ファイルを読み込む
2. docstring が未記載の関数・クラスを特定する
3. 各関数・クラスに以下の形式で docstring を追加する

## docstring のフォーマット

Google スタイルで、日本語で記述する:

```python
def calculate_total(price: int, tax_rate: float) -> float:
    """合計金額を計算する。

    税率を適用して最終的な合計金額を算出します。

    Args:
        price: 商品の価格（税抜き）
        tax_rate: 税率（例: 0.1 は10%）

    Returns:
        税込みの合計金額

    Raises:
        ValueError: price が負の値の場合
    """
```


## ルール

- docstring は日本語で記述する
- 既に docstring がある関数は上書きしない
- 型ヒントがある場合はそれを参考にする
- Args、Returns、Raises の各セクションは該当する場合のみ記載する
````

2. Kiro がスキルフォルダとファイルを作成するのを確認します
3. Kiro の左側で Kiro のアイコンをクリックし、「**AGENT STEERING & SKILLS**」セクションで `python-docstring` スキルが表示されることを確認します

> 💡 **ポイント**: Agent Skills は `name` と `description` をフロントマターに定義します。`description` の内容をもとに、Kiro がユーザーのリクエストに合致する Agent Skills を自動的にマッチングして呼び出します。また、チャットで `/python-docstring` と入力して明示的に呼び出すこともできます。

---

#### 5-2. Agent Skills の動作確認

1. まず、docstring がない Python ファイルを作成します。チャット欄に以下を入力します:

```
以下の内容で calculator.py を作成してください（docstring は付けないでください）:

def add(a: int, b: int) -> int:
    return a + b

def subtract(a: int, b: int) -> int:
    return a - b

def multiply(a: int, b: int) -> int:
    return a * b

def divide(a: int, b: int) -> float:
    if b == 0:
        raise ZeroDivisionError("0で割ることはできません")
    return a / b
```

2. `calculator.py` が作成されたことを確認します

3. チャット欄に以下を入力して送信し、Agent Skills を呼び出します:

```
calculator.py の全ての関数に docstring を追加してください。
```

   - Kiro が `python-docstring` スキルの `description` にマッチし、このスキルが自動的にアクティベートされます
   - または、明示的に `/python-docstring` と入力して呼び出すこともできます

4. Kiro がこのスキルの手順に従い、各関数に日本語の docstring を追加するのを確認します
5. `calculator.py` を開き、Google スタイルの docstring が追加されていることを確認します

> 💡 **ポイント**: Agent Skills は agentskills.io のオープン標準に準拠しているため、チーム内での共有はもちろん、他の AI ツールとの互換性もあります。再利用可能なワークフローをスキルとして定義し、チーム全員で同じ品質の作業を実行できます。


---

### （オプション）MCP サーバーの設定と使用

MCP（Model Context Protocol）サーバーを設定し、Kiro に外部ツールを連携させる方法を体験します。

> ⚠️ **注意**: この手順は Node.js / npx がインストールされている環境でのみ実施できます。環境によっては MCP サーバーが利用できない場合がありますので、その場合はスキップしてください。

---


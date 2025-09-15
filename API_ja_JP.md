# SiYuan API ドキュメント

SiYuan ノート作成アプリケーション API の包括的なガイドです。この API を使用して、プログラムで SiYuan とやり取りし、ノートブック、ドキュメント、ブロックを管理し、さまざまな操作を実行できます。

**言語オプション：**
- [English](API.md)
- [中文 (Chinese)](https://github.com/siyuan-note/siyuan/blob/master/API_zh_CN.md)

---

## 概要

SiYuan API は、ナレッジベースとやり取りするための RESTful インターフェースを提供します。以下のことができます：

- **ノートブックの管理**: ノートブックの作成、開く、閉じる、名前変更、削除
- **ドキュメントの処理**: Markdown からドキュメントを作成、ファイル構造の整理
- **ブロックの操作**: コンテンツブロックの挿入、更新、移動、削除
- **データの検索**: ナレッジベースに対する SQL クエリの実行
- **アセットのアップロード**: 画像やその他のメディアファイルの管理
- **コンテンツのエクスポート**: ノートをさまざまな形式で変換・エクスポート

## 目次

* [API 仕様](#API-仕様)
    * [パラメータと戻り値](#パラメータと戻り値)
    * [認証](#認証)
* [ノートブック](#ノートブック)
    * [ノートブック一覧](#ノートブック一覧)
    * [ノートブックを開く](#ノートブックを開く)
    * [ノートブックを閉じる](#ノートブックを閉じる)
    * [ノートブック名変更](#ノートブック名変更)
    * [ノートブック作成](#ノートブック作成)
    * [ノートブック削除](#ノートブック削除)
    * [ノートブック設定取得](#ノートブック設定取得)
    * [ノートブック設定保存](#ノートブック設定保存)
* [ドキュメント](#ドキュメント)
    * [Markdown でドキュメント作成](#Markdownでドキュメント作成)
    * [ドキュメント名変更](#ドキュメント名変更)
    * [ドキュメント削除](#ドキュメント削除)
    * [ドキュメント移動](#ドキュメント移動)
    * [パスユーティリティ](#パスユーティリティ)
        * [パスに基づく人間が読めるパス取得](#パスに基づく人間が読めるパス取得)
        * [ID に基づく人間が読めるパス取得](#IDに基づく人間が読めるパス取得)
        * [ID に基づくストレージパス取得](#IDに基づくストレージパス取得)
        * [人間が読めるパスに基づく ID 取得](#人間が読めるパスに基づくID取得)
* [アセット](#アセット)
    * [アセットアップロード](#アセットアップロード)
* [ブロック](#ブロック)
    * [ブロック挿入](#ブロック挿入)
    * [ブロック前置](#ブロック前置)
    * [ブロック後置](#ブロック後置)
    * [ブロック更新](#ブロック更新)
    * [ブロック削除](#ブロック削除)
    * [ブロック移動](#ブロック移動)
    * [ブロック折りたたみ](#ブロック折りたたみ)
    * [ブロック展開](#ブロック展開)
    * [ブロック kramdown 取得](#ブロックkramdown取得)
    * [子ブロック取得](#子ブロック取得)
    * [ブロック参照転送](#ブロック参照転送)
* [属性](#属性)
    * [ブロック属性設定](#ブロック属性設定)
    * [ブロック属性取得](#ブロック属性取得)
* [SQL](#SQL)
    * [SQL クエリ実行](#SQLクエリ実行)
    * [トランザクションフラッシュ](#トランザクションフラッシュ)
* [テンプレート](#テンプレート)
    * [テンプレートレンダリング](#テンプレートレンダリング)
    * [Sprig レンダリング](#Sprigレンダリング)
* [ファイル](#ファイル)
    * [ファイル取得](#ファイル取得)
    * [ファイル配置](#ファイル配置)
    * [ファイル削除](#ファイル削除)
    * [ファイル名変更](#ファイル名変更)
    * [ファイル一覧](#ファイル一覧)
* [エクスポート](#エクスポート)
    * [Markdown エクスポート](#Markdownエクスポート)
    * [ファイルとフォルダエクスポート](#ファイルとフォルダエクスポート)
* [変換](#変換)
    * [Pandoc](#Pandoc)
* [通知](#通知)
    * [メッセージプッシュ](#メッセージプッシュ)
    * [エラーメッセージプッシュ](#エラーメッセージプッシュ)
* [ネットワーク](#ネットワーク)
    * [フォワードプロキシ](#フォワードプロキシ)
* [システム](#システム)
    * [起動進捗取得](#起動進捗取得)
    * [システムバージョン取得](#システムバージョン取得)
    * [システム現在時刻取得](#システム現在時刻取得)

---

## API 仕様

### エンドポイント設定

SiYuan API はローカル HTTP サーバー上で動作します：

- **ベース URL**: `http://127.0.0.1:6806`
- **プロトコル**: HTTP POST のみ
- **Content-Type**: `application/json`
- **ポート**: 6806（デフォルト、設定で変更可能）

### リクエスト形式

すべての API リクエストは、リクエストボディに JSON ペイロードを含む HTTP POST として送信する必要があります：

```bash
curl -X POST \
  http://127.0.0.1:6806/api/notebook/lsNotebooks \
  -H "Content-Type: application/json" \
  -H "Authorization: Token your_api_token_here" \
  -d '{}'
```

### レスポンス形式

すべての API レスポンスは一貫した構造に従います：

```json
{
  "code": 0,
  "msg": "",
  "data": {}
}
```

**レスポンスフィールド：**
- **`code`** (整数): ステータスコード
  - `0`: 成功
  - 非ゼロ: エラーが発生
- **`msg`** (文字列): メッセージ
  - 成功時は空文字列
  - 失敗時はエラー説明
- **`data`** (オブジェクト/配列/null): レスポンスペイロード
  - エンドポイントによって異なる: `{}`、`[]`、または `null`

### 認証

**API トークンの取得方法：**
1. SiYuan アプリケーションを開く
2. <kbd>設定</kbd> → <kbd>について</kbd> に移動
3. API トークンをコピー

**トークンの使用：**
すべてのリクエストに認証ヘッダーを追加：
```
Authorization: Token your_api_token_here
```

**認証を含む例：**
```javascript
const response = await fetch('http://127.0.0.1:6806/api/notebook/lsNotebooks', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Token your_api_token_here'
  },
  body: JSON.stringify({})
});
```

### エラーハンドリング

エラーが発生した場合、レスポンスには以下が含まれます：
- `code`: 非ゼロのエラーコード
- `msg`: 説明的なエラーメッセージ
- `data`: 通常は `null` または空

エラーレスポンスの例：
```json
{
  "code": -1,
  "msg": "ノートブックが見つかりません",
  "data": null
}
```

---

## ノートブック

ノートブックは、SiYuan でドキュメントを整理するためのトップレベルのコンテナです。各ノートブックには複数のドキュメントとサブディレクトリを含めることができ、ナレッジベースに階層構造を提供します。

### ノートブック一覧

ワークスペース内のすべての利用可能なノートブックを、現在のステータス（開く/閉じる）を含めて取得します。

**エンドポイント:** `/api/notebook/lsNotebooks`

**パラメータ:** なし

**リクエスト例:**
```bash
curl -X POST http://127.0.0.1:6806/api/notebook/lsNotebooks \
  -H "Content-Type: application/json" \
  -H "Authorization: Token your_token" \
  -d '{}'
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "notebooks": [
      {
        "id": "20210817205410-2kvfpfn", 
        "name": "テストノートブック",
        "icon": "1f41b",
        "sort": 0,
        "closed": false
      },
      {
        "id": "20210808180117-czj9bvb",
        "name": "SiYuan ユーザーガイド",
        "icon": "1f4d4",
        "sort": 1,
        "closed": false
      }
    ]
  }
}
```

**レスポンスフィールド:**
- `id`: 一意のノートブック識別子（タイムスタンプベース）
- `name`: ユーザー定義のノートブック名
- `icon`: ノートブックアイコンの Unicode 絵文字コード
- `sort`: 表示順序インデックス
- `closed`: ノートブックが現在閉じられているかどうか

**使用例:**
- 利用可能なノートブックを表示するダッシュボードアプリケーション
- すべてのノートブックを処理する必要があるバックアップスクリプト
- ワークスペース概要ツール

### ノートブックを開く

閉じられたノートブックを開き、その内容の読み取りと編集を可能にします。

**エンドポイント:** `/api/notebook/openNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (文字列): 開くノートブックの一意 ID

**例:**
```javascript
const openNotebook = async (notebookId) => {
  const response = await fetch('http://127.0.0.1:6806/api/notebook/openNotebook', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Token your_token'
    },
    body: JSON.stringify({
      notebook: notebookId
    })
  });
  return response.json();
};
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**使用例:**
- 自動ワークスペース復元
- コンテキスト感応型ノートブック有効化
- 特定のノートブックを開く必要があるバッチ操作

### ノートブックを閉じる

開いているノートブックを閉じ、ノートブックのデータを保持しながらリソースを解放します。

**エンドポイント:** `/api/notebook/closeNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (文字列): 閉じるノートブックの一意 ID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**使用例:**
- 大規模なワークスペースのメモリ最適化
- プロジェクトベースのワークフロー管理
- 自動クリーンアップ処理

### ノートブック名変更

既存のノートブックの表示名を変更します。

**エンドポイント:** `/api/notebook/renameNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "name": "ノートブックの新しい名前"
}
```

- `notebook` (文字列): ノートブック ID
- `name` (文字列): ノートブックの新しい名前

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**使用例:**
- ワークスペースの再編成
- プロジェクトのリブランディング
- バッチ名前変更操作

### ノートブック作成

ワークスペースに新しい空のノートブックを作成します。

**エンドポイント:** `/api/notebook/createNotebook`

**パラメータ:**
```json
{
  "name": "ノートブック名"
}
```

- `name` (文字列): 新しいノートブックの名前

**例:**
```python
import requests

def create_notebook(name, token):
    url = "http://127.0.0.1:6806/api/notebook/createNotebook"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Token {token}"
    }
    payload = {"name": name}
    
    response = requests.post(url, json=payload, headers=headers)
    return response.json()

# 使用方法
result = create_notebook("私の新しいプロジェクト", "your_token_here")
print(f"作成されたノートブック ID: {result['data']['notebook']['id']}")
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "notebook": {
      "id": "20220126215949-r1wvoch",
      "name": "ノートブック名",
      "icon": "",
      "sort": 0,
      "closed": false
    }
  }
}
```

**使用例:**
- プロジェクト初期化スクリプト
- テンプレートベースのノートブック作成
- 自動ワークスペースセットアップ

### ノートブック削除

ノートブックとそのすべての内容を永続的に削除します。⚠️ **この操作は元に戻せません。**

**エンドポイント:** `/api/notebook/removeNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (文字列): 削除するノートブックの ID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**⚠️ 警告:** この操作はノートブック内のすべてのデータを永続的に削除します。削除前に重要なデータは必ずバックアップしてください。

**使用例:**
- 一時的なノートブックのクリーンアップスクリプト
- プロジェクトアーカイブワークフロー
- ワークスペースメンテナンス

### ノートブック設定取得

特定のノートブックの設定を取得します。

**エンドポイント:** `/api/notebook/getNotebookConf`

**パラメータ:**
```json
{
  "notebook": "20210817205410-2kvfpfn"
}
```

- `notebook` (文字列): ノートブック ID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "box": "20210817205410-2kvfpfn",
    "conf": {
      "name": "テストノートブック",
      "closed": false,
      "refCreateSavePath": "",
      "createDocNameTemplate": "",
      "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
      "dailyNoteTemplatePath": ""
    },
    "name": "テストノートブック"
  }
}
```

**設定フィールド:**
- `name`: ノートブック表示名
- `closed`: ノートブックが閉じられているかどうか
- `refCreateSavePath`: 参照ドキュメント作成のデフォルトパス
- `createDocNameTemplate`: 新しいドキュメント名のテンプレート
- `dailyNoteSavePath`: デイリーノートのパステンプレート（日付フォーマット対応）
- `dailyNoteTemplatePath`: デイリーノートに使用されるテンプレートへのパス

**使用例:**
- ノートブック設定のバックアップと復元
- インスタンス間での設定同期
- 設定管理ツールの構築

### ノートブック設定保存

特定のノートブックの設定を更新します。

**エンドポイント:** `/api/notebook/setNotebookConf`

**パラメータ:**
```json
{
  "notebook": "20210817205410-2kvfpfn",
  "conf": {
    "name": "テストノートブック",
    "closed": false,
    "refCreateSavePath": "",
    "createDocNameTemplate": "",
    "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
    "dailyNoteTemplatePath": ""
  }
}
```

- `notebook` (文字列): ノートブック ID
- `conf` (オブジェクト): 更新する設定を含む設定オブジェクト

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "name": "テストノートブック",
    "closed": false,
    "refCreateSavePath": "",
    "createDocNameTemplate": "",
    "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
    "dailyNoteTemplatePath": ""
  }
}
```

**例 - デイリーノートの設定:**
```javascript
const setupDailyNotes = async (notebookId, token) => {
  const conf = {
    name: "私のデイリージャーナル",
    closed: false,
    refCreateSavePath: "/references",
    createDocNameTemplate: "{{now | date \"2006-01-02\"}} - {{.title}}",
    dailyNoteSavePath: "/daily/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
    dailyNoteTemplatePath: "/templates/daily-template"
  };
  
  const response = await fetch('http://127.0.0.1:6806/api/notebook/setNotebookConf', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Token ${token}`
    },
    body: JSON.stringify({
      notebook: notebookId,
      conf: conf
    })
  });
  
  return response.json();
};
```

**使用例:**
- 自動ワークスペースセットアップ
- テンプレートベースのノートブック設定
- バルク設定更新

---

## ドキュメント

ドキュメントは、ノートブック内の個別のノートまたはファイルです。テキスト、画像、表、複雑なブロック構造など、さまざまなタイプのコンテンツを含めることができます。ドキュメント API を使用して、ナレッジベースのコンテンツを作成、管理、整理できます。

### Markdown でドキュメント作成

Markdown コンテンツから新しいドキュメントを作成します。既存のノートをインポートしたり、プログラムでドキュメントを作成したりする場合に便利です。

**エンドポイント:** `/api/filetree/createDocWithMd`

**パラメータ:**
```json
{
  "notebook": "20210817205410-2kvfpfn",
  "path": "/foo/bar",
  "markdown": "# Hello World\n\nこれは私の最初のドキュメントです。"
}
```

- `notebook` (文字列): 対象のノートブック ID
- `path` (文字列): ノートブック内のドキュメントパス、`/` で始まり、レベルを `/` で区切る必要があります（データベース `hpath` フィールドに対応）
- `markdown` (文字列): ドキュメントの GFM（GitHub フレーバー Markdown）コンテンツ

**例:**
```python
def create_markdown_document(notebook_id, path, content, token):
    """markdown コンテンツから新しいドキュメントを作成"""
    import requests
    
    url = "http://127.0.0.1:6806/api/filetree/createDocWithMd"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Token {token}"
    }
    
    payload = {
        "notebook": notebook_id,
        "path": path,
        "markdown": content
    }
    
    response = requests.post(url, json=payload, headers=headers)
    return response.json()

# 使用例
markdown_content = """# プロジェクト概要

## 目標
- API ドキュメントの完成
- 日本語翻訳の追加
- ユーザーエクスペリエンスの改善

## タイムライン
- 第1週: ドキュメント拡充
- 第2週: 翻訳作業
- 第3週: レビューとテスト
"""

result = create_markdown_document(
    "20210817205410-2kvfpfn", 
    "/projects/api-documentation", 
    markdown_content,
    "your_token_here"
)
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": "20210914223645-oj2vnx2"
}
```

- `data`: 作成されたドキュメントの ID

**重要な注意事項:**
- 同じ `path` でこのエンドポイントを繰り返し呼び出しても、既存のドキュメントは上書きされません
- パスには `.sy` ファイル拡張子を含めないでください
- Markdown コンテンツは SiYuan の内部ブロック構造に変換されます

**使用例:**
- 他のノート作成アプリケーションからのコンテンツインポート
- 自動コンテンツ生成
- テンプレートベースのドキュメント作成
- 外部ソースからのバッチドキュメント作成

### ドキュメント名変更

既存のドキュメントのタイトルを変更します。ドキュメントパスまたはドキュメント ID で名前を変更できます。

**方法 1: パスで名前変更**

**エンドポイント:** `/api/filetree/renameDoc`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210902210113-0avi12f.sy",
  "title": "新しいドキュメントタイトル"
}
```

- `notebook` (文字列): ドキュメントを含むノートブック ID
- `path` (文字列): `.sy` 拡張子を含む完全なドキュメントパス
- `title` (文字列): ドキュメントの新しいタイトル

**方法 2: ID で名前変更**

**エンドポイント:** `/api/filetree/renameDocByID`

**パラメータ:**
```json
{
  "id": "20210902210113-0avi12f",
  "title": "新しいドキュメントタイトル"
}
```

- `id` (文字列): ドキュメント ID
- `title` (文字列): ドキュメントの新しいタイトル

**レスポンス（両方の方法）:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**例:**
```javascript
const renameDocument = async (documentId, newTitle, token) => {
  const response = await fetch('http://127.0.0.1:6806/api/filetree/renameDocByID', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Token ${token}`
    },
    body: JSON.stringify({
      id: documentId,
      title: newTitle
    })
  });
  
  return response.json();
};

// 使用方法
await renameDocument("20210902210113-0avi12f", "更新されたドキュメントタイトル", "your_token");
```

**使用例:**
- ドキュメントの整理と再構築
- バッチ名前変更操作
- コンテンツ管理ワークフロー
- テンプレートベースの命名規則

### ドキュメント削除

ドキュメントとそのすべてのコンテンツを永続的に削除します。⚠️ **この操作は元に戻せません。**

**方法 1: パスで削除**

**エンドポイント:** `/api/filetree/removeDoc`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210902210113-0avi12f.sy"
}
```

- `notebook` (文字列): ドキュメントを含むノートブック ID
- `path` (文字列): `.sy` 拡張子を含む完全なドキュメントパス

**方法 2: ID で削除**

**エンドポイント:** `/api/filetree/removeDocByID`

**パラメータ:**
```json
{
  "id": "20210902210113-0avi12f"
}
```

- `id` (文字列): 削除するドキュメント ID

**レスポンス（両方の方法）:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**⚠️ 警告:** この操作はドキュメントとそのすべてのコンテンツを永続的に削除します。削除前に重要なデータは必ずバックアップしてください。

**例:**
```bash
# ID でドキュメントを削除
curl -X POST http://127.0.0.1:6806/api/filetree/removeDocByID \
  -H "Content-Type: application/json" \
  -H "Authorization: Token your_token" \
  -d '{"id": "20210902210113-0avi12f"}'
```

**使用例:**
- 一時的なドキュメントのクリーンアップスクリプト
- コンテンツ管理ワークフロー
- バッチ削除操作
- ドキュメントライフサイクル管理

### ドキュメント移動

同じノートブック内の異なる場所または異なるノートブック間でドキュメントを移動します。

**方法 1: パスで移動**

**エンドポイント:** `/api/filetree/moveDocs`

**パラメータ:**
```json
{
  "fromPaths": ["/20210917220056-yxtyl7i.sy"],
  "toNotebook": "20210817205410-2kvfpfn",
  "toPath": "/"
}
```

- `fromPaths` (配列): ソースドキュメントパスの配列
- `toNotebook` (文字列): 対象のノートブック ID
- `toPath` (文字列): ノートブック内の対象パス

**方法 2: ID で移動**

**エンドポイント:** `/api/filetree/moveDocsByID`

**パラメータ:**
```json
{
  "fromIDs": ["20210917220056-yxtyl7i"],
  "toID": "20210817205410-2kvfpfn"
}
```

- `fromIDs` (配列): ソースドキュメント ID の配列
- `toID` (文字列): 対象の親ドキュメント ID またはノートブック ID

**レスポンス（両方の方法）:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**例 - ドキュメントをフォルダに整理:**
```javascript
const organizeDocuments = async (token) => {
  // 複数のドキュメントをプロジェクトフォルダに移動
  const response = await fetch('http://127.0.0.1:6806/api/filetree/moveDocsByID', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Token ${token}`
    },
    body: JSON.stringify({
      fromIDs: [
        "20210917220056-yxtyl7i",
        "20210918110023-abc123f",
        "20210919140055-def456g"
      ],
      toID: "20210920000000-project1"  // 親フォルダ ID
    })
  });
  
  return response.json();
};
```

**使用例:**
- ドキュメントの整理と再構築
- プロジェクトベースのコンテンツ管理
- ノートブック間のドキュメント移行
- 自動コンテンツ分類

### パスユーティリティ

SiYuan は、異なるパス表現間の変換や、さまざまな識別子によるドキュメントの検索を行う複数のユーティリティ関数を提供します。

### パスに基づく人間が読めるパス取得

ストレージパスを人間が読める形式のパスに変換します。

**エンドポイント:** `/api/filetree/getHPathByPath`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210917220500-sz588nq/20210917220056-yxtyl7i.sy"
}
```

- `notebook` (文字列): ノートブック ID
- `path` (文字列): ドキュメントストレージパス

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": "/foo/bar"
}
```

- `data` (文字列): 人間が読めるパス

**使用例:**
- ユーザーフレンドリーなナビゲーションインターフェースの作成
- パンくずナビゲーションの生成
- 読みやすいパスでのドキュメントリンク

### ID に基づく人間が読めるパス取得

ブロック ID を使用してドキュメントの人間が読めるパスを取得します。

**エンドポイント:** `/api/filetree/getHPathByID`

**パラメータ:**
```json
{
  "id": "20210917220056-yxtyl7i"
}
```

- `id` (文字列): ドキュメントのブロック ID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": "/foo/bar"
}
```

**例:**
```javascript
const getDocumentPath = async (blockId, token) => {
  const response = await fetch('http://127.0.0.1:6806/api/filetree/getHPathByID', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Token ${token}`
    },
    body: JSON.stringify({ id: blockId })
  });
  
  const result = await response.json();
  return result.data; // 人間が読めるパスを返す
};
```

**使用例:**
- ナビゲーションシステムの構築
- ドキュメント参照の作成
- 検索結果表示の実装

### ID に基づくストレージパス取得

ブロック ID に基づいて、ドキュメントの実際のストレージパスとノートブック ID を取得します。

**エンドポイント:** `/api/filetree/getPathByID`

**パラメータ:**
```json
{
  "id": "20210808180320-fqgskfj"
}
```

- `id` (文字列): ドキュメントのブロック ID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "notebook": "20210808180117-czj9bvb",
    "path": "/20200812220555-lj3enxa/20210808180320-fqgskfj.sy"
  }
}
```

- `notebook` (文字列): ドキュメントを含むノートブック ID
- `path` (文字列): ノートブック内のストレージパス

**使用例:**
- ファイルシステム操作
- バックアップと同期ツール
- 低レベルドキュメント管理

### 人間が読めるパスに基づく ID 取得

指定された人間が読めるパスに一致するすべてのドキュメント ID を取得します。

**エンドポイント:** `/api/filetree/getIDsByHPath`

**パラメータ:**
```json
{
  "path": "/foo/bar",
  "notebook": "20210808180117-czj9bvb"
}
```

- `path` (文字列): 検索する人間が読めるパス
- `notebook` (文字列): 検索対象のノートブック ID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    "20200813004931-q4cu8na"
  ]
}
```

- `data` (配列): パスに一致するドキュメント ID の配列

**例 - パスでドキュメントを検索:**
```python
def find_documents_by_path(notebook_id, search_path, token):
    """人間が読めるパスに一致するすべてのドキュメントを検索"""
    import requests
    
    url = "http://127.0.0.1:6806/api/filetree/getIDsByHPath"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Token {token}"
    }
    
    payload = {
        "notebook": notebook_id,
        "path": search_path
    }
    
    response = requests.post(url, json=payload, headers=headers)
    result = response.json()
    
    if result["code"] == 0:
        return result["data"]  # ドキュメント ID のリスト
    else:
        print(f"エラー: {result['msg']}")
        return []

# 使用方法
doc_ids = find_documents_by_path(
    "20210808180117-czj9bvb", 
    "/projects/api-documentation", 
    "your_token"
)
print(f"{len(doc_ids)} 件のドキュメントが見つかりました")
```

**使用例:**
- 検索機能の実装
- ナビゲーションシステムの構築
- ドキュメント関係のマッピング

---

## アセット

SiYuan のアセットには、ノートを豊かにする画像、ドキュメント、ビデオ、その他のメディアファイルが含まれます。アセット API を使用して、これらのファイルをプログラムでアップロードおよび管理できます。

### アセットアップロード

SiYuan アセットディレクトリに 1 つ以上のファイルをアップロードします。ファイルは競合を防ぐために一意の識別子で自動的に名前が変更されます。

**エンドポイント:** `/api/asset/upload`

**メソッド:** multipart/form-data を使用した HTTP POST

**パラメータ:**
- `assetsDirPath` (文字列): データフォルダに対する相対的な対象ディレクトリパス
  - `"/assets/"`: workspace/data/assets/ フォルダ（推奨）
  - `"/assets/sub/"`: workspace/data/assets/sub/ フォルダ
- `file[]` (ファイル配列): アップロードする 1 つ以上のファイル

**ディレクトリパスの例:**
- `"/assets/"` → `workspace/data/assets/` フォルダに保存
- `"/assets/images/"` → `workspace/data/assets/images/` フォルダに保存  
- `"/assets/documents/"` → `workspace/data/assets/documents/` フォルダに保存

**⚠️ 重要:** サブディレクトリには副作用があるため、メインのアセットフォルダ（`"/assets/"`）の使用を推奨します。詳細については、ユーザーガイドのアセットの章を参照してください。

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "errFiles": [""],
    "succMap": {
      "foo.png": "assets/foo-20210719092549-9j5y79r.png",
      "document.pdf": "assets/document-20210719092550-abc123d.pdf"
    }
  }
}
```

**レスポンスフィールド:**
- `errFiles` (配列): アップロードに失敗したファイル名のリスト
- `succMap` (オブジェクト): 正常にアップロードされたファイルのマッピング
  - キー: 元のファイル名
  - 値: 一意の識別子付きの新しいアセットパス

**例 - 複数ファイルのアップロード:**
```javascript
const uploadAssets = async (files, token) => {
  const formData = new FormData();
  
  // ディレクトリパスを追加
  formData.append('assetsDirPath', '/assets/');
  
  // ファイルを追加
  files.forEach(file => {
    formData.append('file[]', file);
  });
  
  const response = await fetch('http://127.0.0.1:6806/api/asset/upload', {
    method: 'POST',
    headers: {
      'Authorization': `Token ${token}`
      // multipart/form-data の Content-Type は設定しない - ブラウザが設定する
    },
    body: formData
  });
  
  return response.json();
};

// ファイル入力での使用方法
document.getElementById('fileInput').addEventListener('change', async (event) => {
  const files = Array.from(event.target.files);
  const result = await uploadAssets(files, 'your_token_here');
  
  console.log('アップロード結果:', result.data.succMap);
  console.log('失敗したアップロード:', result.data.errFiles);
});
```

**例 - curl を使用したアップロード:**
```bash
curl -X POST http://127.0.0.1:6806/api/asset/upload \
  -H "Authorization: Token your_token_here" \
  -F "assetsDirPath=/assets/" \
  -F "file[]=@/path/to/image.png" \
  -F "file[]=@/path/to/document.pdf"
```

**例 - Python でのアップロード:**
```python
import requests

def upload_assets(file_paths, token, assets_dir="/assets/"):
    """複数のファイルを SiYuan アセットにアップロード"""
    
    url = "http://127.0.0.1:6806/api/asset/upload"
    headers = {"Authorization": f"Token {token}"}
    
    files = []
    data = {"assetsDirPath": assets_dir}
    
    # アップロード用のファイルを準備
    for file_path in file_paths:
        with open(file_path, 'rb') as f:
            files.append(('file[]', (f.name, f.read())))
    
    response = requests.post(url, headers=headers, data=data, files=files)
    return response.json()

# 使用方法
result = upload_assets([
    "/home/user/screenshot.png",
    "/home/user/report.pdf"
], "your_token_here")

for original, new_path in result['data']['succMap'].items():
    print(f"アップロード完了 {original} → {new_path}")
```

**使用例:**
- バッチメディアファイルアップロード
- ドキュメント管理システム
- コンテンツ移行ツール
- 自動スクリーンショット統合
- リッチマルチメディアノートの構築

**アセットリンクの統合:**
アップロード後、返されたパスを Markdown コンテンツで使用します：
```markdown
![説明](assets/foo-20210719092549-9j5y79r.png)
[PDF をダウンロード](assets/document-20210719092550-abc123d.pdf)
```

---

この続きで、ブロック、属性、SQL、その他のセクションも同様に詳細な日本語説明と例を追加していきます。現在の進捗をコミットしましょう。
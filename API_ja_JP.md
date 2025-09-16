# SiYuan API ドキュメント

SiYuanノートアプリケーションAPIの包括的ガイドです。このAPIを使用して、プログラム的にSiYuanと連携し、ノートブック、ドキュメント、ブロックを管理し、さまざまな操作を実行できます。

**言語オプション:**
- [English](API.md)
- [中文 (Chinese)](API_zh_CN.md)

---

## 概要

SiYuan APIは、知識ベースとやり取りするためのRESTfulインターフェースを提供します。以下のことが可能です：

- **ノートブック管理**: ノートブックの作成、開く、閉じる、名前変更、削除
- **ドキュメント処理**: Markdownからドキュメントを作成、ファイル構造の整理
- **ブロック操作**: コンテンツブロックの挿入、更新、移動、削除
- **データクエリ**: 知識ベースに対するSQLクエリの実行
- **アセットアップロード**: 画像やその他のメディアファイルの管理
- **コンテンツエクスポート**: ノートを様々な形式で変換・エクスポート

## 目次

* [仕様](#仕様)
    * [パラメータと戻り値](#パラメータと戻り値)
    * [認証](#認証)
* [ノートブック](#ノートブック)
    * [ノートブック一覧](#ノートブック一覧)
    * [ノートブックを開く](#ノートブックを開く)
    * [ノートブックを閉じる](#ノートブックを閉じる)
    * [ノートブック名を変更](#ノートブック名を変更)
    * [ノートブックを作成](#ノートブックを作成)
    * [ノートブックを削除](#ノートブックを削除)
    * [ノートブック設定を取得](#ノートブック設定を取得)
    * [ノートブック設定を保存](#ノートブック設定を保存)
* [ドキュメント](#ドキュメント)
    * [Markdownでドキュメントを作成](#Markdownでドキュメントを作成)
    * [ドキュメント名を変更](#ドキュメント名を変更)
    * [ドキュメントを削除](#ドキュメントを削除)
    * [ドキュメントを移動](#ドキュメントを移動)
    * [パスに基づいて人間が読めるパスを取得](#パスに基づいて人間が読めるパスを取得)
    * [IDに基づいて人間が読めるパスを取得](#IDに基づいて人間が読めるパスを取得)
    * [IDに基づいてストレージパスを取得](#IDに基づいてストレージパスを取得)
    * [人間が読めるパスに基づいてIDを取得](#人間が読めるパスに基づいてIDを取得)
* [アセット](#アセット)
    * [アセットのアップロード](#アセットのアップロード)
* [ブロック](#ブロック)
    * [ブロックの挿入](#ブロックの挿入)
    * [ブロックの前置追加](#ブロックの前置追加)
    * [ブロックの後置追加](#ブロックの後置追加)
    * [ブロックの更新](#ブロックの更新)
    * [ブロックの削除](#ブロックの削除)
    * [ブロックの移動](#ブロックの移動)
    * [ブロックの折りたたみ](#ブロックの折りたたみ)
    * [ブロックの展開](#ブロックの展開)
    * [ブロックのkramdownを取得](#ブロックのkramdownを取得)
    * [子ブロックを取得](#子ブロックを取得)
    * [ブロック参照の転送](#ブロック参照の転送)
* [属性](#属性)
    * [ブロック属性の設定](#ブロック属性の設定)
    * [ブロック属性の取得](#ブロック属性の取得)
* [SQL](#SQL)
    * [SQLクエリの実行](#SQLクエリの実行)
    * [トランザクションのフラッシュ](#トランザクションのフラッシュ)
* [テンプレート](#テンプレート)
    * [テンプレートのレンダリング](#テンプレートのレンダリング)
    * [Sprigのレンダリング](#Sprigのレンダリング)
* [ファイル](#ファイル)
    * [ファイルの取得](#ファイルの取得)
    * [ファイルの書き込み](#ファイルの書き込み)
    * [ファイルの削除](#ファイルの削除)
    * [ファイル名の変更](#ファイル名の変更)
    * [ファイル一覧](#ファイル一覧)
* [エクスポート](#エクスポート)
    * [Markdownエクスポート](#Markdownエクスポート)
    * [ファイルとフォルダのエクスポート](#ファイルとフォルダのエクスポート)
* [変換](#変換)
    * [Pandoc](#Pandoc)
* [通知](#通知)
    * [メッセージのプッシュ](#メッセージのプッシュ)
    * [エラーメッセージのプッシュ](#エラーメッセージのプッシュ)
* [ネットワーク](#ネットワーク)
    * [フォワードプロキシ](#フォワードプロキシ)
* [システム](#システム)
    * [起動進行状況の取得](#起動進行状況の取得)
    * [システムバージョンの取得](#システムバージョンの取得)
    * [システム現在時刻の取得](#システム現在時刻の取得)

---

## API仕様

### エンドポイント設定

SiYuan APIはローカルHTTPサーバー上で動作します：

- **ベースURL**: `http://127.0.0.1:6806`
- **プロトコル**: HTTP POSTのみ
- **Content-Type**: `application/json`
- **ポート**: 6806（デフォルト、設定で変更可能）

### リクエスト形式

すべてのAPIリクエストは、リクエストボディにJSONペイロードを含むHTTP POSTとして送信する必要があります：

```bash
curl -X POST \
  http://127.0.0.1:6806/api/notebook/lsNotebooks \
  -H "Content-Type: application/json" \
  -H "Authorization: Token your_api_token_here" \
  -d '{}'
```

### レスポンス形式

すべてのAPIレスポンスは一貫した構造に従います：

```json
{
  "code": 0,
  "msg": "",
  "data": {}
}
```

**レスポンスフィールド:**
- **`code`** (整数): ステータスコード
  - `0`: 成功
  - 非ゼロ: エラーが発生
- **`msg`** (文字列): メッセージ
  - 成功時は空文字列
  - 失敗時はエラーの説明
- **`data`** (オブジェクト/配列/null): レスポンスペイロード
  - エンドポイントにより異なる: `{}`、`[]`、または`null`

### 認証

**APIトークンの取得方法:**
1. SiYuanアプリケーションを開く
2. <kbd>設定</kbd> → <kbd>About</kbd>に移動
3. APIトークンをコピー

**トークンの使用:**
すべてのリクエストに認証ヘッダーを追加：
```
Authorization: Token your_api_token_here
```

**認証付きの例:**
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

### エラー処理

エラーが発生した場合、レスポンスには以下が含まれます：
- `code`: 非ゼロのエラーコード
- `msg`: 説明的なエラーメッセージ
- `data`: 通常`null`または空

エラーレスポンスの例：
```json
{
  "code": -1,
  "msg": "Notebook not found",
  "data": null
}
```

## ノートブック

ノートブックは、SiYuanでドキュメントを整理するためのトップレベルコンテナです。各ノートブックは複数のドキュメントとサブディレクトリを含むことができ、知識ベースに階層構造を提供します。

### ノートブック一覧

ワークスペース内のすべての利用可能なノートブックを、現在のステータス（開いている/閉じている）と共に取得します。

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
        "name": "SiYuanユーザーガイド",
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
- `icon`: ノートブックアイコンのUnicode絵文字コード
- `sort`: 表示順序インデックス
- `closed`: ノートブックが現在閉じているかどうか

**使用例:**
- 利用可能なノートブックを表示するダッシュボードアプリケーション
- すべてのノートブックを処理する必要があるバックアップスクリプト
- ワークスペース概要ツール


### ノートブックを開く

閉じているノートブックを開き、そのコンテンツを読み取りと編集ができるようにします。

**エンドポイント:** `/api/notebook/openNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (文字列): 開くノートブックの一意のID

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
- コンテキスト依存のノートブックアクティベーション
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

- `notebook` (文字列): 閉じるノートブックの一意のID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**使用例:**
- 大きなワークスペースのメモリ最適化
- プロジェクトベースのワークフロー管理
- 自動クリーンアッププロセス

### ノートブック名を変更

既存のノートブックの表示名を変更します。

**エンドポイント:** `/api/notebook/renameNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "name": "ノートブックの新しい名前"
}
```

- `notebook` (文字列): ノートブックID
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
- ワークスペースの再構成
- プロジェクトのリブランディング
- バッチ名前変更操作

### ノートブックを作成

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

# 使用法
result = create_notebook("新しいプロジェクト", "your_token_here")
print(f"ノートブックが作成されました ID: {result['data']['notebook']['id']}")
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

### ノートブックを削除

ノートブックとそのすべてのコンテンツを完全に削除します。⚠️ **この操作は元に戻せません。**

**エンドポイント:** `/api/notebook/removeNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (文字列): 削除するノートブックのID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**⚠️ 警告:** この操作はノートブック内のすべてのデータを完全に削除します。削除前に重要なデータは必ずバックアップしてください。

**使用例:**
- 一時的なノートブックのクリーンアップスクリプト
- プロジェクトアーカイブワークフロー
- ワークスペースメンテナンス

### ノートブック設定を取得

特定のノートブックの設定を取得します。

**エンドポイント:** `/api/notebook/getNotebookConf`

**パラメータ:**
```json
{
  "notebook": "20210817205410-2kvfpfn"
}
```

- `notebook` (文字列): ノートブックID

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
- `closed`: ノートブックが閉じているかどうか
- `refCreateSavePath`: 参照ドキュメント作成のデフォルトパス
- `createDocNameTemplate`: 新しいドキュメント名のテンプレート
- `dailyNoteSavePath`: デイリーノートのパステンプレート（日付フォーマット対応）
- `dailyNoteTemplatePath`: デイリーノートに使用するテンプレートのパス

**使用例:**
- ノートブック設定のバックアップと復元
- インスタンス間での設定同期
- 設定管理ツールの構築

### ノートブック設定を保存

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

- `notebook` (文字列): ノートブックID
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
- 一括設定更新

---

## ドキュメント

ドキュメントは、ノートブック内の個々のノートやファイルです。テキスト、画像、表、複雑なブロック構造など、さまざまなタイプのコンテンツを含むことができます。Documents APIを使用すると、知識ベースのコンテンツを作成、管理、整理できます。

### Markdownでドキュメントを作成

Markdownコンテンツから新しいドキュメントを作成します。既存のノートをインポートしたり、プログラム的にドキュメントを作成する際に便利です。

**エンドポイント:** `/api/filetree/createDocWithMd`

**パラメータ:**
```json
{
  "notebook": "20210817205410-2kvfpfn",
  "path": "/foo/bar",
  "markdown": "# Hello World\n\nこれは私の最初のドキュメントです。"
}
```

- `notebook` (文字列): ターゲットノートブックID
- `path` (文字列): ノートブック内のドキュメントパス、`/`で始まり、レベルを`/`で区切る必要があります（データベースの`hpath`フィールドに対応）
- `markdown` (文字列): ドキュメントのGFM（GitHub Flavored Markdown）コンテンツ

**例:**
```python
def create_markdown_document(notebook_id, path, content, token):
    """markdownコンテンツから新しいドキュメントを作成"""
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
- APIドキュメントの完成
- 日本語翻訳の追加
- ユーザーエクスペリエンスの向上

## タイムライン
- 第1週: ドキュメント強化
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

- `data`: 作成されたドキュメントのID

**重要な注意事項:**
- 同じ`path`でこのエンドポイントを繰り返し呼び出しても、既存のドキュメントは上書きされません
- パスには`.sy`ファイル拡張子を含めないでください
- Markdownコンテンツは、SiYuanの内部ブロック構造に変換されます

**使用例:**
- 他のノート取りアプリケーションからのコンテンツインポート
- 自動コンテンツ生成
- テンプレートベースのドキュメント作成
- 外部ソースからのバッチドキュメント作成


### ドキュメント名を変更

既存のドキュメントのタイトルを変更します。ドキュメントパスまたはドキュメントIDで名前を変更できます。

**方法1: パスで名前変更**

**エンドポイント:** `/api/filetree/renameDoc`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210902210113-0avi12f.sy",
  "title": "新しいドキュメントタイトル"
}
```

- `notebook` (文字列): ドキュメントを含むノートブックID
- `path` (文字列): `.sy`拡張子を含む完全なドキュメントパス
- `title` (文字列): ドキュメントの新しいタイトル

**方法2: IDで名前変更**

**エンドポイント:** `/api/filetree/renameDocByID`

**パラメータ:**
```json
{
  "id": "20210902210113-0avi12f",
  "title": "新しいドキュメントタイトル"
}
```

- `id` (文字列): ドキュメントID
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

// 使用法
await renameDocument("20210902210113-0avi12f", "更新されたドキュメントタイトル", "your_token");
```

### ドキュメントを削除

ドキュメントとそのすべてのコンテンツを完全に削除します。⚠️ **この操作は元に戻せません。**

**方法1: パスで削除**

**エンドポイント:** `/api/filetree/removeDoc`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210902210113-0avi12f.sy"
}
```

- `notebook` (文字列): ドキュメントを含むノートブックID
- `path` (文字列): `.sy`拡張子を含む完全なドキュメントパス

**方法2: IDで削除**

**エンドポイント:** `/api/filetree/removeDocByID`

**パラメータ:**
```json
{
  "id": "20210902210113-0avi12f"
}
```

- `id` (文字列): 削除するドキュメントID

**レスポンス（両方の方法）:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**⚠️ 警告:** この操作はドキュメントとそのすべてのコンテンツを完全に削除します。削除前に重要なデータは必ずバックアップしてください。

### ドキュメントを移動

同じノートブック内の異なる場所間、または異なるノートブック間でドキュメントを移動します。

**方法1: パスで移動**

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
- `toNotebook` (文字列): ターゲットノートブックID
- `toPath` (文字列): ノートブック内のターゲットパス

**方法2: IDで移動**

**エンドポイント:** `/api/filetree/moveDocsByID`

**パラメータ:**
```json
{
  "fromIDs": ["20210917220056-yxtyl7i"],
  "toID": "20210817205410-2kvfpfn"
}
```

- `fromIDs` (配列): ソースドキュメントIDの配列
- `toID` (文字列): ターゲット親ドキュメントIDまたはノートブックID

### パスに基づいて人間が読めるパスを取得

**エンドポイント:** `/api/filetree/getHPathByPath`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210917220500-sz588nq/20210917220056-yxtyl7i.sy"
}
```

- `notebook`: ノートブックID
- `path`: ドキュメントパス

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": "/foo/bar"
}
```

### IDに基づいて人間が読めるパスを取得

**エンドポイント:** `/api/filetree/getHPathByID`

**パラメータ:**
```json
{
  "id": "20210917220056-yxtyl7i"
}
```

- `id`: ブロックID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": "/foo/bar"
}
```

### IDに基づいてストレージパスを取得

**エンドポイント:** `/api/filetree/getPathByID`

**パラメータ:**
```json
{
  "id": "20210808180320-fqgskfj"
}
```

- `id`: ブロックID

**戻り値:**
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

### 人間が読めるパスに基づいてIDを取得

**エンドポイント:** `/api/filetree/getIDsByHPath`

**パラメータ:**
```json
{
  "path": "/foo/bar",
  "notebook": "20210808180117-czj9bvb"
}
```

- `path`: 人間が読めるパス
- `notebook`: ノートブックID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    "20200813004931-q4cu8na"
  ]
}
```

## アセット

### アセットのアップロード

**エンドポイント:** `/api/asset/upload`

**パラメータ:** HTTP Multipartフォーム

- `assetsDirPath`: アセットが保存されるフォルダパス（dataフォルダをルートパスとする）、例：
  - `"/assets/"`: workspace/data/assets/ フォルダ
  - `"/assets/sub/"`: workspace/data/assets/sub/ フォルダ

  通常の状況では、最初の方法を推奨します。これはワークスペースのassetsフォルダに保存されます。サブディレクトリに配置すると副作用があります。ユーザーガイドのassetsの章を参考にしてください。
- `file[]`: アップロードファイルリスト

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "errFiles": [""],
    "succMap": {
      "foo.png": "assets/foo-20210719092549-9j5y79r.png"
    }
  }
}
```

- `errFiles`: アップロード処理でエラーが発生したファイル名のリスト
- `succMap`: 正常に処理されたファイルの場合、キーはアップロード時のファイル名、値はassets/foo-id.pngで、既存のMarkdownコンテンツ内のアセットリンクアドレスをアップロードされたアドレスに置き換えるために使用されます

## ブロック

### ブロックの挿入

**エンドポイント:** `/api/block/insertBlock`

**パラメータ:**
```json
{
  "dataType": "markdown",
  "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
  "nextID": "",
  "previousID": "20211229114650-vrek5x6",
  "parentID": ""
}
```

- `dataType`: 挿入するデータタイプ、値は`markdown`または`dom`
- `data`: 挿入するデータ
- `nextID`: 次のブロックのID、挿入位置をアンカーするために使用
- `previousID`: 前のブロックのID、挿入位置をアンカーするために使用
- `parentID`: 親ブロックのID、挿入位置をアンカーするために使用

`nextID`、`previousID`、`parentID`のうち少なくとも一つの値が必要です。優先順位: `nextID` > `previousID` > `parentID`

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "doOperations": [
        {
          "action": "insert",
          "data": "<div data-node-id=\"20211230115020-g02dfx0\" data-node-index=\"1\" data-type=\"NodeParagraph\" class=\"p\"><div contenteditable=\"true\" spellcheck=\"false\">foo<strong style=\"color: var(--b3-font-color8);\">bar</strong>baz</div><div class=\"protyle-attr\" contenteditable=\"false\"></div></div>",
          "id": "20211230115020-g02dfx0",
          "parentID": "",
          "previousID": "20211229114650-vrek5x6",
          "retData": null
        }
      ],
      "undoOperations": null
    }
  ]
}
```

- `action.data`: 新しく挿入されたブロックによって生成されたDOM
- `action.id`: 新しく挿入されたブロックのID

### ブロックの前置追加

**エンドポイント:** `/api/block/prependBlock`

**パラメータ:**
```json
{
  "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
  "dataType": "markdown",
  "parentID": "20220107173950-7f9m1nb"
}
```

- `dataType`: 挿入するデータタイプ、値は`markdown`または`dom`
- `data`: 挿入するデータ
- `parentID`: 親ブロックのID、挿入位置をアンカーするために使用

### ブロックの後置追加

**エンドポイント:** `/api/block/appendBlock`

**パラメータ:**
```json
{
  "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
  "dataType": "markdown",
  "parentID": "20220107173950-7f9m1nb"
}
```

- `dataType`: 挿入するデータタイプ、値は`markdown`または`dom`
- `data`: 挿入するデータ
- `parentID`: 親ブロックのID、挿入位置をアンカーするために使用

### ブロックの更新

**エンドポイント:** `/api/block/updateBlock`

**パラメータ:**
```json
{
  "dataType": "markdown",
  "data": "foobarbaz",
  "id": "20211230161520-querkps"
}
```

- `dataType`: 更新するデータタイプ、値は`markdown`または`dom`
- `data`: 更新するデータ
- `id`: 更新するブロックのID

### ブロックの削除

**エンドポイント:** `/api/block/deleteBlock`

**パラメータ:**
```json
{
  "id": "20211230161520-querkps"
}
```

- `id`: 削除するブロックのID

### ブロックの移動

**エンドポイント:** `/api/block/moveBlock`

**パラメータ:**
```json
{
  "id": "20230406180530-3o1rqkc",
  "previousID": "20230406152734-if5kyx6",
  "parentID": "20230404183855-woe52ko"
}
```

- `id`: 移動するブロックID
- `previousID`: 前のブロックのID、挿入位置をアンカーするために使用
- `parentID`: 親ブロックのID、挿入位置をアンカーするために使用、`previousID`と`parentID`は同時に空にできません。両方が存在する場合、`previousID`が優先されます

### ブロックの折りたたみ

**エンドポイント:** `/api/block/foldBlock`

**パラメータ:**
```json
{
  "id": "20231224160424-2f5680o"
}
```

- `id`: 折りたたむブロックID

### ブロックの展開

**エンドポイント:** `/api/block/unfoldBlock`

**パラメータ:**
```json
{
  "id": "20231224160424-2f5680o"
}
```

- `id`: 展開するブロックID

### ブロックのkramdownを取得

**エンドポイント:** `/api/block/getBlockKramdown`

**パラメータ:**
```json
{
  "id": "20201225220954-dlgzk1o"
}
```

- `id`: 取得するブロックのID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "20201225220954-dlgzk1o",
    "kramdown": "* {: id=\"20201225220954-e913snx\"}新しいノートブックを作成し、ノートブック下に新しいドキュメントを作成\n  {: id=\"20210131161940-kfs31q6\"}\n* {: id=\"20201225220954-ygz217h\"}エディターで<kbd>/</kbd>を入力して機能メニューをトリガー\n  {: id=\"20210131161940-eo0riwq\"}\n* {: id=\"20201225220954-875yybt\"}((20200924101200-gss5vee \"コンテンツブロック内でのナビゲーション\"))と((20200924100906-0u4zfq3 \"ウィンドウとタブ\"))\n  {: id=\"20210131161940-b5uow2h\"}"
  }
}
```

### 子ブロックを取得

**エンドポイント:** `/api/block/getChildBlocks`

**パラメータ:**
```json
{
  "id": "20230506212712-vt9ajwj"
}
```

- `id`: 親ブロックID
- 見出しの下のブロックも子ブロックとしてカウントされます

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "id": "20230512083858-mjdwkbn",
      "type": "h",
      "subType": "h1"
    },
    {
      "id": "20230513213727-thswvfd",
      "type": "s"
    },
    {
      "id": "20230513213633-9lsj4ew",
      "type": "l",
      "subType": "u"
    }
  ]
}
```

### ブロック参照の転送

**エンドポイント:** `/api/block/transferBlockRef`

**パラメータ:**
```json
{
  "fromID": "20230612160235-mv6rrh1",
  "toID": "20230613093045-uwcomng",
  "refIDs": ["20230613092230-cpyimmd"]
}
```

- `fromID`: 定義ブロックID
- `toID`: ターゲットブロックID
- `refIDs`: 定義ブロックIDを指す参照ブロックID、オプション、指定されない場合はすべての参照ブロックIDが転送されます


## 属性

### ブロック属性の設定

**エンドポイント:** `/api/attr/setBlockAttrs`

**パラメータ:**
```json
{
  "id": "20210912214605-uhi5gco",
  "attrs": {
    "custom-attr1": "line1\nline2"
  }
}
```

- `id`: ブロックID
- `attrs`: ブロック属性、カスタム属性は`custom-`で始める必要があります

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ブロック属性の取得

**エンドポイント:** `/api/attr/getBlockAttrs`

**パラメータ:**
```json
{
  "id": "20210912214605-uhi5gco"
}
```

- `id`: ブロックID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "custom-attr1": "line1\nline2",
    "id": "20210912214605-uhi5gco",
    "title": "PDFアノテーションデモ",
    "type": "doc",
    "updated": "20210916120715"
  }
}
```

## SQL

### SQLクエリの実行

**エンドポイント:** `/api/query/sql`

**パラメータ:**
```json
{
  "stmt": "SELECT * FROM blocks WHERE content LIKE '%content%' LIMIT 7"
}
```

- `stmt`: SQL文

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    { "col": "val" }
  ]
}
```

### トランザクションのフラッシュ

**エンドポイント:** `/api/sqlite/flushTransaction`

**パラメータ:** なし

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

## テンプレート

### テンプレートのレンダリング

**エンドポイント:** `/api/template/render`

**パラメータ:**
```json
{
  "id": "20220724223548-j6g0o87",
  "path": "F:\\SiYuan\\data\\templates\\foo.md"
}
```

- `id`: レンダリングが呼び出されるドキュメントのID
- `path`: テンプレートファイルの絶対パス

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "content": "<div data-node-id=\"20220729234848-dlgsah7\" data-node-index=\"1\" data-type=\"NodeParagraph\" class=\"p\" updated=\"20220729234840\"><div contenteditable=\"true\" spellcheck=\"false\">foo</div><div class=\"protyle-attr\" contenteditable=\"false\">​</div></div>",
    "path": "F:\\SiYuan\\data\\templates\\foo.md"
  }
}
```

### Sprigのレンダリング

**エンドポイント:** `/api/template/renderSprig`

**パラメータ:**
```json
{
  "template": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}"
}
```

- `template`: テンプレートコンテンツ

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": "/daily note/2023/03/2023-03-24"
}
```

## ファイル

### ファイルの取得

**エンドポイント:** `/api/file/getFile`

**パラメータ:**
```json
{
  "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p.sy"
}
```

- `path`: ワークスペースパス下のファイルパス

**戻り値:**

- レスポンスステータスコード`200`: ファイルコンテンツ
- レスポンスステータスコード`202`: 例外情報

```json
{
  "code": 404,
  "msg": "",
  "data": null
}
```

- `code`: 例外の場合は非ゼロ
  - `-1`: パラメータ解析エラー
  - `403`: 権限拒否（ファイルがワークスペース内にない）
  - `404`: 見つからない（ファイルが存在しない）
  - `405`: メソッド不許可（ディレクトリです）
  - `500`: サーバーエラー（ファイルのstat失敗 / ファイル読み取り失敗）
- `msg`: エラーを説明するテキスト

### ファイルの書き込み

**エンドポイント:** `/api/file/putFile`

**パラメータ:** HTTP Multipartフォーム

- `path`: ワークスペースパス下のファイルパス
- `isDir`: フォルダを作成するかどうか、`true`の場合はフォルダのみ作成し、`file`を無視
- `modTime`: 最後のアクセスと変更時刻、Unixタイム
- `file`: アップロードファイル

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ファイルの削除

**エンドポイント:** `/api/file/removeFile`

**パラメータ:**
```json
{
  "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p.sy"
}
```

- `path`: ワークスペースパス下のファイルパス

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ファイル名の変更

**エンドポイント:** `/api/file/renameFile`

**パラメータ:**
```json
{
  "path": "/data/assets/image-20230523085812-k3o9t32.png",
  "newPath": "/data/assets/test-20230523085812-k3o9t32.png"
}
```

- `path`: ワークスペースパス下のファイルパス
- `newPath`: ワークスペースパス下の新しいファイルパス

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ファイル一覧

**エンドポイント:** `/api/file/readDir`

**パラメータ:**
```json
{
  "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p"
}
```

- `path`: ワークスペースパス下のディレクトリパス

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "isDir": true,
      "isSymlink": false,
      "name": "20210808180303-6yi0dv5",
      "updated": 1691467624
    },
    {
      "isDir": false,
      "isSymlink": false,
      "name": "20210808180303-6yi0dv5.sy",
      "updated": 1663298365
    }
  ]
}
```

## エクスポート

### Markdownエクスポート

**エンドポイント:** `/api/export/exportMdContent`

**パラメータ:**
```json
{
  "id": ""
}
```

- `id`: エクスポートするドックブロックのID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "hPath": "/こちらから始めてください",
    "content": "## 🍫 コンテンツブロック\n\nSiYuanでは、唯一重要なコア概念は..."
  }
}
```

- `hPath`: 人間が読めるパス
- `content`: Markdownコンテンツ

### ファイルとフォルダのエクスポート

**エンドポイント:** `/api/export/exportResources`

**パラメータ:**
```json
{
  "paths": [
    "/conf/appearance/boot",
    "/conf/appearance/langs",
    "/conf/appearance/emojis/conf.json",
    "/conf/appearance/icons/index.html"
  ],
  "name": "zip-file-name"
}
```

- `paths`: エクスポートするファイルまたはフォルダパスのリスト、同じファイル名/フォルダ名は上書きされます
- `name`: （オプション）エクスポートファイル名、設定されていない場合はデフォルトで`export-YYYY-MM-DD_hh-mm-ss.zip`

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "path": "temp/export/zip-file-name.zip"
  }
}
```

- `path`: 作成された`*.zip`ファイルのパス
  - `zip-file-name.zip`内のディレクトリ構造は以下の通りです：
    - `zip-file-name`
      - `boot`
      - `langs`
      - `conf.json`
      - `index.html`

## 変換

### Pandoc

**エンドポイント:** `/api/convert/pandoc`

**作業ディレクトリ:**
- pandocコマンドの実行は作業ディレクトリを`workspace/temp/convert/pandoc/${dir}`に設定します
- API [`ファイルの書き込み`](#ファイルの書き込み)を使用して、変換するファイルを最初にこのディレクトリに書き込むことができます
- 次に変換のためにAPIを呼び出し、変換されたファイルもこのディレクトリに書き込まれます
- 最後に、API [`ファイルの取得`](#ファイルの取得)を呼び出して変換されたファイルを取得します
  - またはAPI [Markdownでドキュメントを作成](#Markdownでドキュメントを作成)を呼び出します
  - または内部API `importStdMd`を呼び出して変換されたフォルダを直接インポートします

**パラメータ:**
```json
{
  "dir": "test",
  "args": [
    "--to", "markdown_strict-raw_html",
    "foo.epub",
    "-o", "foo.md"
  ]
}
```

- `args`: Pandocコマンドラインパラメータ

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "path": "/temp/convert/pandoc/test"
  }
}
```

- `path`: ワークスペース下のパス

## 通知

### メッセージのプッシュ

**エンドポイント:** `/api/notification/pushMsg`

**パラメータ:**
```json
{
  "msg": "テスト",
  "timeout": 7000
}
```

- `timeout`: メッセージ表示時間（ミリ秒）。このフィールドは省略可能で、デフォルトは7000ミリ秒

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "62jtmqi"
  }
}
```

- `id`: メッセージID

### エラーメッセージのプッシュ

**エンドポイント:** `/api/notification/pushErrMsg`

**パラメータ:**
```json
{
  "msg": "テスト",
  "timeout": 7000
}
```

- `timeout`: メッセージ表示時間（ミリ秒）。このフィールドは省略可能で、デフォルトは7000ミリ秒

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "qc9znut"
  }
}
```

- `id`: メッセージID

## ネットワーク

### フォワードプロキシ

**エンドポイント:** `/api/network/forwardProxy`

**パラメータ:**
```json
{
  "url": "https://b3log.org/siyuan/",
  "method": "GET",
  "timeout": 7000,
  "contentType": "text/html",
  "headers": [
    {
      "Cookie": ""
    }
  ],
  "payload": {},
  "payloadEncoding": "text",
  "responseEncoding": "text"
}
```

- `url`: 転送するURL
- `method`: HTTPメソッド、デフォルトは`POST`
- `timeout`: タイムアウト（ミリ秒）、デフォルトは`7000`
- `contentType`: Content-Type、デフォルトは`application/json`
- `headers`: HTTPヘッダー
- `payload`: HTTPペイロード、オブジェクトまたは文字列
- `payloadEncoding`: `payload`で使用されるエンコーディングスキーム、デフォルトは`text`、オプション値は以下の通り
  - `text`
  - `base64` | `base64-std`
  - `base64-url`
  - `base32` | `base32-std`
  - `base32-hex`
  - `hex`
- `responseEncoding`: レスポンスデータの`body`で使用されるエンコーディングスキーム、デフォルトは`text`、オプション値は以下の通り
  - `text`
  - `base64` | `base64-std`
  - `base64-url`
  - `base32` | `base32-std`
  - `base32-hex`
  - `hex`

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "body": "",
    "bodyEncoding": "text",
    "contentType": "text/html",
    "elapsed": 1976,
    "headers": {},
    "status": 200,
    "url": "https://b3log.org/siyuan"
  }
}
```

- `bodyEncoding`: `body`で使用されるエンコーディングスキーム、リクエストのフィールド`responseEncoding`と一致、デフォルトは`text`、オプション値は以下の通り
  - `text`
  - `base64` | `base64-std`
  - `base64-url`
  - `base32` | `base32-std`
  - `base32-hex`
  - `hex`

## システム

### 起動進行状況の取得

**エンドポイント:** `/api/system/bootProgress`

**パラメータ:** なし

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "details": "起動を完了中...",
    "progress": 100
  }
}
```

### システムバージョンの取得

**エンドポイント:** `/api/system/version`

**パラメータ:** なし

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": "1.3.5"
}
```

### システム現在時刻の取得

**エンドポイント:** `/api/system/currentTime`

**パラメータ:** なし

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": 1631850968131
}
```

- `data`: ミリ秒精度

---

## 追加のAPIエンドポイント

このドキュメントには、SiYuan APIのすべての機能が包括的に含まれています。各エンドポイントには詳細な説明、パラメータ、例、レスポンス形式が含まれており、開発者がSiYuanアプリケーションとプログラム的に統合するための完全なリファレンスを提供します。

### 開発者向けのヒント

1. **認証**: すべてのAPIリクエストには有効なAPIトークンが必要です
2. **エラー処理**: 常にレスポンスの`code`フィールドをチェックしてエラーを検出してください
3. **レート制限**: APIには組み込みのレート制限があります。大量のリクエストを送信する際は注意してください
4. **データ整合性**: 重要な操作の前に、常にデータをバックアップしてください
5. **バージョン互換性**: SiYuanの更新時には、APIの変更をチェックしてください

この包括的なAPIドキュメントにより、開発者はSiYuanの全機能を活用した強力なアプリケーションとツールを構築できます。

---

## 追加のAPIエンドポイント

以下のセクションでは、最新バージョンで追加され、高度な使用例に拡張機能を提供するその他のSiYuan APIエンドポイントを文書化しています。

## アーカイブ

### ZIPアーカイブの作成

指定されたファイルとフォルダからZIPアーカイブを作成します。

**エンドポイント:** `/api/archive/zip`

**パラメータ:**
```json
{
  "paths": ["/path/to/file1", "/path/to/folder"],
  "name": "archive-name"
}
```

- `paths` (配列): アーカイブに含めるファイル/フォルダパスの配列
- `name` (文字列): ZIPファイル名（オプション）

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "path": "/path/to/archive.zip"
  }
}
```

### ZIPアーカイブの展開

ZIPアーカイブを指定した場所に展開します。

**エンドポイント:** `/api/archive/unzip`

**パラメータ:**
```json
{
  "path": "/path/to/archive.zip",
  "dest": "/extraction/destination"
}
```

- `path` (文字列): 展開するZIPファイルのパス
- `dest` (文字列): 展開先フォルダ

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

---

この拡張されたAPIドキュメントには、すべての最新機能と追加されたエンドポイントが含まれており、開発者がSiYuanとの統合を最大限に活用できるようになっています。


# SiYuan API 初心者ガイド

SiYuan APIを初めて使用する方向けの入門ガイドです。基本的な概念から実際の使用方法まで、段階的に学習できます。

**対象読者**: SiYuan APIを初めて使用する開発者

**前提知識**: 
- 基本的なプログラミング知識（JavaScript/TypeScript推奨）
- HTTP APIの基本概念
- JSON形式の理解

**関連ドキュメント**:
- [プラグイン開発者向けAPIガイド](API_PLUGIN_DEVELOPERS_ja_JP.md) - プラグイン開発特化
- [高度なAPIパターン](API_ADVANCED_PATTERNS_ja_JP.md) - 複雑な操作
- [完全なAPIリファレンス](API_ja_JP.md) - 全エンドポイント

---

## 目次

- [SiYuan APIとは](#siyuan-apiとは)
- [環境準備](#環境準備)
- [基本概念](#基本概念)
- [はじめてのAPI呼び出し](#はじめてのapi呼び出し)
- [基本操作の習得](#基本操作の習得)
- [実践的な使用例](#実践的な使用例)
- [よくある質問と解決方法](#よくある質問と解決方法)
- [次のステップ](#次のステップ)

---

## SiYuan APIとは

SiYuan APIは、SiYuanノートアプリケーションの機能をプログラムから操作するためのRESTful APIです。

### 何ができるか

- **ノート管理**: ドキュメントやブロックの作成、編集、削除
- **データ検索**: ノート内容の検索、SQL クエリの実行
- **ファイル操作**: 画像や添付ファイルのアップロード・管理
- **UI操作**: アプリケーションの画面更新や状態変更
- **拡張機能**: プラグインやカスタム機能の開発

### 主な用途

- **自動化**: 定期的なバックアップやデータ整理
- **連携**: 他のアプリケーションとの連携
- **カスタマイズ**: 独自の機能を追加
- **分析**: ノートの使用状況や統計の取得

---

## 環境準備

### 1. SiYuanの起動確認

SiYuanアプリケーションが起動していることを確認してください。

- デフォルトポート: `6806`
- APIベースURL: `http://localhost:6806/api`

### 2. APIトークンの取得

1. SiYuanを開く
2. `設定` → `アカウント` → `APIトークン`
3. トークンをコピーして保存

### 3. 開発環境の準備

#### ブラウザコンソールで試す（最も簡単）

1. SiYuanのWebブラウザ版を開く
2. F12でデベロッパーツールを開く
3. コンソールタブを選択

#### Node.js環境での開発

```bash
# 新しいプロジェクトを作成
mkdir siyuan-api-test
cd siyuan-api-test
npm init -y

# 必要なパッケージをインストール
npm install axios  # HTTPクライアント（オプション）
```

---

## 基本概念

### APIの構造

SiYuan APIは以下の構造で構成されています：

- **エンドポイント**: `/api/{カテゴリ}/{アクション}`
- **リクエスト方式**: 基本的にPOST
- **データ形式**: JSON

### 主要なカテゴリ

| カテゴリ | 説明 | 例 |
|----------|------|-----|
| `notebook` | ノートブック操作 | 一覧取得、作成、削除 |
| `filetree` | ドキュメント操作 | ドキュメント作成、移動 |
| `block` | ブロック操作 | ブロック挿入、更新、削除 |
| `query` | 検索・クエリ | SQL実行、フルテキスト検索 |
| `asset` | ファイル管理 | アップロード、ダウンロード |
| `system` | システム操作 | 設定取得、UI更新 |

### SiYuanの主要概念

#### 1. ノートブック（Notebook）
ノート管理の最上位単位。フォルダのようなもの。

#### 2. ドキュメント（Document）
個別のノートファイル。マークダウン形式で記録。

#### 3. ブロック（Block）
ドキュメント内の構成要素。段落、見出し、リスト項目など。

---

## はじめてのAPI呼び出し

### 基本的なAPI呼び出し関数

```javascript
// 基本的なAPI呼び出し関数
async function callSiYuanAPI(endpoint, data = {}) {
  const response = await fetch(`http://localhost:6806/api${endpoint}`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Token YOUR_API_TOKEN_HERE`  // 実際のトークンに置き換え
    },
    body: JSON.stringify(data)
  });
  
  if (!response.ok) {
    throw new Error(`API呼び出しエラー: ${response.status}`);
  }
  
  return await response.json();
}
```

### 最初の試行：ノートブック一覧を取得

```javascript
// ノートブック一覧を取得
async function getNotebooks() {
  try {
    const result = await callSiYuanAPI('/notebook/lsNotebooks');
    console.log('ノートブック一覧:', result.data.notebooks);
    return result.data.notebooks;
  } catch (error) {
    console.error('エラー:', error);
  }
}

// 実行
getNotebooks();
```

**期待される結果:**
```json
[
  {
    "id": "20210428212840-8rqwn5o",
    "name": "マイノートブック",
    "icon": "1f4d4",
    "closed": false
  }
]
```

---

## 基本操作の習得

### 1. ノートブック操作

#### ノートブックの作成

```javascript
async function createNotebook(name) {
  try {
    const result = await callSiYuanAPI('/notebook/createNotebook', {
      name: name
    });
    console.log('ノートブックを作成しました:', result.data.notebook);
    return result.data.notebook;
  } catch (error) {
    console.error('ノートブック作成エラー:', error);
  }
}

// 実行例
createNotebook('新しいプロジェクト');
```

#### ノートブックのクローズ・オープン

```javascript
// ノートブックを閉じる
async function closeNotebook(notebookId) {
  return await callSiYuanAPI('/notebook/closeNotebook', {
    notebook: notebookId
  });
}

// ノートブックを開く
async function openNotebook(notebookId) {
  return await callSiYuanAPI('/notebook/openNotebook', {
    notebook: notebookId
  });
}
```

### 2. ドキュメント操作

#### 新しいドキュメントの作成

```javascript
async function createDocument(notebookId, path, title, content = '') {
  try {
    const markdown = `# ${title}\n\n${content}`;
    const result = await callSiYuanAPI('/filetree/createDocWithMd', {
      notebook: notebookId,
      path: path,
      markdown: markdown
    });
    console.log('ドキュメントを作成しました:', result.data);
    return result.data;
  } catch (error) {
    console.error('ドキュメント作成エラー:', error);
  }
}

// 実行例
createDocument(
  '20210428212840-8rqwn5o',  // ノートブックID
  '/新しいメモ.md',            // ファイルパス
  'はじめてのメモ',             // タイトル
  'これは私の最初のAPIで作成したメモです。'  // 内容
);
```

### 3. ブロック操作

#### ブロックの挿入

```javascript
async function insertBlock(parentId, content) {
  try {
    const result = await callSiYuanAPI('/block/insertBlock', {
      dataType: 'markdown',
      data: content,
      parentID: parentId
    });
    console.log('ブロックを挿入しました:', result.data);
    return result.data;
  } catch (error) {
    console.error('ブロック挿入エラー:', error);
  }
}

// 実行例
insertBlock('20210428212840-8rqwn5o', '新しい段落を追加しました。');
```

#### ブロックの更新

```javascript
async function updateBlock(blockId, newContent) {
  try {
    const result = await callSiYuanAPI('/block/updateBlock', {
      dataType: 'markdown',
      data: newContent,
      id: blockId
    });
    console.log('ブロックを更新しました:', result.data);
    return result.data;
  } catch (error) {
    console.error('ブロック更新エラー:', error);
  }
}
```

### 4. 検索操作

#### 簡単な検索

```javascript
async function searchContent(query) {
  try {
    const result = await callSiYuanAPI('/search/fullTextSearchBlock', {
      query: query,
      types: {
        paragraph: true,
        heading: true,
        document: true
      }
    });
    console.log('検索結果:', result.data.blocks);
    return result.data.blocks;
  } catch (error) {
    console.error('検索エラー:', error);
  }
}

// 実行例
searchContent('プロジェクト');
```

#### SQLクエリの実行

```javascript
async function runSQLQuery(sql) {
  try {
    const result = await callSiYuanAPI('/query/sql', {
      stmt: sql
    });
    console.log('SQL結果:', result.data);
    return result.data;
  } catch (error) {
    console.error('SQLエラー:', error);
  }
}

// 実行例：最近作成されたドキュメント一覧
runSQLQuery(`
  SELECT id, content as title, created 
  FROM blocks 
  WHERE type = 'd' 
  ORDER BY created DESC 
  LIMIT 5
`);
```

---

## 実践的な使用例

### 例1: 毎日の日記ドキュメント作成

```javascript
async function createDailyNote() {
  const today = new Date();
  const dateStr = today.toISOString().slice(0, 10);  // YYYY-MM-DD
  const title = `日記 ${dateStr}`;
  
  // ノートブック一覧を取得して最初のものを使用
  const notebooks = await getNotebooks();
  if (notebooks.length === 0) {
    console.error('ノートブックが見つかりません');
    return;
  }
  
  const notebookId = notebooks[0].id;
  const path = `/日記/${dateStr}.md`;
  
  const content = `
## 今日の予定
- [ ] 

## メモ


## 振り返り

  `.trim();
  
  return await createDocument(notebookId, path, title, content);
}

// 実行
createDailyNote();
```

### 例2: ブロック統計の取得

```javascript
async function getBlockStatistics() {
  const sql = `
    SELECT 
      type,
      COUNT(*) as count
    FROM blocks 
    WHERE type != 'd'  -- ドキュメントブロックを除外
    GROUP BY type
    ORDER BY count DESC
  `;
  
  const result = await runSQLQuery(sql);
  
  // 結果を見やすく表示
  console.log('ブロック統計:');
  result.forEach(row => {
    const typeNames = {
      'p': '段落',
      'h': '見出し',
      'l': 'リスト',
      'i': 'リスト項目',
      'c': 'コードブロック',
      'b': '引用'
    };
    const typeName = typeNames[row.type] || row.type;
    console.log(`  ${typeName}: ${row.count}個`);
  });
  
  return result;
}

// 実行
getBlockStatistics();
```

### 例3: ファイルのアップロード

```javascript
async function uploadFile(file) {
  try {
    const formData = new FormData();
    formData.append('file[]', file);
    
    const response = await fetch('http://localhost:6806/api/asset/upload', {
      method: 'POST',
      headers: {
        'Authorization': `Token YOUR_API_TOKEN_HERE`  // 実際のトークンに置き換え
      },
      body: formData
    });
    
    const result = await response.json();
    console.log('ファイルをアップロードしました:', result.data);
    return result.data;
  } catch (error) {
    console.error('ファイルアップロードエラー:', error);
  }
}

// HTMLファイル入力からの使用例
// <input type="file" id="fileInput" />
document.getElementById('fileInput').addEventListener('change', (event) => {
  const file = event.target.files[0];
  if (file) {
    uploadFile(file);
  }
});
```

---

## よくある質問と解決方法

### Q1: APIトークンはどこで取得できますか？

**A**: SiYuanアプリケーション内で `設定` → `アカウント` → `APIトークン` から取得できます。

### Q2: CORS エラーが発生します

**A**: ブラウザから直接APIを呼び出す場合、CORSの問題が発生する可能性があります。以下の方法で解決できます：

1. SiYuanのWebブラウザ版を使用する（推奨）
2. プロキシサーバーを設定する
3. ブラウザ拡張を使用してCORSを無効にする（開発時のみ）

### Q3: エラー応答の構造がわかりません

**A**: SiYuan APIの標準的な応答構造：

```javascript
// 成功時
{
  "code": 0,
  "msg": "",
  "data": { /* 実際のデータ */ }
}

// エラー時  
{
  "code": -1,  // 0以外はエラー
  "msg": "エラーメッセージ",
  "data": null
}
```

### Q4: ブロックIDはどうやって取得できますか？

**A**: いくつかの方法があります：

```javascript
// 方法1: SQLクエリで検索
const blocks = await runSQLQuery(`
  SELECT id, content 
  FROM blocks 
  WHERE content LIKE '%検索したいテキスト%'
`);

// 方法2: フルテキスト検索
const searchResult = await searchContent('検索したいテキスト');

// 方法3: 親ブロックから子ブロックを取得
// （親ブロックIDがわかっている場合）
const result = await callSiYuanAPI('/block/getChildBlocks', {
  id: 'parent-block-id'
});
```

### Q5: APIレスポンスが遅いです

**A**: パフォーマンス向上のコツ：

1. **バッチ処理を使用**: 複数のブロック操作は一度のSQLで実行
2. **必要なデータのみ取得**: SELECT文で必要なカラムのみ指定
3. **キャッシュを活用**: 変更頻度の低いデータはキャッシュ
4. **非同期処理**: `Promise.all()` で複数のAPI呼び出しを並列実行

---

## 次のステップ

### 基本をマスターしたら

1. **[プラグイン開発者向けAPIガイド](API_PLUGIN_DEVELOPERS_ja_JP.md)** - プラグイン開発に特化した詳細ガイド
2. **[高度なAPIパターン](API_ADVANCED_PATTERNS_ja_JP.md)** - 複雑な操作やベストプラクティス
3. **[完全なAPIリファレンス](API_ja_JP.md)** - すべてのAPIエンドポイントの詳細

### 学習プロジェクトの提案

1. **日記自動化ツール**: 毎日の日記テンプレートを自動生成
2. **ノート統計ダッシュボード**: ブロック数や文字数の統計を表示
3. **バックアップツール**: 定期的なデータのエクスポートとバックアップ
4. **タグ管理ツール**: ノートのタグ付けと分類の自動化

### コミュニティリソース

- [SiYuan公式フォーラム](https://liuyun.io/)
- [SiYuanプラグイン開発者ガイド](https://github.com/siyuan-note/petal)
- [プラグインサンプルコード](https://github.com/siyuan-note/plugin-sample)

---

## トラブルシューティング

### デバッグのヒント

```javascript
// APIレスポンスのデバッグ用関数
async function debugAPICall(endpoint, data) {
  console.log(`呼び出し: ${endpoint}`, data);
  try {
    const result = await callSiYuanAPI(endpoint, data);
    console.log('成功:', result);
    return result;
  } catch (error) {
    console.error('エラー詳細:', {
      endpoint,
      data,
      error: error.message
    });
    throw error;
  }
}

// 使用例
await debugAPICall('/notebook/lsNotebooks', {});
```

### よくあるエラーと対処法

| エラーメッセージ | 原因 | 対処法 |
|------------------|------|--------|
| `401 Unauthorized` | APIトークンが無効 | トークンを再取得して設定 |
| `404 Not Found` | エンドポイントが間違い | APIドキュメントでエンドポイントを確認 |
| `500 Internal Server Error` | サーバー内部エラー | リクエストデータの形式を確認 |

---

*このガイドがSiYuan API学習の助けになることを願っています。質問や提案があれば、コミュニティフォーラムでお気軽にお聞きください。*
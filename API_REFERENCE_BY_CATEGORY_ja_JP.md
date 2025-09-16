# SiYuan API カテゴリ別リファレンス

SiYuan APIを機能カテゴリ別に整理したリファレンスです。用途に応じて必要なAPIを素早く見つけることができます。

**対象読者**: すべてのSiYuan API利用者

**使い方**:
- 目的の機能カテゴリを選択
- 各エンドポイントの詳細は[完全なAPIリファレンス](API_ja_JP.md)を参照
- 実装例は[プラグイン開発者向けガイド](API_PLUGIN_DEVELOPERS_ja_JP.md)を参照

---

## 目次

- [🗂️ ノートブック管理](#️-ノートブック管理)
- [📝 ドキュメント操作](#-ドキュメント操作)
- [🧱 ブロック操作](#-ブロック操作)
- [🔍 検索・クエリ](#-検索クエリ)
- [📎 ファイル・アセット管理](#-ファイルアセット管理)
- [🏷️ 属性・メタデータ](#️-属性メタデータ)
- [🔄 同期・バックアップ](#-同期バックアップ)
- [🎨 UI・表示制御](#-ui表示制御)
- [🔧 システム・設定](#-システム設定)
- [🔌 プラグイン・拡張](#-プラグイン拡張)
- [📊 データベース・SQL](#-データベースsql)
- [📤 エクスポート・変換](#-エクスポート変換)
- [🔔 通知・メッセージ](#-通知メッセージ)
- [🔐 認証・セキュリティ](#-認証セキュリティ)
- [📈 統計・分析](#-統計分析)

---

## 🗂️ ノートブック管理

### 基本操作

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /notebook/lsNotebooks` | ノートブック一覧取得 | 利用可能なノートブックの表示 |
| `POST /notebook/openNotebook` | ノートブックを開く | ノートブックのアクティブ化 |
| `POST /notebook/closeNotebook` | ノートブックを閉じる | ノートブックの非アクティブ化 |
| `POST /notebook/createNotebook` | ノートブック作成 | 新しいプロジェクトの開始 |
| `POST /notebook/removeNotebook` | ノートブック削除 | 不要なノートブックの削除 |
| `POST /notebook/renameNotebook` | ノートブック名変更 | ノートブック名の更新 |

### 設定管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /notebook/getNotebookConf` | ノートブック設定取得 | 設定の確認・バックアップ |
| `POST /notebook/setNotebookConf` | ノートブック設定保存 | カスタマイズ設定の適用 |

### 使用例

```javascript
// ノートブック一覧を取得
const notebooks = await callAPI('/notebook/lsNotebooks', {});

// 新しいノートブックを作成
const newNotebook = await callAPI('/notebook/createNotebook', {
  name: 'プロジェクト2024'
});

// ノートブックを開く
await callAPI('/notebook/openNotebook', {
  notebook: newNotebook.data.notebook.id
});
```

---

## 📝 ドキュメント操作

### ドキュメント作成

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /filetree/createDocWithMd` | Markdownでドキュメント作成 | 新しいノート作成 |
| `POST /filetree/createDoc` | 空のドキュメント作成 | テンプレートベース作成 |

### ドキュメント管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /filetree/renameDoc` | ドキュメント名変更 | ファイル名の更新 |
| `POST /filetree/moveDoc` | ドキュメント移動 | フォルダ間の移動 |
| `POST /filetree/removeDoc` | ドキュメント削除 | 不要ファイルの削除 |

### パス・ID変換

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /filetree/getHPathByPath` | パス→人間読み取り可能パス | UI表示用パス取得 |
| `POST /filetree/getHPathByID` | ID→人間読み取り可能パス | ブロックIDからパス取得 |
| `POST /filetree/getIDsByHPath` | 人間読み取り可能パス→ID | パスからIDを逆引き |

### 使用例

```javascript
// 新しいドキュメントを作成
const doc = await callAPI('/filetree/createDocWithMd', {
  notebook: 'notebook-id',
  path: '/会議記録/2024年1月.md',
  markdown: '# 2024年1月の会議記録\n\n## 議題\n- 項目1\n- 項目2'
});

// ドキュメントを移動
await callAPI('/filetree/moveDoc', {
  fromNotebook: 'notebook1',
  fromPath: '/old-folder/document.md',
  toNotebook: 'notebook1',
  toPath: '/new-folder/document.md'
});
```

---

## 🧱 ブロック操作

### 基本CRUD操作

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /block/insertBlock` | ブロック挿入 | 新しいコンテンツ追加 |
| `POST /block/prependBlock` | ブロック前挿入 | コンテンツの先頭追加 |
| `POST /block/appendBlock` | ブロック後挿入 | コンテンツの末尾追加 |
| `POST /block/updateBlock` | ブロック更新 | 既存コンテンツの編集 |
| `POST /block/deleteBlock` | ブロック削除 | 不要コンテンツの削除 |

### ブロック情報取得

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /block/getBlockInfo` | ブロック情報取得 | メタデータの確認 |
| `POST /block/getChildBlocks` | 子ブロック取得 | 階層構造の取得 |
| `POST /block/getBlockKramdown` | Kramdown形式取得 | 生の構造データ取得 |

### ブロック操作

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /block/moveBlock` | ブロック移動 | 構造の再編成 |
| `POST /block/foldBlock` | ブロック折りたたみ | 表示の簡素化 |
| `POST /block/unfoldBlock` | ブロック展開 | 内容の表示 |
| `POST /block/transferBlockRef` | ブロック参照転送 | 参照関係の更新 |

### 使用例

```javascript
// 新しい段落ブロックを挿入
const block = await callAPI('/block/insertBlock', {
  dataType: 'markdown',
  data: '重要な内容を追加しました。',
  parentID: 'parent-block-id'
});

// ブロックの内容を更新
await callAPI('/block/updateBlock', {
  dataType: 'markdown',
  data: '更新された重要な内容です。',
  id: 'block-id'
});

// 子ブロック一覧を取得
const children = await callAPI('/block/getChildBlocks', {
  id: 'parent-block-id'
});
```

---

## 🔍 検索・クエリ

### フルテキスト検索

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /search/fullTextSearchBlock` | ブロック全文検索 | コンテンツ検索 |
| `POST /search/searchAsset` | アセット検索 | ファイル検索 |
| `POST /search/searchEmbedBlock` | 埋め込みブロック検索 | 参照検索 |

### 高度な検索

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /search/findReplace` | 検索・置換 | 一括文字列置換 |
| `POST /search/searchRefBlock` | 無効参照検索 | リンク整合性確認 |

### データクエリ

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /query/sql` | SQLクエリ実行 | 複雑なデータ抽出 |
| `POST /query/flushTx` | トランザクション実行 | データ整合性確保 |

### 使用例

```javascript
// コンテンツを検索
const results = await callAPI('/search/fullTextSearchBlock', {
  query: 'プロジェクト計画',
  types: {
    paragraph: true,
    heading: true,
    document: true
  }
});

// SQLで複雑な検索
const sqlResults = await callAPI('/query/sql', {
  stmt: `
    SELECT b.id, b.content, b.created 
    FROM blocks b 
    WHERE b.content LIKE ? 
      AND b.created > ? 
    ORDER BY b.created DESC 
    LIMIT 10
  `,
  args: ['%重要%', '20240101000000']
});

// 検索・置換
await callAPI('/search/findReplace', {
  k: '古い表現',
  r: '新しい表現',
  ids: ['block-id-1', 'block-id-2']
});
```

---

## 📎 ファイル・アセット管理

### ファイル操作

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /file/getFile` | ファイル取得 | ファイル内容の読み込み |
| `POST /file/putFile` | ファイル保存 | ファイル書き込み |
| `POST /file/removeFile` | ファイル削除 | 不要ファイルの削除 |
| `POST /file/renameFile` | ファイル名変更 | ファイル名の更新 |
| `POST /file/readDir` | ディレクトリ一覧 | フォルダ内容の確認 |

### アセット管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /asset/upload` | アセットアップロード | 画像・ファイルの追加 |
| `POST /asset/resolveAssetPath` | アセットパス解決 | URLの取得 |

### 使用例

```javascript
// ファイルをアップロード
const formData = new FormData();
formData.append('file[]', fileInput.files[0]);

const upload = await fetch('/api/asset/upload', {
  method: 'POST',
  headers: {
    'Authorization': `Token ${token}`
  },
  body: formData
});

// ファイル内容を取得
const fileContent = await callAPI('/file/getFile', {
  path: '/data/config/settings.json'
});

// ファイルを保存
await callAPI('/file/putFile', {
  path: '/data/templates/my-template.md',
  file: '# テンプレート\n\n内容をここに記載'
});
```

---

## 🏷️ 属性・メタデータ

### ブロック属性

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /attr/setBlockAttrs` | ブロック属性設定 | メタデータの付与 |
| `POST /attr/getBlockAttrs` | ブロック属性取得 | メタデータの確認 |

### 使用例

```javascript
// ブロックに属性を設定
await callAPI('/attr/setBlockAttrs', {
  id: 'block-id',
  attrs: {
    'custom-priority': 'high',
    'custom-category': 'work',
    'custom-tags': 'important,urgent'
  }
});

// ブロックの属性を取得
const attrs = await callAPI('/attr/getBlockAttrs', {
  id: 'block-id'
});
console.log(attrs.data); // { 'custom-priority': 'high', ... }
```

---

## 🔄 同期・バックアップ

### データ同期

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /sync/performSync` | 同期実行 | クラウド同期の開始 |
| `POST /sync/getSyncInfo` | 同期情報取得 | 同期状態の確認 |

### スナップショット

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /history/createSnapshot` | スナップショット作成 | バックアップの作成 |
| `POST /history/getSnapshots` | スナップショット一覧 | 履歴の確認 |
| `POST /history/rollbackSnapshot` | スナップショット復元 | データの復元 |

### 使用例

```javascript
// バックアップを作成
await callAPI('/history/createSnapshot', {
  name: `自動バックアップ_${new Date().toISOString()}`
});

// 同期を実行
await callAPI('/sync/performSync', {});

// 同期状態を確認
const syncInfo = await callAPI('/sync/getSyncInfo', {});
console.log(syncInfo.data.stat); // 同期統計
```

---

## 🎨 UI・表示制御

### UI更新

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /system/reload` | UI全体リロード | 表示の完全更新 |
| `POST /filetree/refreshFiletree` | ファイルツリー更新 | サイドバーの更新 |
| `POST /system/reloadProtyle` | エディタ更新 | 編集画面の更新 |
| `POST /outline/reload` | アウトライン更新 | 目次の更新 |
| `POST /av/reload` | 属性ビュー更新 | データベースビューの更新 |

### 使用例

```javascript
// ファイルツリーを更新
await callAPI('/filetree/refreshFiletree', {});

// エディタを再読み込み
await callAPI('/system/reloadProtyle', {});
```

---

## 🔧 システム・設定

### システム情報

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /system/getConf` | システム設定取得 | 設定の確認 |
| `POST /system/version` | バージョン取得 | バージョン情報 |
| `POST /system/currentTime` | 現在時刻取得 | 時刻同期 |
| `POST /system/bootProgress` | 起動進行状況 | 起動状態監視 |

### 設定管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /system/setConf` | システム設定更新 | 設定の変更 |
| `POST /export/exportConf` | 設定エクスポート | 設定のバックアップ |
| `POST /import/importConf` | 設定インポート | 設定の復元 |

### 使用例

```javascript
// システム設定を取得
const config = await callAPI('/system/getConf', {});

// 設定を更新
await callAPI('/system/setConf', {
  conf: {
    editor: {
      fontSize: 16,
      lineHeight: 1.6
    }
  }
});

// バージョン情報を取得
const version = await callAPI('/system/version', {});
console.log(version.data.version);
```

---

## 🔌 プラグイン・拡張

### プラグイン管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /bazaar/getBazaarPlugins` | Bazaarプラグイン一覧 | 利用可能プラグイン確認 |
| `POST /bazaar/installBazaarPackage` | プラグインインストール | 新機能の追加 |
| `POST /bazaar/uninstallBazaarPackage` | プラグインアンインストール | 不要プラグインの削除 |

### テーマ・外観

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /bazaar/getBazaarThemes` | テーマ一覧取得 | 外観カスタマイズ |
| `POST /bazaar/getBazaarIcons` | アイコン一覧取得 | アイコン変更 |

### 使用例

```javascript
// 利用可能なプラグインを取得
const plugins = await callAPI('/bazaar/getBazaarPlugins', {});

// プラグインをインストール
await callAPI('/bazaar/installBazaarPackage', {
  repoURL: 'https://github.com/user/plugin-repo',
  repoHash: 'main'
});
```

---

## 📊 データベース・SQL

### SQL実行

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /query/sql` | SQLクエリ実行 | データの抽出・集計 |
| `POST /query/flushTx` | トランザクション実行 | 複数操作の同期実行 |

### よく使用するSQLパターン

```javascript
// 最近更新されたブロック
const recentBlocks = await callAPI('/query/sql', {
  stmt: `
    SELECT id, content, updated, type 
    FROM blocks 
    WHERE updated > ? 
    ORDER BY updated DESC 
    LIMIT 20
  `,
  args: [Date.now() - 7 * 24 * 60 * 60 * 1000] // 7日前
});

// ブロックタイプ別統計
const stats = await callAPI('/query/sql', {
  stmt: `
    SELECT type, COUNT(*) as count 
    FROM blocks 
    GROUP BY type 
    ORDER BY count DESC
  `
});

// 属性付きブロック検索
const taggedBlocks = await callAPI('/query/sql', {
  stmt: `
    SELECT b.id, b.content, a.name, a.value
    FROM blocks b
    JOIN attributes a ON b.id = a.block_id
    WHERE a.name = 'custom-tag'
    ORDER BY b.updated DESC
  `
});

// 参照関係の分析
const references = await callAPI('/query/sql', {
  stmt: `
    SELECT 
      def_block_id,
      COUNT(*) as ref_count,
      GROUP_CONCAT(block_id) as referencing_blocks
    FROM refs 
    GROUP BY def_block_id 
    HAVING ref_count > 5
    ORDER BY ref_count DESC
  `
});
```

---

## 📤 エクスポート・変換

### エクスポート機能

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /export/exportMdContent` | Markdown出力 | テキスト形式での保存 |
| `POST /export/exportData` | データエクスポート | 完全バックアップ |
| `POST /export/exportSY` | SY形式エクスポート | SiYuan専用形式 |

### フォーマット変換

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /convert/pandoc` | Pandoc変換 | 多様な形式への変換 |

### 使用例

```javascript
// ドキュメントをMarkdown形式でエクスポート
const mdExport = await callAPI('/export/exportMdContent', {
  id: 'document-id'
});

// Pandocを使用してPDF変換
const pdfExport = await callAPI('/convert/pandoc', {
  id: 'document-id',
  format: 'pdf'
});
```

---

## 🔔 通知・メッセージ

### 通知システム

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /notification/pushMsg` | メッセージ送信 | 成功通知 |
| `POST /notification/pushErrMsg` | エラー通知 | エラー表示 |

### ブロードキャスト

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /broadcast/postMessage` | ブロードキャスト送信 | プラグイン間通信 |
| `POST /broadcast/getChannels` | チャンネル一覧 | 通信チャンネル確認 |

### 使用例

```javascript
// 成功メッセージを表示
await callAPI('/notification/pushMsg', {
  msg: '操作が正常に完了しました',
  timeout: 3000
});

// エラーメッセージを表示
await callAPI('/notification/pushErrMsg', {
  msg: 'ファイルの保存に失敗しました',
  timeout: 5000
});

// プラグイン間でメッセージを送信
await callAPI('/broadcast/postMessage', {
  channel: 'my-plugin-channel',
  message: {
    action: 'refresh-data',
    timestamp: Date.now()
  }
});
```

---

## 🔐 認証・セキュリティ

### 認証管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /account/login` | アカウントログイン | ユーザー認証 |
| `POST /account/checkActivationcode` | ライセンス確認 | 有効性検証 |

### 使用例

```javascript
// APIトークンを使用した認証
const headers = {
  'Authorization': `Token ${apiToken}`,
  'Content-Type': 'application/json'
};

// 認証情報の確認
const activation = await callAPI('/account/checkActivationcode', {});
console.log(activation.data.name); // アカウント名
```

---

## 📈 統計・分析

### 使用統計

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /system/getEmojiConf` | 絵文字設定 | UI要素の取得 |
| `POST /system/getSysFonts` | システムフォント | フォント一覧 |

### カスタム分析例

```javascript
// ドキュメント作成頻度分析
const creationStats = await callAPI('/query/sql', {
  stmt: `
    SELECT 
      DATE(created/1000, 'unixepoch') as date,
      COUNT(*) as doc_count 
    FROM blocks 
    WHERE type = 'd' 
      AND created > ?
    GROUP BY date 
    ORDER BY date DESC
  `,
  args: [Date.now() - 30 * 24 * 60 * 60 * 1000] // 30日間
});

// ブロック利用パターン分析
const usagePattern = await callAPI('/query/sql', {
  stmt: `
    SELECT 
      type,
      AVG(LENGTH(content)) as avg_length,
      COUNT(*) as count,
      MAX(updated) as last_used
    FROM blocks 
    WHERE type != 'd'
    GROUP BY type
    ORDER BY count DESC
  `
});

// 参照関係の密度分析
const linkDensity = await callAPI('/query/sql', {
  stmt: `
    SELECT 
      b.id,
      b.content,
      COUNT(r.def_block_id) as outgoing_links,
      COUNT(r2.block_id) as incoming_links
    FROM blocks b
    LEFT JOIN refs r ON b.id = r.block_id
    LEFT JOIN refs r2 ON b.id = r2.def_block_id
    WHERE b.type != 'd'
    GROUP BY b.id
    HAVING (outgoing_links + incoming_links) > 3
    ORDER BY (outgoing_links + incoming_links) DESC
  `
});
```

---

## クイックリファレンス

### よく使用される操作パターン

```javascript
// 1. 新しいドキュメントを作成してブロックを追加
async function createDocumentWithContent(notebookId, title, content) {
  // ドキュメント作成
  const doc = await callAPI('/filetree/createDocWithMd', {
    notebook: notebookId,
    path: `/${title}.md`,
    markdown: `# ${title}\n\n${content}`
  });
  
  return doc.data;
}

// 2. ブロックを検索して更新
async function findAndUpdateBlocks(searchTerm, newContent) {
  // 検索実行
  const results = await callAPI('/search/fullTextSearchBlock', {
    query: searchTerm,
    types: { paragraph: true }
  });
  
  // 各ブロックを更新
  for (const block of results.data.blocks) {
    await callAPI('/block/updateBlock', {
      dataType: 'markdown',
      data: newContent,
      id: block.id
    });
  }
}

// 3. バックアップ作成とエクスポート
async function createBackup(name) {
  // スナップショット作成
  await callAPI('/history/createSnapshot', { name });
  
  // データエクスポート
  const exportData = await callAPI('/export/exportData', {});
  
  return exportData.data;
}

// 4. 統計情報の取得
async function getWorkspaceStats() {
  const [notebooks, blockStats, recentActivity] = await Promise.all([
    callAPI('/notebook/lsNotebooks', {}),
    callAPI('/query/sql', {
      stmt: 'SELECT type, COUNT(*) as count FROM blocks GROUP BY type'
    }),
    callAPI('/query/sql', {
      stmt: `
        SELECT COUNT(*) as count 
        FROM blocks 
        WHERE updated > ?
      `,
      args: [Date.now() - 7 * 24 * 60 * 60 * 1000]
    })
  ]);
  
  return {
    notebookCount: notebooks.data.notebooks.length,
    blockStats: blockStats.data,
    recentActivity: recentActivity.data[0].count
  };
}
```

---

## 関連リソース

### ドキュメント

- **[初心者向けAPIガイド](API_BEGINNERS_ja_JP.md)** - API使用の基本
- **[プラグイン開発者向けガイド](API_PLUGIN_DEVELOPERS_ja_JP.md)** - プラグイン開発特化
- **[高度なAPIパターン](API_ADVANCED_PATTERNS_ja_JP.md)** - 複雑な操作パターン
- **[完全なAPIリファレンス](API_ja_JP.md)** - 全エンドポイントの詳細

### ベストプラクティス

- **[プラグイン開発ベストプラクティス](PLUGIN_BEST_PRACTICES_ja_JP.md)** - 品質の高いプラグイン開発

### コミュニティ

- [SiYuan公式フォーラム](https://liuyun.io/)
- [プラグイン開発ガイド](https://github.com/siyuan-note/petal)
- [サンプルプラグイン](https://github.com/siyuan-note/plugin-sample)

---

*このリファレンスは定期的に更新されます。新しいAPIエンドポイントが追加された場合、適切なカテゴリに分類して追加していきます。*
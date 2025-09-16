# SiYuan プラグイン開発者向け API ガイド

SiYuanプラグインを開発する際に最もよく使用されるAPIエンドポイントと使用例をまとめたガイドです。

**対象読者**: SiYuanプラグインを開発する開発者

**関連ドキュメント**:
- [完全なAPIリファレンス](API_ja_JP.md) - 全てのAPIエンドポイント
- [初心者向けAPIガイド](API_BEGINNERS_ja_JP.md) - API使用の基本
- [高度なAPIパターン](API_ADVANCED_PATTERNS_ja_JP.md) - 複雑な操作パターン

---

## 目次

- [基本設定](#基本設定)
- [ブロック操作](#ブロック操作)
- [ドキュメント操作](#ドキュメント操作)
- [ノートブック操作](#ノートブック操作)
- [データベース操作（SQL）](#データベース操作sql)
- [ファイル・アセット管理](#ファイルアセット管理)
- [UI更新・同期](#ui更新同期)
- [プラグイン間連携](#プラグイン間連携)
- [実用的なコード例](#実用的なコード例)

---

## 基本設定

### APIクライアントの初期化

```typescript
interface SiYuanAPI {
  baseURL: string;
  token: string;
}

const api: SiYuanAPI = {
  baseURL: 'http://localhost:6806/api',
  token: window.siyuan.config.api.token  // プラグイン内でのトークン取得
};

async function callAPI(endpoint: string, data: any) {
  const response = await fetch(`${api.baseURL}${endpoint}`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Token ${api.token}`
    },
    body: JSON.stringify(data)
  });
  return await response.json();
}
```

---

## ブロック操作

プラグイン開発で最も重要な機能の一つです。

### ブロックの挿入

新しいコンテンツブロックを作成します。

```typescript
// マークダウンブロックの挿入
async function insertMarkdownBlock(parentID: string, markdown: string) {
  return await callAPI('/block/insertBlock', {
    dataType: 'markdown',
    data: markdown,
    parentID: parentID
  });
}

// 使用例
await insertMarkdownBlock('20210428212840-8rqwn5o', '# 新しいセクション');
```

### ブロックの更新

既存のブロックを更新します。

```typescript
async function updateBlock(id: string, markdown: string) {
  return await callAPI('/block/updateBlock', {
    dataType: 'markdown',
    data: markdown,
    id: id
  });
}
```

### ブロック情報の取得

```typescript
async function getBlockInfo(id: string) {
  return await callAPI('/block/getBlockInfo', {
    id: id
  });
}
```

### 子ブロックの取得

```typescript
async function getChildBlocks(parentID: string) {
  return await callAPI('/block/getChildBlocks', {
    id: parentID
  });
}
```

---

## ドキュメント操作

### 新しいドキュメントの作成

```typescript
async function createDocumentWithMarkdown(notebook: string, path: string, markdown: string) {
  return await callAPI('/filetree/createDocWithMd', {
    notebook: notebook,
    path: path,
    markdown: markdown
  });
}

// 使用例
await createDocumentWithMarkdown(
  '20210428212840-8rqwn5o', 
  '/Daily Notes/2023-12-01.md',
  '# 2023年12月1日\n\n今日のタスク:\n- [ ] プラグイン開発'
);
```

### ドキュメントの移動

```typescript
async function moveDocument(fromNotebook: string, fromPath: string, toNotebook: string, toPath: string) {
  return await callAPI('/filetree/moveDoc', {
    fromNotebook: fromNotebook,
    fromPath: fromPath,
    toNotebook: toNotebook,
    toPath: toPath
  });
}
```

---

## ノートブック操作

### ノートブック一覧の取得

```typescript
async function listNotebooks() {
  return await callAPI('/notebook/lsNotebooks', {});
}

// 使用例
const notebooks = await listNotebooks();
console.log(notebooks.data.notebooks);
```

### ノートブックの作成

```typescript
async function createNotebook(name: string) {
  return await callAPI('/notebook/createNotebook', {
    name: name
  });
}
```

---

## データベース操作（SQL）

### SQLクエリの実行

プラグインでのデータ分析や高度な操作に使用します。

```typescript
async function executeSQLQuery(sql: string) {
  return await callAPI('/query/sql', {
    stmt: sql
  });
}

// 使用例：すべてのドキュメントタイトルを取得
const documentTitles = await executeSQLQuery(`
  SELECT id, content 
  FROM blocks 
  WHERE type = 'd' 
  ORDER BY created DESC 
  LIMIT 10
`);
```

### よく使用するSQL例

```typescript
// 特定の日付範囲のブロックを検索
const recentBlocks = await executeSQLQuery(`
  SELECT * FROM blocks 
  WHERE created > '20231201000000' 
  AND type != 'd'
  ORDER BY created DESC
`);

// タグを持つブロックの検索
const taggedBlocks = await executeSQLQuery(`
  SELECT b.*, a.value as tag_value
  FROM blocks b
  JOIN attributes a ON b.id = a.block_id
  WHERE a.name = 'custom-tag'
`);

// ブロック参照の統計
const referenceStats = await executeSQLQuery(`
  SELECT def_block_id, COUNT(*) as reference_count
  FROM refs
  GROUP BY def_block_id
  ORDER BY reference_count DESC
`);
```

---

## ファイル・アセット管理

### ファイルのアップロード

```typescript
async function uploadAsset(file: File) {
  const formData = new FormData();
  formData.append('file[]', file);
  
  const response = await fetch(`${api.baseURL}/asset/upload`, {
    method: 'POST',
    headers: {
      'Authorization': `Token ${api.token}`
    },
    body: formData
  });
  
  return await response.json();
}
```

### ファイルの取得

```typescript
async function getFile(path: string) {
  return await callAPI('/file/getFile', {
    path: path
  });
}
```

---

## UI更新・同期

プラグインがUIを変更した後の同期処理です。

### UIリロード

```typescript
async function reloadUI() {
  return await callAPI('/system/reload', {});
}
```

### ファイルツリーの更新

```typescript
async function reloadFiletree() {
  return await callAPI('/filetree/refreshFiletree', {});
}
```

### エディターの更新

```typescript
async function reloadProtyle() {
  return await callAPI('/system/reloadProtyle', {});
}
```

---

## プラグイン間連携

### メッセージの送信

```typescript
async function sendBroadcastMessage(channel: string, message: any) {
  return await callAPI('/broadcast/postMessage', {
    channel: channel,
    message: message
  });
}

// 使用例
await sendBroadcastMessage('my-plugin-channel', {
  action: 'data-updated',
  data: { blockId: '20210428212840-8rqwn5o' }
});
```

### 通知の表示

```typescript
async function showNotification(message: string, timeout: number = 3000) {
  return await callAPI('/notification/pushMsg', {
    msg: message,
    timeout: timeout
  });
}
```

---

## 実用的なコード例

### 1. 自動バックアップ機能

```typescript
class AutoBackupPlugin {
  private intervalId: number;

  async startAutoBackup() {
    this.intervalId = setInterval(async () => {
      try {
        await callAPI('/export/createSnapshot', {
          name: `自動バックアップ_${new Date().toISOString()}`
        });
        await showNotification('自動バックアップが完了しました');
      } catch (error) {
        console.error('バックアップエラー:', error);
      }
    }, 30 * 60 * 1000); // 30分間隔
  }

  stopAutoBackup() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
    }
  }
}
```

### 2. ブロック統計プラグイン

```typescript
class BlockStatsPlugin {
  async getBlockStatistics() {
    const stats = await executeSQLQuery(`
      SELECT 
        type,
        COUNT(*) as count,
        AVG(LENGTH(content)) as avg_length
      FROM blocks 
      WHERE type != 'd'
      GROUP BY type
    `);

    return stats.data;
  }

  async generateStatsReport() {
    const stats = await this.getBlockStatistics();
    const reportMarkdown = this.formatStatsAsMarkdown(stats);
    
    // レポートドキュメントを作成
    await createDocumentWithMarkdown(
      await this.getCurrentNotebook(),
      `/統計レポート_${new Date().toISOString().slice(0, 10)}.md`,
      reportMarkdown
    );
  }

  private formatStatsAsMarkdown(stats: any[]): string {
    let markdown = '# ブロック統計レポート\n\n';
    markdown += '| タイプ | 数量 | 平均文字数 |\n';
    markdown += '|--------|------|------------|\n';
    
    stats.forEach(stat => {
      markdown += `| ${stat.type} | ${stat.count} | ${Math.round(stat.avg_length)} |\n`;
    });
    
    return markdown;
  }
}
```

### 3. スマート検索プラグイン

```typescript
class SmartSearchPlugin {
  async searchWithContext(query: string) {
    // フルテキスト検索
    const fullTextResults = await callAPI('/search/fullTextSearchBlock', {
      query: query,
      types: {
        mathBlock: true,
        table: true,
        blockquote: true,
        superBlock: true,
        paragraph: true,
        document: true,
        heading: true,
        list: true,
        listItem: true,
        codeBlock: true,
        htmlBlock: true
      }
    });

    // 関連ブロックも取得
    const enhancedResults = await Promise.all(
      fullTextResults.data.blocks.map(async (block: any) => {
        const childBlocks = await getChildBlocks(block.id);
        return {
          ...block,
          children: childBlocks.data
        };
      })
    );

    return enhancedResults;
  }
}
```

### 4. タグ管理プラグイン

```typescript
class TagManagerPlugin {
  async getAllTags() {
    return await executeSQLQuery(`
      SELECT DISTINCT name, value 
      FROM attributes 
      WHERE name LIKE 'custom-%'
      ORDER BY name, value
    `);
  }

  async getBlocksByTag(tagName: string, tagValue?: string) {
    let sql = `
      SELECT b.*, a.value as tag_value
      FROM blocks b
      JOIN attributes a ON b.id = a.block_id
      WHERE a.name = ?
    `;
    
    const params = [tagName];
    if (tagValue) {
      sql += ' AND a.value = ?';
      params.push(tagValue);
    }
    
    return await executeSQLQuery(sql);
  }

  async addTagToBlock(blockId: string, tagName: string, tagValue: string) {
    return await callAPI('/attr/setBlockAttrs', {
      id: blockId,
      attrs: {
        [tagName]: tagValue
      }
    });
  }
}
```

---

## エラーハンドリングのベストプラクティス

```typescript
async function safeAPICall<T>(apiCall: () => Promise<T>): Promise<T | null> {
  try {
    return await apiCall();
  } catch (error) {
    console.error('API呼び出しエラー:', error);
    await showNotification('操作中にエラーが発生しました', 5000);
    return null;
  }
}

// 使用例
const notebooks = await safeAPICall(() => listNotebooks());
if (notebooks) {
  // 正常処理
  console.log(notebooks.data);
} else {
  // エラー処理
  console.log('ノートブック一覧の取得に失敗しました');
}
```

---

## パフォーマンス最適化のヒント

1. **バッチ処理の使用**:
```typescript
// ❌ 悪い例：個別にAPI呼び出し
for (const blockId of blockIds) {
  await updateBlock(blockId, newContent);
}

// ✅ 良い例：SQLトランザクションを使用
await executeSQLQuery(`
  UPDATE blocks SET content = ? WHERE id IN (${blockIds.map(() => '?').join(',')})
`, [newContent, ...blockIds]);
```

2. **キャッシュの活用**:
```typescript
class CachedNotebookService {
  private cache = new Map<string, any>();
  
  async getNotebook(id: string) {
    if (this.cache.has(id)) {
      return this.cache.get(id);
    }
    
    const notebook = await callAPI('/notebook/getNotebookConf', { notebook: id });
    this.cache.set(id, notebook);
    return notebook;
  }
}
```

---

## 参考リンク

- [SiYuan プラグイン開発ベストプラクティス](PLUGIN_BEST_PRACTICES_ja_JP.md)
- [SiYuan 高度なプラグインパターン](ADVANCED_PLUGIN_PATTERNS_ja_JP.md)
- [SiYuan プラグインショーケース](PLUGIN_SHOWCASE_ja_JP.md)
- [完全なAPIリファレンス](API_ja_JP.md)

---

*このガイドは定期的に更新されます。新しいAPIエンドポイントやベストプラクティスがあれば追加していきます。*
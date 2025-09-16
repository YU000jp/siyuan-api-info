# SiYuan API 未記載エンドポイント補完

メインのAPIドキュメントに記載されていない、または詳細が不足しているAPIエンドポイントの補完ドキュメントです。

**対象読者**: 完全なAPI機能を必要とする上級開発者

**注意**: これらのエンドポイントは実装されているが、メインドキュメントでは詳細が不足している可能性があります。

---

## 目次

- [AI統合・ChatGPT関連](#ai統合chatgpt関連)
- [ブックマーク機能](#ブックマーク機能)
- [クリップボード操作](#クリップボード操作)
- [グラフビュー関連](#グラフビュー関連)
- [インボックス機能](#インボックス機能)
- [プラグイン・Petal API](#プラグインpetal-api)
- [参照・リンク管理](#参照リンク管理)
- [リポジトリ管理](#リポジトリ管理)
- [間隔反復（Riff）システム](#間隔反復riffシステム)
- [ストレージ操作](#ストレージ操作)
- [タグ管理](#タグ管理)
- [UI制御](#ui制御)
- [拡張機能管理](#拡張機能管理)

---

## AI統合・ChatGPT関連

### ChatGPT統合

| エンドポイント | 説明 | リクエスト例 |
|---------------|------|-------------|
| `POST /ai/chatGPT` | ChatGPTとの連携 | `{ "prompt": "要約してください", "content": "..." }` |
| `POST /ai/chatGPTWithAction` | アクション付きChatGPT連携 | `{ "prompt": "...", "action": "summarize" }` |

### 使用例

```javascript
// ChatGPTで内容を要約
async function summarizeWithChatGPT(content) {
  return await callAPI('/ai/chatGPT', {
    prompt: '以下の内容を3つのポイントに要約してください',
    content: content,
    model: 'gpt-3.5-turbo'
  });
}

// アクション付きでChatGPTを実行
async function chatGPTWithAction(prompt, action, blockId) {
  return await callAPI('/ai/chatGPTWithAction', {
    prompt: prompt,
    action: action,
    id: blockId
  });
}
```

---

## ブックマーク機能

### ブックマーク管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /bookmark/getBookmark` | ブックマーク取得 | 保存されたブックマーク一覧 |
| `POST /bookmark/renameBookmark` | ブックマーク名変更 | ブックマークの管理 |
| `POST /bookmark/removeBookmark` | ブックマーク削除 | 不要なブックマークの削除 |

### 使用例

```javascript
// ブックマーク一覧を取得
async function getBookmarks() {
  return await callAPI('/bookmark/getBookmark', {});
}

// ブックマークの名前を変更
async function renameBookmark(bookmarkId, newName) {
  return await callAPI('/bookmark/renameBookmark', {
    id: bookmarkId,
    name: newName
  });
}

// ブックマークを削除
async function removeBookmark(bookmarkId) {
  return await callAPI('/bookmark/removeBookmark', {
    id: bookmarkId
  });
}
```

---

## クリップボード操作

### クリップボード管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /clipboard/readFilePaths` | クリップボードファイルパス読取 | ドラッグ&ドロップ処理 |

### 使用例

```javascript
// クリップボードからファイルパスを読み取り
async function readClipboardFilePaths() {
  return await callAPI('/clipboard/readFilePaths', {});
}
```

---

## グラフビュー関連

### グラフ表示・分析

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /graph/getGraph` | グラフデータ取得 | ノード間関係の可視化 |
| `POST /graph/getLocalGraph` | ローカルグラフ取得 | 特定ノード周辺のグラフ |
| `POST /graph/resetGraph` | グラフリセット | グラフ表示の初期化 |

### 使用例

```javascript
// 全体のグラフデータを取得
async function getFullGraph() {
  return await callAPI('/graph/getGraph', {
    k: "", // 検索キーワード（空=全体）
    conf: {
      dailyNote: false,
      type: {
        blockquote: true,
        codeBlock: true,
        document: true,
        heading: true,
        list: true,
        listItem: true,
        mathBlock: true,
        paragraph: true,
        super: true,
        table: true
      }
    }
  });
}

// 特定ブロック周辺のローカルグラフを取得
async function getLocalGraph(blockId) {
  return await callAPI('/graph/getLocalGraph', {
    id: blockId,
    conf: {
      dailyNote: false
    }
  });
}
```

---

## インボックス機能

### インボックス管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /inbox/getShorthands` | ショートハンド一覧取得 | クイック入力機能 |

### 使用例

```javascript
// ショートハンド一覧を取得
async function getInboxShorthands() {
  return await callAPI('/inbox/getShorthands', {});
}
```

---

## プラグイン・Petal API

### プラグインサポート

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /petal/loadPlugin` | プラグイン読み込み | 動的プラグイン管理 |
| `POST /petal/enablePlugin` | プラグイン有効化 | プラグイン状態管理 |
| `POST /petal/disablePlugin` | プラグイン無効化 | プラグイン状態管理 |
| `POST /petal/uninstallPlugin` | プラグインアンインストール | プラグイン削除 |
| `POST /petal/setPluginConf` | プラグイン設定保存 | プラグイン設定管理 |

### 使用例

```javascript
// プラグインを動的に読み込み
async function loadPlugin(pluginName) {
  return await callAPI('/petal/loadPlugin', {
    name: pluginName
  });
}

// プラグインを有効化
async function enablePlugin(pluginName) {
  return await callAPI('/petal/enablePlugin', {
    name: pluginName,
    enabled: true
  });
}

// プラグイン設定を保存
async function setPluginConfig(pluginName, config) {
  return await callAPI('/petal/setPluginConf', {
    name: pluginName,
    conf: config
  });
}
```

---

## 参照・リンク管理

### 参照関係操作

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /ref/getBacklinkDoc` | バックリンクドキュメント取得 | 逆参照の確認 |
| `POST /ref/getBacklink2` | 高度なバックリンク取得 | 詳細な参照関係 |
| `POST /ref/getRef` | 参照情報取得 | 参照データの分析 |
| `POST /ref/refreshBacklink` | バックリンク更新 | 参照関係の再計算 |

### 使用例

```javascript
// ドキュメントのバックリンクを取得
async function getDocumentBacklinks(docId) {
  return await callAPI('/ref/getBacklinkDoc', {
    id: docId,
    k: "", // 検索キーワード
    mk: "" // マーク
  });
}

// 詳細なバックリンク情報を取得
async function getAdvancedBacklinks(blockId, keywords = "") {
  return await callAPI('/ref/getBacklink2', {
    id: blockId,
    k: keywords,
    beforeLen: 512,
    afterLen: 512
  });
}

// 参照情報を取得
async function getReferences(blockId) {
  return await callAPI('/ref/getRef', {
    id: blockId,
    k: ""
  });
}
```

---

## リポジトリ管理

### データリポジトリ操作

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /repo/createSnapshot` | リポジトリスナップショット作成 | バージョン管理 |
| `POST /repo/checkoutRepo` | リポジトリチェックアウト | バージョン切り替え |
| `POST /repo/getRepoFile` | リポジトリファイル取得 | 履歴ファイルアクセス |
| `POST /repo/diffRepoSnapshots` | スナップショット比較 | バージョン間差分表示 |
| `POST /repo/indexRepo` | リポジトリインデックス | 検索インデックス更新 |

### 使用例

```javascript
// リポジトリのスナップショットを作成
async function createRepoSnapshot(memo = "") {
  return await callAPI('/repo/createSnapshot', {
    memo: memo
  });
}

// 特定のスナップショットにチェックアウト
async function checkoutSnapshot(snapshotId) {
  return await callAPI('/repo/checkoutRepo', {
    id: snapshotId
  });
}

// 2つのスナップショット間の差分を取得
async function diffSnapshots(leftId, rightId) {
  return await callAPI('/repo/diffRepoSnapshots', {
    left: leftId,
    right: rightId
  });
}
```

---

## 間隔反復（Riff）システム

### フラッシュカード・学習管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /riff/getFlashcards` | フラッシュカード取得 | 学習カードの管理 |
| `POST /riff/reviewFlashcard` | フラッシュカード復習 | 学習進捗の記録 |
| `POST /riff/skipReviewFlashcard` | 復習スキップ | 学習のスキップ |
| `POST /riff/getRiffDecks` | デッキ一覧取得 | 学習デッキの管理 |
| `POST /riff/createRiffDeck` | デッキ作成 | 新しい学習セット |
| `POST /riff/removeRiffDeck` | デッキ削除 | 不要なデッキの削除 |

### 使用例

```javascript
// 復習予定のフラッシュカードを取得
async function getReviewFlashcards(deckId) {
  return await callAPI('/riff/getFlashcards', {
    deckID: deckId
  });
}

// フラッシュカードの復習結果を記録
async function reviewFlashcard(deckId, cardId, rating) {
  return await callAPI('/riff/reviewFlashcard', {
    deckID: deckId,
    cardID: cardId,
    rating: rating // 1-4 (Again, Hard, Good, Easy)
  });
}

// 新しいフラッシュカードデッキを作成
async function createFlashcardDeck(name, blockIds) {
  return await callAPI('/riff/createRiffDeck', {
    name: name,
    blockIDs: blockIds
  });
}

// デッキ一覧を取得
async function getRiffDecks() {
  return await callAPI('/riff/getRiffDecks', {});
}
```

---

## ストレージ操作

### 低レベルストレージアクセス

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /storage/getRecentDocs` | 最近のドキュメント取得 | 履歴表示 |
| `POST /storage/setLocalStorage` | ローカルストレージ設定 | クライアント側データ保存 |
| `POST /storage/getLocalStorage` | ローカルストレージ取得 | クライアント側データ取得 |

### 使用例

```javascript
// 最近開いたドキュメント一覧を取得
async function getRecentDocuments() {
  return await callAPI('/storage/getRecentDocs', {});
}

// ローカルストレージにデータを保存
async function setLocalStorageValue(key, value) {
  return await callAPI('/storage/setLocalStorage', {
    app: window.siyuan.config.system.appDir,
    key: key,
    val: value
  });
}

// ローカルストレージからデータを取得
async function getLocalStorageValue(key) {
  return await callAPI('/storage/getLocalStorage', {
    app: window.siyuan.config.system.appDir,
    key: key
  });
}
```

---

## タグ管理

### タグ操作

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /tag/getTag` | タグ一覧取得 | タグの管理・表示 |
| `POST /tag/renameTag` | タグ名変更 | タグの整理 |
| `POST /tag/removeTag` | タグ削除 | 不要なタグの削除 |

### 使用例

```javascript
// すべてのタグを取得
async function getAllTags() {
  return await callAPI('/tag/getTag', {
    k: "" // 検索キーワード（空=全て）
  });
}

// タグの名前を変更
async function renameTag(oldName, newName) {
  return await callAPI('/tag/renameTag', {
    oldLabel: oldName,
    newLabel: newName
  });
}

// タグを削除
async function removeTag(tagName) {
  return await callAPI('/tag/removeTag', {
    label: tagName
  });
}
```

---

## UI制御

### 高度なUI操作

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /ui/getOpenedTab` | 開いているタブ取得 | タブ状態の確認 |
| `POST /ui/closeTab` | タブを閉じる | タブ管理 |
| `POST /ui/closeAllTabs` | 全タブを閉じる | UI初期化 |
| `POST /ui/moveTab` | タブ移動 | タブ順序変更 |

### 使用例

```javascript
// 現在開いているタブを取得
async function getOpenTabs() {
  return await callAPI('/ui/getOpenedTab', {});
}

// 特定のタブを閉じる
async function closeTab(tabId) {
  return await callAPI('/ui/closeTab', {
    id: tabId
  });
}

// すべてのタブを閉じる
async function closeAllTabs() {
  return await callAPI('/ui/closeAllTabs', {});
}
```

---

## 拡張機能管理

### スニペット・拡張管理

| エンドポイント | 説明 | 用途 |
|---------------|------|------|
| `POST /extension/getSnippet` | スニペット取得 | コードスニペット管理 |
| `POST /extension/setSnippet` | スニペット保存 | カスタムスニペット登録 |
| `POST /extension/removeSnippet` | スニペット削除 | 不要スニペット削除 |

### 使用例

```javascript
// スニペット一覧を取得
async function getSnippets(type = "css") {
  return await callAPI('/extension/getSnippet', {
    type: type // "css" または "js"
  });
}

// 新しいスニペットを保存
async function saveSnippet(type, name, content) {
  return await callAPI('/extension/setSnippet', {
    type: type,
    name: name,
    content: content,
    enabled: true
  });
}

// スニペットを削除
async function removeSnippet(type, name) {
  return await callAPI('/extension/removeSnippet', {
    type: type,
    name: name
  });
}
```

---

## 統合使用例

### 高度なワークフロー例

```javascript
// 1. AIを活用した自動要約＋タグ付けワークフロー
async function aiSummaryAndTagWorkflow(blockId) {
  // ブロック内容を取得
  const blockInfo = await callAPI('/block/getBlockInfo', { id: blockId });
  
  // AIで要約を生成
  const summary = await callAPI('/ai/chatGPT', {
    prompt: '以下の内容を要約し、適切なタグを3つ提案してください',
    content: blockInfo.data.content
  });
  
  // 要約を新しいブロックとして追加
  await callAPI('/block/insertBlock', {
    dataType: 'markdown',
    data: `**AI要約**: ${summary.data.summary}`,
    parentID: blockId
  });
  
  // 提案されたタグを属性として設定
  const tags = summary.data.suggested_tags;
  await callAPI('/attr/setBlockAttrs', {
    id: blockId,
    attrs: {
      'ai-tags': tags.join(', '),
      'ai-summary-date': new Date().toISOString()
    }
  });
}

// 2. 学習カード自動生成ワークフロー
async function createStudyCardsFromDocument(docId) {
  // ドキュメントの重要なブロックを抽出
  const blocks = await callAPI('/query/sql', {
    stmt: `
      SELECT id, content 
      FROM blocks 
      WHERE root_id = ? 
        AND type IN ('h', 'p') 
        AND LENGTH(content) > 50 
        AND LENGTH(content) < 500
    `,
    args: [docId]
  });
  
  // 学習デッキを作成
  const deck = await callAPI('/riff/createRiffDeck', {
    name: `学習カード_${Date.now()}`,
    blockIDs: blocks.data.map(b => b.id)
  });
  
  return deck.data;
}

// 3. 参照関係の可視化と分析
async function analyzeDocumentConnections(docId) {
  // バックリンクを取得
  const backlinks = await callAPI('/ref/getBacklinkDoc', {
    id: docId,
    k: ""
  });
  
  // ローカルグラフを取得
  const localGraph = await callAPI('/graph/getLocalGraph', {
    id: docId
  });
  
  // 参照統計を計算
  const referenceStats = await callAPI('/query/sql', {
    stmt: `
      SELECT 
        COUNT(CASE WHEN block_id = ? THEN 1 END) as outgoing_refs,
        COUNT(CASE WHEN def_block_id = ? THEN 1 END) as incoming_refs
      FROM refs
    `,
    args: [docId, docId]
  });
  
  return {
    backlinks: backlinks.data,
    graph: localGraph.data,
    stats: referenceStats.data[0]
  };
}
```

---

## 注意事項

1. **APIバージョン**: これらのエンドポイントは実装バージョンによって利用可能性が異なる場合があります。
2. **実験的機能**: 一部のエンドポイントは実験的機能である可能性があります。
3. **パフォーマンス**: 高度なAPIを使用する際は、パフォーマンスへの影響を考慮してください。
4. **エラーハンドリング**: 新しいAPIエンドポイントを使用する際は、適切なエラーハンドリングを実装してください。

---

## 関連ドキュメント

- **[初心者向けAPIガイド](API_BEGINNERS_ja_JP.md)** - 基本的なAPI使用方法
- **[プラグイン開発者向けガイド](API_PLUGIN_DEVELOPERS_ja_JP.md)** - プラグイン開発特化
- **[高度なAPIパターン](API_ADVANCED_PATTERNS_ja_JP.md)** - 複雑な操作パターン
- **[カテゴリ別APIリファレンス](API_REFERENCE_BY_CATEGORY_ja_JP.md)** - 機能別API一覧
- **[完全なAPIリファレンス](API_ja_JP.md)** - 全エンドポイントの詳細

---

*このドキュメントは、実装されているAPIエンドポイントの調査に基づいて作成されています。実際の使用前に、最新のSiYuanバージョンでの動作確認をお勧めします。*
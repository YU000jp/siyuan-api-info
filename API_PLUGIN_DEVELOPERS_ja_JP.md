# SiYuan プラグイン開発者向け API ガイド

SiYuanプラグインを開発する際に必要なAPIエンドポイント、実装パターン、ベストプラクティスを網羅した完全ガイドです。

**対象読者**: SiYuanプラグインを開発する開発者（初級〜上級）

**前提知識**: 
- JavaScript/TypeScript の基本的な知識
- SiYuan の基本的な使用経験
- プラグイン開発の基本概念

**関連ドキュメント**:
- [プラグイン開発ベストプラクティス](PLUGIN_BEST_PRACTICES_ja_JP.md) - 開発手法とコード品質
- [プラグインショーケース](PLUGIN_SHOWCASE_ja_JP.md) - 優秀なプラグイン事例
- [完全なAPIリファレンス](API_ja_JP.md) - 全てのAPIエンドポイント
- [初心者向けAPIガイド](API_BEGINNERS_ja_JP.md) - API使用の基本

---

## 📋 目次

### 🚀 スタートアップ
- [開発環境のセットアップ](#開発環境のセットアップ)
- [プラグインプロジェクトの作成](#プラグインプロジェクトの作成)
- [基本設定とAPI初期化](#基本設定とapi初期化)

### 🔧 コア機能開発
- [ブロック操作](#ブロック操作)
- [ドキュメント操作](#ドキュメント操作)
- [ノートブック操作](#ノートブック操作)
- [データベース操作（SQL）](#データベース操作sql)
- [検索機能の実装](#検索機能の実装)

### 🎨 UI・インタラクション
- [UI更新・同期](#ui更新同期)
- [カスタムダイアログとパネル](#カスタムダイアログとパネル)
- [コンテキストメニューの拡張](#コンテキストメニューの拡張)
- [キーボードショートカット](#キーボードショートカット)

### 📁 ファイル・リソース管理
- [ファイル・アセット管理](#ファイルアセット管理)
- [設定の保存と管理](#設定の保存と管理)
- [多言語対応（i18n）](#多言語対応i18n)

### 🔗 プラグイン統合
- [プラグイン間連携](#プラグイン間連携)
- [イベントシステムの活用](#イベントシステムの活用)
- [外部サービス連携](#外部サービス連携)

### 📊 実践的な開発例
- [実用的なプラグイン例](#実用的なプラグイン例)
- [パフォーマンス最適化](#パフォーマンス最適化)
- [デバッグとテスト](#デバッグとテスト)
- [配布と更新](#配布と更新)

---

## 🚀 開発環境のセットアップ

### プラグイン開発に必要なツール

```bash
# Node.js と npm/pnpm をインストール
node --version  # v16 以上推奨
npm --version   # または pnpm --version

# TypeScript をグローバルにインストール
npm install -g typescript

# 推奨される開発ツール
npm install -g @typescript-eslint/eslint-plugin prettier
```

### SiYuan プラグイン開発環境の構築

```bash
# プラグインテンプレートをクローン
git clone https://github.com/siyuan-note/plugin-sample-vite-svelte my-plugin
cd my-plugin

# 依存関係をインストール
npm install

# 開発モードで起動（ホットリロード有効）
npm run dev
```

## 🏗️ プラグインプロジェクトの作成

### plugin.json の設定

```json
{
  "name": "my-awesome-plugin",
  "author": "Your Name",
  "url": "https://github.com/yourusername/my-awesome-plugin",
  "version": "1.0.0",
  "minAppVersion": "2.10.0",
  "backends": ["windows", "linux", "darwin"],
  "frontends": ["desktop", "mobile"],
  "displayName": {
    "en_US": "My Awesome Plugin",
    "ja_JP": "私の素晴らしいプラグイン",
    "zh_CN": "我的超棒插件"
  },
  "description": {
    "en_US": "An awesome plugin for SiYuan",
    "ja_JP": "SiYuan用の素晴らしいプラグイン",
    "zh_CN": "SiYuan的超棒插件"
  },
  "readme": {
    "en_US": "README.md",
    "ja_JP": "README_ja_JP.md",
    "zh_CN": "README_zh_CN.md"
  },
  "funding": {
    "openCollective": "",
    "patreon": "",
    "github": "",
    "custom": ["https://example.com/sponsor"]
  }
}
```

### TypeScript 設定の最適化

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"],
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"],
      "@types/*": ["types/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## 🔧 基本設定とAPI初期化

### プラグインクラスの基本構造

```typescript
import {
  Plugin,
  showMessage,
  confirm,
  Dialog,
  Menu,
  openTab,
  adaptHotkey,
  getFrontend,
  getBackend,
  IModel,
  Setting,
  fetchPost,
  IPluginDockTab
} from "siyuan";
import "./index.scss";

const STORAGE_NAME = "my-plugin-settings";

export default class MyPlugin extends Plugin {
  private isMobile: boolean;
  private settingUtils: Setting;
  private dockTab: IPluginDockTab;

  async onload() {
    // プラットフォーム検出
    this.isMobile = getFrontend() === "mobile";
    
    // 設定の初期化
    await this.initSettings();
    
    // UI要素の作成
    this.initUI();
    
    // イベントリスナーの登録
    this.initEventListeners();
    
    console.log(`プラグイン ${this.name} が読み込まれました`);
  }

  async onunload() {
    // クリーンアップ処理
    this.saveSettings();
    console.log(`プラグイン ${this.name} がアンロードされました`);
  }

  private async initSettings() {
    // 設定の初期化
    this.settingUtils = new Setting({
      confirmCallback: () => {
        this.saveSettings();
      }
    });
    
    await this.loadSettings();
  }

  private initUI() {
    // トップバーアイコンの追加
    const topBarElement = this.addTopBar({
      icon: "iconEmoji",
      title: "My Plugin",
      position: "right",
      callback: () => {
        this.showMainDialog();
      }
    });

    // ドックタブの追加
    this.dockTab = this.addDock({
      config: {
        position: "LeftBottom",
        size: { width: 200, height: 0 },
        icon: "iconEmoji",
        title: "My Plugin Dock",
      },
      data: {
        text: "This is my plugin dock"
      },
      type: "my-plugin-dock",
      init() {
        this.element.innerHTML = `<div class="my-plugin-dock">
          <div class="toolbar">
            <div class="toolbar__item">プラグインドック</div>
          </div>
          <div class="content">
            <!-- ドックの内容 -->
          </div>
        </div>`;
      }
    });
  }

  private initEventListeners() {
    // キーボードショートカットの登録
    this.addCommand({
      langKey: "showMainDialog",
      hotkey: "⌘⇧P",
      callback: () => {
        this.showMainDialog();
      }
    });

    // コンテキストメニューの追加
    this.eventBus.on("click-blockicon", ({ detail }) => {
      const menu = new Menu("my-plugin-menu");
      menu.addItem({
        icon: "iconEmoji",
        label: "My Plugin Action",
        click: () => {
          this.handleBlockAction(detail.blockElements[0]);
        }
      });
      menu.open({ x: detail.event.clientX, y: detail.event.clientY });
    });
  }
}
```

### APIクライアントの実装

```typescript
class SiYuanAPIClient {
  private baseURL: string;
  private token: string;

  constructor() {
    this.baseURL = '/api';
    // プラグイン内では認証トークンは自動で処理される
  }

  async request<T = any>(endpoint: string, data: any = {}): Promise<T> {
    try {
      const response = await fetchPost(endpoint, data);
      
      if (response.code !== 0) {
        throw new Error(`API Error: ${response.msg}`);
      }
      
      return response.data;
    } catch (error) {
      console.error(`API request failed: ${endpoint}`, error);
      throw error;
    }
  }

  // よく使用されるAPI呼び出しをラップ
  async getBlock(id: string) {
    return this.request('/block/getBlockInfo', { id });
  }

  async updateBlock(id: string, data: string, dataType: string = 'markdown') {
    return this.request('/block/updateBlock', { id, data, dataType });
  }

  async insertBlock(parentID: string, data: string, dataType: string = 'markdown') {
    return this.request('/block/insertBlock', { parentID, data, dataType });
  }

  async sql(stmt: string, args?: any[]) {
    return this.request('/query/sql', { stmt, args });
  }

  async getNotebooks() {
    return this.request('/notebook/lsNotebooks');
  }

  async search(query: string, types?: any) {
    return this.request('/search/fullTextSearchBlock', { 
      query, 
      types: types || {
        paragraph: true,
        heading: true,
        document: true
      }
    });
  }
}

// プラグイン内でAPIクライアントを使用
const apiClient = new SiYuanAPIClient();
```

---

## 🧱 ブロック操作

プラグイン開発で最も重要な機能の一つです。ブロックの作成、編集、削除、移動などの操作を実装できます。

### ブロック操作の基本クラス

```typescript
class BlockManager {
  constructor(private apiClient: SiYuanAPIClient) {}

  // ブロックの詳細情報を取得
  async getBlockInfo(blockId: string) {
    try {
      const blockInfo = await this.apiClient.getBlock(blockId);
      return {
        id: blockInfo.id,
        type: blockInfo.type,
        content: blockInfo.content,
        parentId: blockInfo.parent_id,
        rootId: blockInfo.root_id,
        created: new Date(parseInt(blockInfo.created)),
        updated: new Date(parseInt(blockInfo.updated))
      };
    } catch (error) {
      showMessage(`ブロック情報の取得に失敗しました: ${error.message}`, 3000, 'error');
      return null;
    }
  }

  // 新しいブロックを挿入
  async insertBlock(parentId: string, content: string, type: 'markdown' | 'dom' = 'markdown') {
    try {
      const result = await this.apiClient.insertBlock(parentId, content, type);
      showMessage('ブロックを追加しました', 2000, 'info');
      return result;
    } catch (error) {
      showMessage(`ブロックの追加に失敗しました: ${error.message}`, 3000, 'error');
      throw error;
    }
  }

  // ブロックの内容を更新
  async updateBlock(blockId: string, content: string) {
    try {
      await this.apiClient.updateBlock(blockId, content);
      showMessage('ブロックを更新しました', 2000, 'info');
    } catch (error) {
      showMessage(`ブロックの更新に失敗しました: ${error.message}`, 3000, 'error');
      throw error;
    }
  }

  // ブロックを削除
  async deleteBlock(blockId: string) {
    try {
      const confirmResult = await confirm(
        "ブロック削除",
        "このブロックを削除してもよろしいですか？",
        () => {
          return this.apiClient.request('/block/deleteBlock', { id: blockId });
        }
      );
      
      if (confirmResult) {
        showMessage('ブロックを削除しました', 2000, 'info');
      }
    } catch (error) {
      showMessage(`ブロックの削除に失敗しました: ${error.message}`, 3000, 'error');
    }
  }

  // 子ブロックを取得
  async getChildBlocks(parentId: string) {
    try {
      return await this.apiClient.request('/block/getChildBlocks', { id: parentId });
    } catch (error) {
      console.error('子ブロックの取得に失敗:', error);
      return { data: [] };
    }
  }

  // ブロックを移動
  async moveBlock(blockId: string, targetParentId: string, previousId?: string) {
    try {
      await this.apiClient.request('/block/moveBlock', {
        id: blockId,
        parentID: targetParentId,
        previousID: previousId
      });
      showMessage('ブロックを移動しました', 2000, 'info');
    } catch (error) {
      showMessage(`ブロックの移動に失敗しました: ${error.message}`, 3000, 'error');
    }
  }
}
```

### 実用的なブロック操作の例

```typescript
// プラグインでのブロック操作の実装例
class ContentProcessor {
  constructor(private blockManager: BlockManager) {}

  // マークダウンテーブルをブロックに変換
  async convertMarkdownTableToBlocks(tableMarkdown: string, parentId: string) {
    const lines = tableMarkdown.trim().split('\n');
    const headers = lines[0].split('|').map(h => h.trim()).filter(h => h);
    const rows = lines.slice(2); // ヘッダーと区切り線をスキップ

    // テーブルヘッダーをブロックとして挿入
    const headerContent = `## ${headers.join(' | ')}`;
    await this.blockManager.insertBlock(parentId, headerContent);

    // 各行をリストアイテムとして挿入
    for (const row of rows) {
      const cells = row.split('|').map(c => c.trim()).filter(c => c);
      const listContent = `- ${cells.join(' | ')}`;
      await this.blockManager.insertBlock(parentId, listContent);
    }
  }

  // ブロック内のリンクを自動変換
  async convertLinksInBlock(blockId: string) {
    const blockInfo = await this.blockManager.getBlockInfo(blockId);
    if (!blockInfo) return;

    const urlRegex = /(https?:\/\/[^\s]+)/g;
    const updatedContent = blockInfo.content.replace(urlRegex, '[$1]($1)');
    
    if (updatedContent !== blockInfo.content) {
      await this.blockManager.updateBlock(blockId, updatedContent);
    }
  }

  // ブロックに自動タグ付け
  async addAutoTags(blockId: string, keywords: string[]) {
    const blockInfo = await this.blockManager.getBlockInfo(blockId);
    if (!blockInfo) return;

    const matchedKeywords = keywords.filter(keyword => 
      blockInfo.content.toLowerCase().includes(keyword.toLowerCase())
    );

    if (matchedKeywords.length > 0) {
      const tags = matchedKeywords.map(tag => `#${tag}`).join(' ');
      const updatedContent = `${blockInfo.content}\n\n${tags}`;
      await this.blockManager.updateBlock(blockId, updatedContent);
    }
  }
}
```

## 📝 ドキュメント操作

ドキュメント（ページ）レベルでの操作を行うための機能実装です。

### ドキュメント管理クラス

```typescript
class DocumentManager {
  constructor(private apiClient: SiYuanAPIClient) {}

  // 新しいドキュメントを作成
  async createDocument(notebookId: string, path: string, title: string, content: string = '') {
    try {
      const markdown = `# ${title}\n\n${content}`;
      const result = await this.apiClient.request('/filetree/createDocWithMd', {
        notebook: notebookId,
        path: path,
        markdown: markdown
      });
      
      showMessage(`ドキュメント「${title}」を作成しました`, 2000, 'info');
      return result;
    } catch (error) {
      showMessage(`ドキュメント作成に失敗しました: ${error.message}`, 3000, 'error');
      throw error;
    }
  }

  // ドキュメントテンプレートを使用した作成
  async createFromTemplate(notebookId: string, templateId: string, variables: Record<string, string>) {
    try {
      // テンプレートを取得
      const template = await this.apiClient.request('/template/render', {
        id: templateId,
        path: "/",
        ...variables
      });

      const timestamp = new Date().toISOString().slice(0, 19).replace('T', '_');
      const path = `/generated_${timestamp}.md`;
      
      return await this.apiClient.request('/filetree/createDocWithMd', {
        notebook: notebookId,
        path: path,
        markdown: template
      });
    } catch (error) {
      showMessage(`テンプレートからの作成に失敗しました: ${error.message}`, 3000, 'error');
      throw error;
    }
  }

  // ドキュメントの一括操作
  async batchProcessDocuments(notebookId: string, processor: (docId: string) => Promise<void>) {
    try {
      // ノートブック内のすべてのドキュメントを取得
      const documents = await this.apiClient.sql(`
        SELECT id, content as title 
        FROM blocks 
        WHERE type = 'd' AND box = '${notebookId}'
        ORDER BY created DESC
      `);

      let processed = 0;
      for (const doc of documents) {
        try {
          await processor(doc.id);
          processed++;
        } catch (error) {
          console.error(`ドキュメント ${doc.id} の処理に失敗:`, error);
        }
      }

      showMessage(`${processed}件のドキュメントを処理しました`, 2000, 'info');
    } catch (error) {
      showMessage(`一括処理に失敗しました: ${error.message}`, 3000, 'error');
    }
  }

  // ドキュメントの統計情報を取得
  async getDocumentStats(docId: string) {
    try {
      const stats = await this.apiClient.sql(`
        SELECT 
          COUNT(*) as total_blocks,
          COUNT(CASE WHEN type = 'p' THEN 1 END) as paragraphs,
          COUNT(CASE WHEN type = 'h' THEN 1 END) as headings,
          COUNT(CASE WHEN type = 'l' THEN 1 END) as lists,
          SUM(LENGTH(content)) as total_chars
        FROM blocks 
        WHERE root_id = '${docId}' AND type != 'd'
      `);

      return stats[0];
    } catch (error) {
      console.error('ドキュメント統計の取得に失敗:', error);
      return null;
    }
  }
}
```

## 📚 ノートブック操作

ノートブック（ワークスペース）レベルでの管理機能です。

### ノートブック管理クラス

```typescript
class NotebookManager {
  constructor(private apiClient: SiYuanAPIClient) {}

  // ノートブック一覧の取得（詳細情報付き）
  async getNotebooksWithStats() {
    try {
      const notebooks = await this.apiClient.getNotebooks();
      
      // 各ノートブックの統計情報を取得
      const notebooksWithStats = await Promise.all(
        notebooks.notebooks.map(async (notebook: any) => {
          const stats = await this.getNotebookStats(notebook.id);
          return {
            ...notebook,
            stats
          };
        })
      );

      return notebooksWithStats;
    } catch (error) {
      console.error('ノートブック一覧の取得に失敗:', error);
      return [];
    }
  }

  // ノートブックの統計情報
  async getNotebookStats(notebookId: string) {
    try {
      const result = await this.apiClient.sql(`
        SELECT 
          COUNT(CASE WHEN type = 'd' THEN 1 END) as documents,
          COUNT(CASE WHEN type != 'd' THEN 1 END) as blocks,
          COUNT(CASE WHEN type = 'p' THEN 1 END) as paragraphs,
          MAX(updated) as last_updated
        FROM blocks 
        WHERE box = '${notebookId}'
      `);

      return result[0];
    } catch (error) {
      console.error('ノートブック統計の取得に失敗:', error);
      return {
        documents: 0,
        blocks: 0,
        paragraphs: 0,
        last_updated: null
      };
    }
  }

  // ノートブックの作成（高度な設定付き）
  async createNotebook(name: string, config?: NotebookConfig) {
    try {
      const result = await this.apiClient.request('/notebook/createNotebook', { name });
      
      // 作成後に設定を適用
      if (config) {
        await this.setNotebookConfig(result.notebook.id, config);
      }
      
      showMessage(`ノートブック「${name}」を作成しました`, 2000, 'info');
      return result;
    } catch (error) {
      showMessage(`ノートブック作成に失敗しました: ${error.message}`, 3000, 'error');
      throw error;
    }
  }

  // ノートブック設定の更新
  async setNotebookConfig(notebookId: string, config: NotebookConfig) {
    try {
      await this.apiClient.request('/notebook/setNotebookConf', {
        notebook: notebookId,
        conf: config
      });
    } catch (error) {
      console.error('ノートブック設定の更新に失敗:', error);
    }
  }

  // ノートブック内の検索
  async searchInNotebook(notebookId: string, query: string) {
    try {
      const results = await this.apiClient.search(query, {
        paragraph: true,
        heading: true,
        document: true
      });

      // 指定されたノートブックの結果のみフィルタ
      const filteredResults = results.blocks.filter((block: any) => 
        block.box === notebookId
      );

      return filteredResults;
    } catch (error) {
      console.error('ノートブック内検索に失敗:', error);
      return [];
    }
  }
}

interface NotebookConfig {
  name?: string;
  closed?: boolean;
  refCreateSavePath?: string;
  createDocNameTemplate?: string;
  dailyNoteSavePath?: string;
  dailyNoteTemplatePath?: string;
}
```

## 🗄️ データベース操作（SQL）

SiYuanの内部データベースを活用した高度なデータ操作です。

### SQL操作マネージャー

```typescript
class SQLManager {
  constructor(private apiClient: SiYuanAPIClient) {}

  // 安全なSQLクエリ実行
  async safeQuery(query: string, params: any[] = []) {
    try {
      // SQLインジェクション対策のため、パラメータ化クエリを推奨
      const result = await this.apiClient.sql(query, params);
      return result;
    } catch (error) {
      console.error('SQLクエリ実行エラー:', error);
      showMessage(`データベースエラー: ${error.message}`, 3000, 'error');
      return [];
    }
  }

  // よく使用するクエリテンプレート
  async getRecentDocuments(limit: number = 10) {
    return this.safeQuery(`
      SELECT id, content as title, created, updated, box
      FROM blocks 
      WHERE type = 'd' 
      ORDER BY updated DESC 
      LIMIT ?
    `, [limit]);
  }

  async getBlocksByType(type: string, notebookId?: string) {
    let query = `
      SELECT id, content, created, updated, parent_id, root_id
      FROM blocks 
      WHERE type = ?
    `;
    const params = [type];

    if (notebookId) {
      query += ' AND box = ?';
      params.push(notebookId);
    }

    query += ' ORDER BY created DESC';
    return this.safeQuery(query, params);
  }

  async getOrphanedBlocks() {
    return this.safeQuery(`
      SELECT b1.id, b1.content, b1.type
      FROM blocks b1
      LEFT JOIN blocks b2 ON b1.parent_id = b2.id
      WHERE b1.parent_id != '' 
        AND b2.id IS NULL 
        AND b1.type != 'd'
    `);
  }

  // 高度な統計クエリ
  async getContentStatistics() {
    return this.safeQuery(`
      SELECT 
        type,
        COUNT(*) as count,
        AVG(LENGTH(content)) as avg_length,
        MAX(LENGTH(content)) as max_length,
        MIN(LENGTH(content)) as min_length
      FROM blocks 
      WHERE type != 'd'
      GROUP BY type
      ORDER BY count DESC
    `);
  }

  // 参照関係の分析
  async getBacklinkAnalysis(blockId: string) {
    return this.safeQuery(`
      SELECT 
        b.id,
        b.content,
        b.type,
        COUNT(r.def_block_id) as reference_count
      FROM blocks b
      LEFT JOIN refs r ON b.id = r.block_id
      WHERE r.def_block_id = ?
      GROUP BY b.id, b.content, b.type
      ORDER BY reference_count DESC
    `, [blockId]);
  }

  // カスタム属性の検索
  async findBlocksWithAttribute(attributeName: string, attributeValue?: string) {
    let query = `
      SELECT b.id, b.content, b.type, a.value
      FROM blocks b
      JOIN attributes a ON b.id = a.block_id
      WHERE a.name = ?
    `;
    const params = [attributeName];

    if (attributeValue) {
      query += ' AND a.value = ?';
      params.push(attributeValue);
    }

    return this.safeQuery(query, params);
  }
}
```

## 🔍 検索機能の実装

プラグインに検索機能を統合するための実装パターンです。

### 検索エンジンクラス

```typescript
class SearchEngine {
  constructor(
    private apiClient: SiYuanAPIClient,
    private sqlManager: SQLManager
  ) {}

  // 統合検索機能
  async comprehensiveSearch(query: string, options: SearchOptions = {}) {
    const results = {
      documents: [],
      blocks: [],
      attributes: [],
      backlinks: []
    };

    try {
      // フルテキスト検索
      if (options.includeFullText !== false) {
        const fullTextResults = await this.apiClient.search(query, {
          paragraph: true,
          heading: true,
          document: true,
          codeBlock: options.includeCode || false,
          mathBlock: options.includeMath || false
        });
        results.blocks = fullTextResults.blocks;
      }

      // 属性検索
      if (options.includeAttributes) {
        const attributeResults = await this.sqlManager.safeQuery(`
          SELECT b.id, b.content, a.name, a.value
          FROM blocks b
          JOIN attributes a ON b.id = a.block_id
          WHERE a.value LIKE ?
        `, [`%${query}%`]);
        results.attributes = attributeResults;
      }

      // バックリンク検索
      if (options.includeBacklinks) {
        const backlinkResults = await this.searchBacklinks(query);
        results.backlinks = backlinkResults;
      }

      return results;
    } catch (error) {
      console.error('検索エラー:', error);
      showMessage(`検索に失敗しました: ${error.message}`, 3000, 'error');
      return results;
    }
  }

  // セマンティック検索の実装
  async semanticSearch(query: string, threshold: number = 0.7) {
    // NOTE: これは概念的な実装例です
    // 実際のセマンティック検索には外部AI APIが必要です
    try {
      // 1. クエリの意図を分析
      const intent = await this.analyzeSearchIntent(query);
      
      // 2. 意図に基づいて検索戦略を変更
      const searchStrategy = this.getSearchStrategy(intent);
      
      // 3. 複数の検索手法を組み合わせ
      const results = await this.executeSearchStrategy(query, searchStrategy);
      
      return results;
    } catch (error) {
      console.error('セマンティック検索エラー:', error);
      return [];
    }
  }

  private async searchBacklinks(query: string) {
    return this.sqlManager.safeQuery(`
      SELECT DISTINCT
        b.id,
        b.content,
        b.type,
        rb.content as referencing_content
      FROM blocks b
      JOIN refs r ON b.id = r.def_block_id
      JOIN blocks rb ON r.block_id = rb.id
      WHERE b.content LIKE ? OR rb.content LIKE ?
    `, [`%${query}%`, `%${query}%`]);
  }

  private async analyzeSearchIntent(query: string): Promise<SearchIntent> {
    const keywords = query.toLowerCase().split(/\s+/);
    
    return {
      type: this.determineSearchType(keywords),
      entities: this.extractEntities(keywords),
      timeFrame: this.extractTimeFrame(query),
      priority: this.calculatePriority(keywords)
    };
  }

  private determineSearchType(keywords: string[]): 'content' | 'structural' | 'temporal' | 'relational' {
    if (keywords.some(k => ['最近', '今日', '昨日', '先週'].includes(k))) {
      return 'temporal';
    }
    if (keywords.some(k => ['リンク', '参照', '関連'].includes(k))) {
      return 'relational';
    }
    if (keywords.some(k => ['見出し', 'タイトル', '構造'].includes(k))) {
      return 'structural';
    }
    return 'content';
  }

  private extractEntities(keywords: string[]): string[] {
    // 固有名詞や重要なキーワードを抽出
    return keywords.filter(k => k.length > 2);
  }

  private extractTimeFrame(query: string): TimeFrame | null {
    const timePatterns = {
      today: /今日|本日/,
      yesterday: /昨日/,
      thisWeek: /今週|この週/,
      lastWeek: /先週|前週/,
      thisMonth: /今月/,
      lastMonth: /先月|前月/
    };

    for (const [period, pattern] of Object.entries(timePatterns)) {
      if (pattern.test(query)) {
        return { period: period as any, query };
      }
    }
    return null;
  }

  private calculatePriority(keywords: string[]): number {
    // キーワードの重要度を計算
    const importantWords = ['重要', '緊急', '優先', 'TODO', 'タスク'];
    return keywords.filter(k => importantWords.includes(k)).length;
  }
}

interface SearchOptions {
  includeFullText?: boolean;
  includeAttributes?: boolean;
  includeBacklinks?: boolean;
  includeCode?: boolean;
  includeMath?: boolean;
  notebookId?: string;
  timeRange?: { start: Date; end: Date };
}

interface SearchIntent {
  type: 'content' | 'structural' | 'temporal' | 'relational';
  entities: string[];
  timeFrame: TimeFrame | null;
  priority: number;
}

interface TimeFrame {
  period: 'today' | 'yesterday' | 'thisWeek' | 'lastWeek' | 'thisMonth' | 'lastMonth';
  query: string;
}
```

## 🎨 UI更新・同期

プラグインがUIを変更した後の同期処理とカスタムUI要素の実装です。

### UI管理クラス

```typescript
class UIManager {
  constructor(private plugin: Plugin) {}

  // プロトタイプエディターの更新
  async refreshEditor(blockId?: string) {
    try {
      if (blockId) {
        // 特定のブロックのみ更新
        const blockElement = document.querySelector(`[data-node-id="${blockId}"]`);
        if (blockElement) {
          blockElement.dispatchEvent(new CustomEvent('refresh'));
        }
      } else {
        // エディター全体を更新
        await fetchPost('/system/reloadProtyle', {});
      }
    } catch (error) {
      console.error('エディター更新エラー:', error);
    }
  }

  // ファイルツリーの更新
  async refreshFileTree() {
    try {
      await fetchPost('/filetree/refreshFiletree', {});
      showMessage('ファイルツリーを更新しました', 1000, 'info');
    } catch (error) {
      console.error('ファイルツリー更新エラー:', error);
    }
  }

  // アウトラインの更新
  async refreshOutline() {
    try {
      await fetchPost('/outline/reload', {});
    } catch (error) {
      console.error('アウトライン更新エラー:', error);
    }
  }

  // カスタム通知の表示
  showCustomNotification(message: string, type: 'info' | 'success' | 'warning' | 'error' = 'info') {
    const icons = {
      info: '💡',
      success: '✅',
      warning: '⚠️',
      error: '❌'
    };

    showMessage(`${icons[type]} ${message}`, 3000, type);
  }

  // プログレスバー付きの操作
  async withProgress<T>(
    operation: (updateProgress: (percent: number, message?: string) => void) => Promise<T>,
    title: string = '処理中...'
  ): Promise<T> {
    let progressDialog: Dialog;
    
    try {
      // プログレスダイアログを作成
      progressDialog = new Dialog({
        title,
        content: `
          <div class="progress-container">
            <div class="progress-bar">
              <div class="progress-fill" style="width: 0%"></div>
            </div>
            <div class="progress-text">開始中...</div>
          </div>
        `,
        width: '400px',
        destroyCallback: () => {
          // クリーンアップ
        }
      });

      const updateProgress = (percent: number, message?: string) => {
        const progressFill = progressDialog.element.querySelector('.progress-fill') as HTMLElement;
        const progressText = progressDialog.element.querySelector('.progress-text') as HTMLElement;
        
        if (progressFill) {
          progressFill.style.width = `${Math.min(100, Math.max(0, percent))}%`;
        }
        if (progressText && message) {
          progressText.textContent = message;
        }
      };

      const result = await operation(updateProgress);
      progressDialog.destroy();
      return result;
    } catch (error) {
      if (progressDialog) {
        progressDialog.destroy();
      }
      throw error;
    }
  }
}
```

## 🎛️ カスタムダイアログとパネル

### ダイアログマネージャー

```typescript
class DialogManager {
  // 設定ダイアログの作成
  createSettingsDialog(settings: any, onSave: (newSettings: any) => void) {
    const dialog = new Dialog({
      title: "プラグイン設定",
      content: this.generateSettingsHTML(settings),
      width: "600px",
      height: "400px",
      destroyCallback: () => {
        // 設定を保存
        const formData = this.extractFormData(dialog.element);
        onSave(formData);
      }
    });

    this.attachSettingsEventListeners(dialog);
    return dialog;
  }

  private generateSettingsHTML(settings: any): string {
    return `
      <div class="config__panel">
        <div class="config__item">
          <label class="fn__flex">
            <span class="fn__flex-1">自動保存を有効にする</span>
            <input type="checkbox" class="b3-switch" id="autoSave" ${settings.autoSave ? 'checked' : ''}>
          </label>
        </div>
        <div class="config__item">
          <label class="fn__flex">
            <span class="fn__flex-1">通知音を再生する</span>
            <input type="checkbox" class="b3-switch" id="playSound" ${settings.playSound ? 'checked' : ''}>
          </label>
        </div>
        <div class="config__item">
          <label>タイムアウト時間（秒）</label>
          <input type="number" class="b3-text-field" id="timeout" value="${settings.timeout || 5}" min="1" max="60">
        </div>
        <div class="config__item">
          <label>APIエンドポイント</label>
          <input type="text" class="b3-text-field" id="apiEndpoint" value="${settings.apiEndpoint || ''}" placeholder="https://api.example.com">
        </div>
        <div class="config__item">
          <label>除外するファイルパターン（改行区切り）</label>
          <textarea class="b3-text-field" id="excludePatterns" rows="3">${(settings.excludePatterns || []).join('\n')}</textarea>
        </div>
      </div>
    `;
  }

  private attachSettingsEventListeners(dialog: Dialog) {
    const form = dialog.element;
    
    // リアルタイム検証
    form.addEventListener('input', (e) => {
      const target = e.target as HTMLInputElement;
      if (target.type === 'number') {
        this.validateNumberInput(target);
      } else if (target.type === 'url' || target.id === 'apiEndpoint') {
        this.validateURLInput(target);
      }
    });
  }

  private validateNumberInput(input: HTMLInputElement) {
    const value = parseInt(input.value);
    const min = parseInt(input.getAttribute('min') || '0');
    const max = parseInt(input.getAttribute('max') || '100');
    
    if (value < min || value > max) {
      input.classList.add('error');
    } else {
      input.classList.remove('error');
    }
  }

  private validateURLInput(input: HTMLInputElement) {
    try {
      if (input.value && !input.value.startsWith('http')) {
        input.classList.add('error');
      } else {
        input.classList.remove('error');
      }
    } catch {
      input.classList.add('error');
    }
  }

  // 確認ダイアログの作成
  async showConfirmDialog(title: string, message: string, dangerousAction: boolean = false): Promise<boolean> {
    return new Promise((resolve) => {
      const dialog = new Dialog({
        title,
        content: `
          <div class="b3-dialog__content">
            <div class="ft__breakword">${message}</div>
          </div>
          <div class="b3-dialog__action">
            <button class="b3-button b3-button--cancel">キャンセル</button>
            <div class="fn__space"></div>
            <button class="b3-button ${dangerousAction ? 'b3-button--danger' : 'b3-button--text'}">確認</button>
          </div>
        `,
        width: "400px",
        destroyCallback: () => resolve(false)
      });

      // ボタンイベントの設定
      const cancelBtn = dialog.element.querySelector('.b3-button--cancel');
      const confirmBtn = dialog.element.querySelector('.b3-button:not(.b3-button--cancel)');

      cancelBtn?.addEventListener('click', () => {
        dialog.destroy();
        resolve(false);
      });

      confirmBtn?.addEventListener('click', () => {
        dialog.destroy();
        resolve(true);
      });
    });
  }

  private extractFormData(element: HTMLElement): any {
    const formData: any = {};
    
    // チェックボックス
    element.querySelectorAll('input[type="checkbox"]').forEach((input: HTMLInputElement) => {
      formData[input.id] = input.checked;
    });

    // テキスト・数値入力
    element.querySelectorAll('input[type="text"], input[type="number"]').forEach((input: HTMLInputElement) => {
      formData[input.id] = input.type === 'number' ? parseInt(input.value) : input.value;
    });

    // テキストエリア
    element.querySelectorAll('textarea').forEach((textarea: HTMLTextAreaElement) => {
      if (textarea.id === 'excludePatterns') {
        formData[textarea.id] = textarea.value.split('\n').filter(line => line.trim());
      } else {
        formData[textarea.id] = textarea.value;
      }
    });

    return formData;
  }
}
```

## 📱 コンテキストメニューの拡張

```typescript
class ContextMenuManager {
  constructor(private plugin: Plugin) {
    this.initializeContextMenus();
  }

  private initializeContextMenus() {
    // ブロックアイコンのクリック時
    this.plugin.eventBus.on("click-blockicon", ({ detail }) => {
      this.showBlockContextMenu(detail);
    });

    // エディター内での右クリック
    this.plugin.eventBus.on("click-editorcontent", ({ detail }) => {
      this.showEditorContextMenu(detail);
    });
  }

  private showBlockContextMenu(detail: any) {
    const menu = new Menu("block-menu");
    
    // ブロック情報を表示
    menu.addItem({
      icon: "iconInfo",
      label: "ブロック情報",
      click: () => {
        this.showBlockInfo(detail.blockElements[0]);
      }
    });

    // ブロックをコピー
    menu.addItem({
      icon: "iconCopy",
      label: "ブロックをコピー",
      click: () => {
        this.copyBlockToClipboard(detail.blockElements[0]);
      }
    });

    // ブロックをエクスポート
    menu.addItem({
      icon: "iconExport",
      label: "Markdownでエクスポート",
      click: () => {
        this.exportBlockAsMarkdown(detail.blockElements[0]);
      }
    });

    menu.addSeparator();

    // AIで要約
    menu.addItem({
      icon: "iconAI",
      label: "AI要約を生成",
      click: async () => {
        await this.generateAISummary(detail.blockElements[0]);
      }
    });

    // カスタムタグ付け
    menu.addItem({
      icon: "iconTags",
      label: "タグを追加",
      submenu: this.createTagSubmenu(detail.blockElements[0])
    });

    menu.open({ x: detail.event.clientX, y: detail.event.clientY });
  }

  private createTagSubmenu(blockElement: HTMLElement): Menu {
    const submenu = new Menu("tag-submenu");
    const predefinedTags = ["重要", "TODO", "完了", "レビュー", "メモ"];

    predefinedTags.forEach(tag => {
      submenu.addItem({
        label: tag,
        click: () => {
          this.addTagToBlock(blockElement, tag);
        }
      });
    });

    submenu.addSeparator();
    
    submenu.addItem({
      label: "カスタムタグ...",
      click: () => {
        this.showCustomTagDialog(blockElement);
      }
    });

    return submenu;
  }

  private async showBlockInfo(blockElement: HTMLElement) {
    const blockId = blockElement.getAttribute('data-node-id');
    if (!blockId) return;

    try {
      const apiClient = new SiYuanAPIClient();
      const blockInfo = await apiClient.getBlock(blockId);
      
      const dialog = new Dialog({
        title: "ブロック情報",
        content: `
          <div class="block-info">
            <div class="info-item">
              <strong>ID:</strong> <code>${blockInfo.id}</code>
            </div>
            <div class="info-item">
              <strong>タイプ:</strong> ${blockInfo.type}
            </div>
            <div class="info-item">
              <strong>作成日時:</strong> ${new Date(parseInt(blockInfo.created)).toLocaleString()}
            </div>
            <div class="info-item">
              <strong>更新日時:</strong> ${new Date(parseInt(blockInfo.updated)).toLocaleString()}
            </div>
            <div class="info-item">
              <strong>文字数:</strong> ${blockInfo.content.length}
            </div>
            <div class="info-item">
              <strong>内容:</strong>
              <pre class="block-content">${blockInfo.content}</pre>
            </div>
          </div>
        `,
        width: "500px"
      });
    } catch (error) {
      showMessage(`ブロック情報の取得に失敗しました: ${error.message}`, 3000, 'error');
    }
  }

  private async addTagToBlock(blockElement: HTMLElement, tag: string) {
    const blockId = blockElement.getAttribute('data-node-id');
    if (!blockId) return;

    try {
      const apiClient = new SiYuanAPIClient();
      await apiClient.request('/attr/setBlockAttrs', {
        id: blockId,
        attrs: {
          'custom-tag': tag
        }
      });
      
      showMessage(`タグ「${tag}」を追加しました`, 2000, 'info');
    } catch (error) {
      showMessage(`タグの追加に失敗しました: ${error.message}`, 3000, 'error');
    }
  }
}
```

## ⌨️ キーボードショートカット

```typescript
class ShortcutManager {
  constructor(private plugin: Plugin) {
    this.registerShortcuts();
  }

  private registerShortcuts() {
    // 基本的なショートカット
    this.plugin.addCommand({
      langKey: "quickInsert",
      langText: "クイック挿入",
      hotkey: "⌘⇧I",
      callback: () => {
        this.showQuickInsertDialog();
      }
    });

    // 検索ショートカット
    this.plugin.addCommand({
      langKey: "advancedSearch",
      langText: "高度な検索",
      hotkey: "⌘⇧F",
      callback: () => {
        this.showAdvancedSearchDialog();
      }
    });

    // ブロック操作ショートカット
    this.plugin.addCommand({
      langKey: "duplicateBlock",
      langText: "ブロックを複製",
      hotkey: "⌘D",
      callback: () => {
        this.duplicateCurrentBlock();
      }
    });

    // カスタムスニペット
    this.plugin.addCommand({
      langKey: "insertTemplate",
      langText: "テンプレート挿入",
      hotkey: "⌘⇧T",
      callback: () => {
        this.showTemplateSelector();
      }
    });
  }

  private showQuickInsertDialog() {
    const quickItems = [
      { label: "現在の日時", value: new Date().toLocaleString() },
      { label: "TODO項目", value: "- [ ] " },
      { label: "重要マーク", value: "⚠️ **重要**: " },
      { label: "コードブロック", value: "```javascript\n\n```" },
      { label: "引用", value: "> " }
    ];

    const dialog = new Dialog({
      title: "クイック挿入",
      content: `
        <div class="quick-insert-items">
          ${quickItems.map((item, index) => `
            <div class="quick-item" data-value="${item.value}" data-index="${index}">
              <div class="quick-item-label">${item.label}</div>
              <div class="quick-item-preview">${item.value}</div>
            </div>
          `).join('')}
        </div>
      `,
      width: "400px"
    });

    // アイテムクリックの処理
    dialog.element.addEventListener('click', (e) => {
      const item = (e.target as Element).closest('.quick-item') as HTMLElement;
      if (item) {
        const value = item.getAttribute('data-value');
        this.insertAtCursor(value);
        dialog.destroy();
      }
    });

    // キーボードナビゲーション
    dialog.element.addEventListener('keydown', (e) => {
      this.handleQuickInsertKeyboard(e, dialog.element, quickItems);
    });
  }

  private insertAtCursor(text: string) {
    // 現在のカーソル位置にテキストを挿入
    const selection = window.getSelection();
    if (selection && selection.rangeCount > 0) {
      const range = selection.getRangeAt(0);
      const textNode = document.createTextNode(text);
      range.insertNode(textNode);
      range.setStartAfter(textNode);
      range.setEndAfter(textNode);
      selection.removeAllRanges();
      selection.addRange(range);
    }
  }

  private async duplicateCurrentBlock() {
    const activeElement = document.activeElement;
    const blockElement = activeElement?.closest('[data-node-id]') as HTMLElement;
    
    if (!blockElement) {
      showMessage('アクティブなブロックが見つかりません', 2000, 'warning');
      return;
    }

    const blockId = blockElement.getAttribute('data-node-id');
    if (!blockId) return;

    try {
      const apiClient = new SiYuanAPIClient();
      const blockInfo = await apiClient.getBlock(blockId);
      
      await apiClient.request('/block/insertBlock', {
        parentID: blockInfo.parent_id,
        previousID: blockId,
        dataType: 'markdown',
        data: blockInfo.content
      });
      
      showMessage('ブロックを複製しました', 2000, 'info');
    } catch (error) {
      showMessage(`ブロックの複製に失敗しました: ${error.message}`, 3000, 'error');
    }
  }
}
```

## 📁 ファイル・アセット管理

プラグインでのファイル操作とアセット管理の実装方法です。

### ファイル管理クラス

```typescript
class FileManager {
  constructor(private apiClient: SiYuanAPIClient) {}

  // ファイルのアップロード（高度な機能付き）
  async uploadFiles(files: File[], options: UploadOptions = {}) {
    const results = [];
    const uiManager = new UIManager(this.plugin);

    await uiManager.withProgress(async (updateProgress) => {
      for (let i = 0; i < files.length; i++) {
        const file = files[i];
        updateProgress((i / files.length) * 100, `アップロード中: ${file.name}`);

        try {
          // ファイルサイズチェック
          if (options.maxSize && file.size > options.maxSize) {
            throw new Error(`ファイルサイズが制限を超えています: ${file.name}`);
          }

          // ファイル形式チェック
          if (options.allowedTypes && !options.allowedTypes.includes(file.type)) {
            throw new Error(`サポートされていないファイル形式: ${file.name}`);
          }

          const result = await this.uploadSingleFile(file, options);
          results.push({ file: file.name, success: true, result });
        } catch (error) {
          results.push({ file: file.name, success: false, error: error.message });
        }
      }

      updateProgress(100, '完了');
    }, 'ファイルアップロード');

    return results;
  }

  private async uploadSingleFile(file: File, options: UploadOptions) {
    const formData = new FormData();
    formData.append('file[]', file);

    const response = await fetch('/api/asset/upload', {
      method: 'POST',
      body: formData
    });

    if (!response.ok) {
      throw new Error(`アップロード失敗: ${response.statusText}`);
    }

    const result = await response.json();
    
    // アップロード後の処理
    if (options.afterUpload) {
      await options.afterUpload(result.data);
    }

    return result.data;
  }

  // ファイルのダウンロード
  async downloadFile(path: string, filename?: string) {
    try {
      const fileData = await this.apiClient.request('/file/getFile', { path });
      
      const blob = new Blob([fileData], { type: 'application/octet-stream' });
      const url = URL.createObjectURL(blob);
      
      const a = document.createElement('a');
      a.href = url;
      a.download = filename || path.split('/').pop() || 'download';
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
      
      showMessage(`ファイル「${filename}」をダウンロードしました`, 2000, 'info');
    } catch (error) {
      showMessage(`ダウンロードに失敗しました: ${error.message}`, 3000, 'error');
    }
  }

  // 画像の最適化
  async optimizeImage(file: File, maxWidth: number = 1920, quality: number = 0.8): Promise<File> {
    return new Promise((resolve, reject) => {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      const img = new Image();

      img.onload = () => {
        // アスペクト比を維持してリサイズ
        const ratio = Math.min(maxWidth / img.width, maxWidth / img.height);
        canvas.width = img.width * ratio;
        canvas.height = img.height * ratio;

        ctx?.drawImage(img, 0, 0, canvas.width, canvas.height);

        canvas.toBlob((blob) => {
          if (blob) {
            const optimizedFile = new File([blob], file.name, {
              type: 'image/jpeg',
              lastModified: Date.now()
            });
            resolve(optimizedFile);
          } else {
            reject(new Error('画像の最適化に失敗しました'));
          }
        }, 'image/jpeg', quality);
      };

      img.onerror = () => reject(new Error('画像の読み込みに失敗しました'));
      img.src = URL.createObjectURL(file);
    });
  }

  // ファイル一覧の取得（フィルター付き）
  async listFiles(directory: string = '/data/assets', filter?: FileFilter) {
    try {
      const files = await this.apiClient.request('/file/readDir', { path: directory });
      
      if (!filter) return files;

      return files.filter((file: any) => {
        if (filter.extensions && !filter.extensions.some(ext => file.name.endsWith(ext))) {
          return false;
        }
        if (filter.minSize && file.size < filter.minSize) {
          return false;
        }
        if (filter.maxSize && file.size > filter.maxSize) {
          return false;
        }
        if (filter.modifiedAfter && new Date(file.modified) < filter.modifiedAfter) {
          return false;
        }
        return true;
      });
    } catch (error) {
      console.error('ファイル一覧の取得に失敗:', error);
      return [];
    }
  }
}

interface UploadOptions {
  maxSize?: number;
  allowedTypes?: string[];
  afterUpload?: (result: any) => Promise<void>;
}

interface FileFilter {
  extensions?: string[];
  minSize?: number;
  maxSize?: number;
  modifiedAfter?: Date;
}
```

## ⚙️ 設定の保存と管理

```typescript
class SettingsManager {
  private readonly STORAGE_KEY = 'plugin-settings';
  private settings: any = {};

  constructor(private plugin: Plugin) {
    this.loadSettings();
  }

  // 設定の読み込み
  async loadSettings() {
    try {
      const data = await this.plugin.loadData(this.STORAGE_KEY);
      this.settings = data ? JSON.parse(data) : this.getDefaultSettings();
    } catch (error) {
      console.error('設定の読み込みに失敗:', error);
      this.settings = this.getDefaultSettings();
    }
  }

  // 設定の保存
  async saveSettings() {
    try {
      await this.plugin.saveData(this.STORAGE_KEY, JSON.stringify(this.settings));
      showMessage('設定を保存しました', 1000, 'info');
    } catch (error) {
      console.error('設定の保存に失敗:', error);
      showMessage('設定の保存に失敗しました', 3000, 'error');
    }
  }

  // 設定値の取得
  get<T>(key: string, defaultValue?: T): T {
    return this.getNestedValue(this.settings, key) ?? defaultValue;
  }

  // 設定値の更新
  async set(key: string, value: any) {
    this.setNestedValue(this.settings, key, value);
    await this.saveSettings();
  }

  // 設定のリセット
  async reset() {
    this.settings = this.getDefaultSettings();
    await this.saveSettings();
  }

  // デフォルト設定
  private getDefaultSettings() {
    return {
      general: {
        autoSave: true,
        notifications: true,
        language: 'ja_JP'
      },
      ui: {
        theme: 'auto',
        fontSize: 14,
        showLineNumbers: true
      },
      advanced: {
        apiTimeout: 5000,
        maxRetries: 3,
        cacheSize: 100
      }
    };
  }

  // ネストされた値の取得
  private getNestedValue(obj: any, path: string) {
    return path.split('.').reduce((current, key) => current?.[key], obj);
  }

  // ネストされた値の設定
  private setNestedValue(obj: any, path: string, value: any) {
    const keys = path.split('.');
    const lastKey = keys.pop()!;
    const target = keys.reduce((current, key) => {
      if (!(key in current)) current[key] = {};
      return current[key];
    }, obj);
    target[lastKey] = value;
  }

  // 設定画面の表示
  showSettingsDialog() {
    const settingTab = new Setting({
      confirmCallback: () => {
        this.saveSettings();
      }
    });

    // 一般設定
    settingTab.addItem({
      title: '自動保存',
      description: '変更を自動的に保存する',
      type: 'checkbox',
      key: 'general.autoSave',
      value: this.get('general.autoSave'),
      action: {
        callback: (value: boolean) => {
          this.set('general.autoSave', value);
        }
      }
    });

    // UI設定
    settingTab.addItem({
      title: 'テーマ',
      description: 'プラグインのテーマを選択',
      type: 'select',
      key: 'ui.theme',
      value: this.get('ui.theme'),
      options: {
        auto: '自動',
        light: 'ライト',
        dark: 'ダーク'
      },
      action: {
        callback: (value: string) => {
          this.set('ui.theme', value);
          this.applyTheme(value);
        }
      }
    });

    settingTab.open();
  }

  private applyTheme(theme: string) {
    document.documentElement.setAttribute('data-plugin-theme', theme);
  }
}
```

## 🌐 多言語対応（i18n）

```typescript
class I18nManager {
  private translations: Record<string, Record<string, string>> = {};
  private currentLanguage: string = 'ja_JP';

  constructor(private plugin: Plugin) {
    this.detectLanguage();
    this.loadTranslations();
  }

  private detectLanguage() {
    // SiYuanの言語設定を取得
    this.currentLanguage = window.siyuan?.config?.lang || 'ja_JP';
  }

  private async loadTranslations() {
    const languages = ['en_US', 'ja_JP', 'zh_CN'];
    
    for (const lang of languages) {
      try {
        const response = await fetch(`/plugins/${this.plugin.name}/i18n/${lang}.json`);
        if (response.ok) {
          this.translations[lang] = await response.json();
        }
      } catch (error) {
        console.warn(`翻訳ファイル ${lang}.json の読み込みに失敗:`, error);
      }
    }
  }

  // 翻訳文字列の取得
  t(key: string, params?: Record<string, string>): string {
    const translation = this.translations[this.currentLanguage]?.[key] 
                     || this.translations['en_US']?.[key] 
                     || key;

    if (params) {
      return this.interpolate(translation, params);
    }

    return translation;
  }

  // パラメータの補間
  private interpolate(text: string, params: Record<string, string>): string {
    return text.replace(/\{\{(\w+)\}\}/g, (match, key) => {
      return params[key] || match;
    });
  }

  // 複数形の処理
  tn(key: string, count: number, params?: Record<string, string>): string {
    const pluralKey = count === 1 ? key : `${key}_plural`;
    const translation = this.t(pluralKey, { ...params, count: count.toString() });
    return translation;
  }

  // 日付・時刻のローカライズ
  formatDate(date: Date, format: 'short' | 'long' = 'short'): string {
    const options: Intl.DateTimeFormatOptions = 
      format === 'long' 
        ? { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit' }
        : { year: 'numeric', month: 'short', day: 'numeric' };

    return new Intl.DateTimeFormat(this.currentLanguage.replace('_', '-'), options).format(date);
  }

  // 数値のローカライズ
  formatNumber(number: number): string {
    return new Intl.NumberFormat(this.currentLanguage.replace('_', '-')).format(number);
  }
}

// 翻訳ファイルの例 (i18n/ja_JP.json)
const jaTranslations = {
  "plugin_name": "素晴らしいプラグイン",
  "welcome_message": "{{name}}さん、こんにちは！",
  "item_count": "{{count}}個のアイテム",
  "item_count_plural": "{{count}}個のアイテム",
  "error_network": "ネットワークエラーが発生しました",
  "success_save": "正常に保存されました",
  "confirm_delete": "このアイテムを削除してもよろしいですか？",
  "button_ok": "OK",
  "button_cancel": "キャンセル"
};
```

## 🔗 プラグイン間連携

```typescript
class PluginCommunication {
  private eventBus: EventTarget;
  private messageHandlers: Map<string, Function[]> = new Map();

  constructor(private plugin: Plugin) {
    this.eventBus = new EventTarget();
    this.initializeCommunication();
  }

  private initializeCommunication() {
    // グローバルイベントバスへの登録
    if (!window.pluginEventBus) {
      window.pluginEventBus = new EventTarget();
    }

    // プラグイン間メッセージの受信
    window.pluginEventBus.addEventListener('plugin-message', (event: any) => {
      this.handleIncomingMessage(event.detail);
    });
  }

  // 他のプラグインにメッセージを送信
  sendMessage(targetPlugin: string, type: string, data: any) {
    const message = {
      from: this.plugin.name,
      to: targetPlugin,
      type,
      data,
      timestamp: Date.now()
    };

    const event = new CustomEvent('plugin-message', { detail: message });
    window.pluginEventBus.dispatchEvent(event);
  }

  // ブロードキャストメッセージの送信
  broadcast(type: string, data: any) {
    this.sendMessage('*', type, data);
  }

  // メッセージハンドラーの登録
  onMessage(type: string, handler: (data: any, from: string) => void) {
    if (!this.messageHandlers.has(type)) {
      this.messageHandlers.set(type, []);
    }
    this.messageHandlers.get(type)!.push(handler);
  }

  private handleIncomingMessage(message: any) {
    // 自分宛てまたはブロードキャストメッセージのみ処理
    if (message.to !== this.plugin.name && message.to !== '*') {
      return;
    }

    // 自分自身からのメッセージは無視
    if (message.from === this.plugin.name) {
      return;
    }

    const handlers = this.messageHandlers.get(message.type) || [];
    handlers.forEach(handler => {
      try {
        handler(message.data, message.from);
      } catch (error) {
        console.error(`メッセージハンドラーエラー:`, error);
      }
    });
  }

  // データ共有機能
  async shareData(key: string, data: any, options: ShareOptions = {}) {
    const sharedData = {
      key,
      data,
      from: this.plugin.name,
      timestamp: Date.now(),
      ttl: options.ttl || 0,
      permissions: options.permissions || ['read']
    };

    // 共有ストレージに保存
    await this.plugin.saveData(`shared:${key}`, JSON.stringify(sharedData));

    // 他のプラグインに通知
    this.broadcast('data-shared', { key, permissions: sharedData.permissions });
  }

  async getSharedData(key: string, fromPlugin?: string): Promise<any> {
    try {
      const dataStr = await this.plugin.loadData(`shared:${key}`);
      if (!dataStr) return null;

      const sharedData = JSON.parse(dataStr);

      // TTLチェック
      if (sharedData.ttl > 0 && Date.now() > sharedData.timestamp + sharedData.ttl) {
        return null;
      }

      // 権限チェック
      if (!sharedData.permissions.includes('read')) {
        throw new Error('データの読み取り権限がありません');
      }

      // 特定のプラグインからのデータのみ取得
      if (fromPlugin && sharedData.from !== fromPlugin) {
        return null;
      }

      return sharedData.data;
    } catch (error) {
      console.error('共有データの取得に失敗:', error);
      return null;
    }
  }
}

interface ShareOptions {
  ttl?: number; // 生存時間（ミリ秒）
  permissions?: ('read' | 'write')[];
}

// プラグイン連携の使用例
class CollaborativePlugin extends Plugin {
  private communication: PluginCommunication;

  async onload() {
    this.communication = new PluginCommunication(this);

    // メッセージハンドラーの登録
    this.communication.onMessage('request-data', (data, from) => {
      this.handleDataRequest(data, from);
    });

    this.communication.onMessage('data-updated', (data, from) => {
      this.handleDataUpdate(data, from);
    });

    // 定期的なデータ同期
    setInterval(() => {
      this.syncWithOtherPlugins();
    }, 30000); // 30秒間隔
  }

  private async handleDataRequest(request: any, from: string) {
    // データリクエストの処理
    const requestedData = await this.getRequestedData(request.type);
    this.communication.sendMessage(from, 'data-response', {
      requestId: request.id,
      data: requestedData
    });
  }

  private async syncWithOtherPlugins() {
    // 他のプラグインとのデータ同期
    this.communication.broadcast('sync-request', {
      timestamp: Date.now(),
      version: this.version
    });
  }
}
```

## 🎯 イベントシステムの活用

SiYuanのイベントシステムを効果的に活用する方法です。

### イベントマネージャー

```typescript
class EventManager {
  private eventListeners: Map<string, Function[]> = new Map();

  constructor(private plugin: Plugin) {
    this.setupEventListeners();
  }

  private setupEventListeners() {
    // ブロック作成イベント
    this.plugin.eventBus.on('ws-main', (event: any) => {
      if (event.detail.cmd === 'create') {
        this.handleBlockCreated(event.detail);
      } else if (event.detail.cmd === 'update') {
        this.handleBlockUpdated(event.detail);
      } else if (event.detail.cmd === 'remove') {
        this.handleBlockRemoved(event.detail);
      }
    });

    // ドキュメント切り替えイベント  
    this.plugin.eventBus.on('switch-protyle', (event: any) => {
      this.handleDocumentSwitch(event.detail);
    });

    // 保存イベント
    this.plugin.eventBus.on('loaded-protyle', (event: any) => {
      this.handleDocumentLoaded(event.detail);
    });
  }

  // イベントハンドラーの登録
  on(eventType: string, handler: Function) {
    if (!this.eventListeners.has(eventType)) {
      this.eventListeners.set(eventType, []);
    }
    this.eventListeners.get(eventType)!.push(handler);
  }

  // イベントの発火
  emit(eventType: string, data: any) {
    const handlers = this.eventListeners.get(eventType) || [];
    handlers.forEach(handler => {
      try {
        handler(data);
      } catch (error) {
        console.error(`イベントハンドラーエラー [${eventType}]:`, error);
      }
    });
  }

  private handleBlockCreated(detail: any) {
    console.log('ブロックが作成されました:', detail);
    this.emit('block-created', detail);
    
    // 自動タグ付け機能
    this.autoTagNewBlock(detail);
  }

  private handleBlockUpdated(detail: any) {
    console.log('ブロックが更新されました:', detail);
    this.emit('block-updated', detail);
    
    // 自動保存トリガー
    this.triggerAutoSave(detail);
  }

  private handleDocumentSwitch(detail: any) {
    console.log('ドキュメントが切り替わりました:', detail);
    this.emit('document-switched', detail);
    
    // ドキュメント固有の設定を適用
    this.applyDocumentSpecificSettings(detail);
  }

  private async autoTagNewBlock(detail: any) {
    if (detail.data && detail.data.length > 0) {
      const blockId = detail.data[0].id;
      const content = detail.data[0].content;
      
      // キーワードベースの自動タグ付け
      const autoTags = this.extractAutoTags(content);
      if (autoTags.length > 0) {
        const apiClient = new SiYuanAPIClient();
        await apiClient.request('/attr/setBlockAttrs', {
          id: blockId,
          attrs: {
            'auto-tags': autoTags.join(', ')
          }
        });
      }
    }
  }

  private extractAutoTags(content: string): string[] {
    const tagRules = [
      { pattern: /(TODO|タスク|課題)/i, tag: 'task' },
      { pattern: /(重要|緊急|優先)/i, tag: 'important' },
      { pattern: /(メモ|ノート|記録)/i, tag: 'note' },
      { pattern: /(会議|ミーティング)/i, tag: 'meeting' },
      { pattern: /(アイデア|提案)/i, tag: 'idea' }
    ];

    return tagRules
      .filter(rule => rule.pattern.test(content))
      .map(rule => rule.tag);
  }
}
```

## 🌍 外部サービス連携

外部API やサービスとの連携方法を示します。

### 外部API統合クラス

```typescript
class ExternalAPIIntegration {
  private apiCache: Map<string, { data: any; expires: number }> = new Map();

  constructor(private settings: SettingsManager) {}

  // REST API呼び出し（汎用）
  async callExternalAPI(url: string, options: RequestInit = {}) {
    const cacheKey = `${url}:${JSON.stringify(options)}`;
    const cached = this.apiCache.get(cacheKey);
    
    if (cached && Date.now() < cached.expires) {
      return cached.data;
    }

    try {
      const response = await fetch(url, {
        ...options,
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        }
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      const data = await response.json();
      
      // 5分間キャッシュ
      this.apiCache.set(cacheKey, {
        data,
        expires: Date.now() + 5 * 60 * 1000
      });

      return data;
    } catch (error) {
      console.error('外部API呼び出しエラー:', error);
      throw error;
    }
  }

  // OpenAI API統合
  async callOpenAI(prompt: string, options: OpenAIOptions = {}) {
    const apiKey = this.settings.get('openai.apiKey');
    if (!apiKey) {
      throw new Error('OpenAI APIキーが設定されていません');
    }

    return this.callExternalAPI('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${apiKey}`
      },
      body: JSON.stringify({
        model: options.model || 'gpt-3.5-turbo',
        messages: [
          { role: 'user', content: prompt }
        ],
        max_tokens: options.maxTokens || 1000,
        temperature: options.temperature || 0.7
      })
    });
  }

  // GitHub API統合
  async callGitHubAPI(endpoint: string, options: GitHubOptions = {}) {
    const token = this.settings.get('github.token');
    const baseUrl = 'https://api.github.com';

    return this.callExternalAPI(`${baseUrl}${endpoint}`, {
      method: options.method || 'GET',
      headers: {
        'Authorization': token ? `token ${token}` : undefined,
        'Accept': 'application/vnd.github.v3+json'
      },
      body: options.body ? JSON.stringify(options.body) : undefined
    });
  }

  // Webhook の設定と処理
  setupWebhook(endpoint: string, handler: (data: any) => void) {
    // Express.js などのサーバーフレームワークを使用する場合
    // 実際の実装はプラグインの要件に応じて調整
    console.log(`Webhook setup for ${endpoint}`);
    
    // イベントリスナーとして登録
    window.addEventListener('webhook-received', (event: any) => {
      if (event.detail.endpoint === endpoint) {
        handler(event.detail.data);
      }
    });
  }
}

interface OpenAIOptions {
  model?: string;
  maxTokens?: number;
  temperature?: number;
}

interface GitHubOptions {
  method?: string;
  body?: any;
}
```

## 📊 パフォーマンス最適化

### パフォーマンス監視とプロファイリング

```typescript
class PerformanceMonitor {
  private metrics: Map<string, PerformanceMetric[]> = new Map();

  // 処理時間の測定
  async measureAsync<T>(name: string, operation: () => Promise<T>): Promise<T> {
    const startTime = performance.now();
    const startMemory = this.getMemoryUsage();

    try {
      const result = await operation();
      const endTime = performance.now();
      const endMemory = this.getMemoryUsage();

      this.recordMetric(name, {
        duration: endTime - startTime,
        memoryDelta: endMemory - startMemory,
        timestamp: Date.now(),
        success: true
      });

      return result;
    } catch (error) {
      const endTime = performance.now();
      this.recordMetric(name, {
        duration: endTime - startTime,
        memoryDelta: 0,
        timestamp: Date.now(),
        success: false,
        error: error.message
      });
      throw error;
    }
  }

  // メトリクスの記録
  private recordMetric(name: string, metric: PerformanceMetric) {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }
    
    const metrics = this.metrics.get(name)!;
    metrics.push(metric);
    
    // 最新の100件のみ保持
    if (metrics.length > 100) {
      metrics.splice(0, metrics.length - 100);
    }
  }

  // パフォーマンス統計の取得
  getStatistics(name: string): PerformanceStats | null {
    const metrics = this.metrics.get(name);
    if (!metrics || metrics.length === 0) return null;

    const successfulMetrics = metrics.filter(m => m.success);
    const durations = successfulMetrics.map(m => m.duration);
    
    return {
      name,
      count: metrics.length,
      successRate: successfulMetrics.length / metrics.length,
      avgDuration: durations.reduce((a, b) => a + b, 0) / durations.length,
      minDuration: Math.min(...durations),
      maxDuration: Math.max(...durations),
      p95Duration: this.percentile(durations, 95),
      totalMemoryDelta: successfulMetrics.reduce((sum, m) => sum + m.memoryDelta, 0)
    };
  }

  // すべての統計情報を取得
  getAllStatistics(): PerformanceStats[] {
    return Array.from(this.metrics.keys())
      .map(name => this.getStatistics(name))
      .filter(stats => stats !== null) as PerformanceStats[];
  }

  private getMemoryUsage(): number {
    return (performance as any).memory?.usedJSHeapSize || 0;
  }

  private percentile(values: number[], p: number): number {
    const sorted = values.slice().sort((a, b) => a - b);
    const index = Math.ceil((p / 100) * sorted.length) - 1;
    return sorted[index] || 0;
  }
}

interface PerformanceMetric {
  duration: number;
  memoryDelta: number;
  timestamp: number;
  success: boolean;
  error?: string;
}

interface PerformanceStats {
  name: string;
  count: number;
  successRate: number;
  avgDuration: number;
  minDuration: number;
  maxDuration: number;
  p95Duration: number;
  totalMemoryDelta: number;
}
```

## 🧪 デバッグとテスト

### デバッグユーティリティ

```typescript
class DebugUtils {
  private static instance: DebugUtils;
  private debugEnabled: boolean = false;
  private logs: DebugLog[] = [];

  static getInstance(): DebugUtils {
    if (!DebugUtils.instance) {
      DebugUtils.instance = new DebugUtils();
    }
    return DebugUtils.instance;
  }

  enableDebug(enabled: boolean = true) {
    this.debugEnabled = enabled;
    if (enabled) {
      console.log('🐛 Debug mode enabled');
    }
  }

  log(level: 'info' | 'warn' | 'error', message: string, data?: any) {
    const logEntry: DebugLog = {
      level,
      message,
      data,
      timestamp: new Date(),
      stack: new Error().stack
    };

    this.logs.push(logEntry);
    
    // 最新の1000件のみ保持
    if (this.logs.length > 1000) {
      this.logs.splice(0, this.logs.length - 1000);
    }

    if (this.debugEnabled) {
      const prefix = level === 'error' ? '❌' : level === 'warn' ? '⚠️' : 'ℹ️';
      console.log(`${prefix} [${this.formatTime(logEntry.timestamp)}] ${message}`, data || '');
    }
  }

  // API呼び出しのデバッグ
  async debugAPICall<T>(endpoint: string, data: any, apiCall: () => Promise<T>): Promise<T> {
    this.log('info', `API Call: ${endpoint}`, data);
    
    try {
      const result = await apiCall();
      this.log('info', `API Success: ${endpoint}`, result);
      return result;
    } catch (error) {
      this.log('error', `API Error: ${endpoint}`, { error: error.message, data });
      throw error;
    }
  }

  // ログのエクスポート
  exportLogs(): string {
    return JSON.stringify(this.logs, null, 2);
  }

  // ログの表示
  showDebugPanel() {
    const dialog = new Dialog({
      title: "デバッグログ",
      content: this.generateDebugPanelHTML(),
      width: "800px",
      height: "600px"
    });

    this.attachDebugPanelEvents(dialog);
  }

  private generateDebugPanelHTML(): string {
    const recentLogs = this.logs.slice(-100).reverse();
    
    return `
      <div class="debug-panel">
        <div class="debug-controls">
          <button class="b3-button" id="clear-logs">ログクリア</button>
          <button class="b3-button" id="export-logs">ログエクスポート</button>
          <label>
            <input type="checkbox" id="auto-scroll" checked> 自動スクロール
          </label>
        </div>
        <div class="debug-logs" id="debug-logs">
          ${recentLogs.map(log => `
            <div class="debug-log-entry ${log.level}">
              <div class="log-header">
                <span class="log-level">${log.level.toUpperCase()}</span>
                <span class="log-time">${this.formatTime(log.timestamp)}</span>
              </div>
              <div class="log-message">${log.message}</div>
              ${log.data ? `<div class="log-data"><pre>${JSON.stringify(log.data, null, 2)}</pre></div>` : ''}
            </div>
          `).join('')}
        </div>
      </div>
    `;
  }

  private attachDebugPanelEvents(dialog: Dialog) {
    const element = dialog.element;
    
    element.querySelector('#clear-logs')?.addEventListener('click', () => {
      this.logs = [];
      showMessage('ログをクリアしました', 1000, 'info');
    });

    element.querySelector('#export-logs')?.addEventListener('click', () => {
      const blob = new Blob([this.exportLogs()], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `debug-logs-${Date.now()}.json`;
      a.click();
      URL.revokeObjectURL(url);
    });
  }

  private formatTime(date: Date): string {
    return date.toLocaleTimeString('ja-JP', { 
      hour12: false, 
      hour: '2-digit', 
      minute: '2-digit', 
      second: '2-digit', 
      fractionalSecondDigits: 3 
    });
  }
}

interface DebugLog {
  level: 'info' | 'warn' | 'error';
  message: string;
  data?: any;
  timestamp: Date;
  stack?: string;
}
```

## 📦 配布と更新

### 自動更新システム

```typescript
class UpdateManager {
  private readonly UPDATE_CHECK_INTERVAL = 24 * 60 * 60 * 1000; // 24時間
  private updateCheckTimer?: number;

  constructor(
    private plugin: Plugin,
    private settings: SettingsManager
  ) {}

  // 自動更新チェックの開始
  startUpdateCheck() {
    if (this.settings.get('updates.autoCheck', true)) {
      this.checkForUpdates();
      this.updateCheckTimer = setInterval(() => {
        this.checkForUpdates();
      }, this.UPDATE_CHECK_INTERVAL);
    }
  }

  // 更新チェックの停止
  stopUpdateCheck() {
    if (this.updateCheckTimer) {
      clearInterval(this.updateCheckTimer);
      this.updateCheckTimer = undefined;
    }
  }

  // 更新の確認
  async checkForUpdates(): Promise<UpdateInfo | null> {
    try {
      const currentVersion = this.plugin.i18n.version || '1.0.0';
      const updateInfo = await this.fetchUpdateInfo();
      
      if (this.isNewerVersion(updateInfo.version, currentVersion)) {
        this.showUpdateNotification(updateInfo);
        return updateInfo;
      }
      
      return null;
    } catch (error) {
      console.error('更新チェックエラー:', error);
      return null;
    }
  }

  // 更新情報の取得
  private async fetchUpdateInfo(): Promise<UpdateInfo> {
    const repoUrl = this.plugin.i18n.url || '';
    const apiUrl = repoUrl.replace('github.com', 'api.github.com/repos') + '/releases/latest';
    
    const response = await fetch(apiUrl);
    if (!response.ok) {
      throw new Error(`更新情報の取得に失敗: ${response.statusText}`);
    }
    
    const release = await response.json();
    
    return {
      version: release.tag_name,
      releaseNotes: release.body,
      downloadUrl: release.zipball_url,
      publishedAt: new Date(release.published_at)
    };
  }

  // バージョン比較
  private isNewerVersion(newVersion: string, currentVersion: string): boolean {
    const normalize = (v: string) => v.replace(/^v/, '').split('.').map(n => parseInt(n, 10));
    const newParts = normalize(newVersion);
    const currentParts = normalize(currentVersion);
    
    for (let i = 0; i < Math.max(newParts.length, currentParts.length); i++) {
      const newPart = newParts[i] || 0;
      const currentPart = currentParts[i] || 0;
      
      if (newPart > currentPart) return true;
      if (newPart < currentPart) return false;
    }
    
    return false;
  }

  // 更新通知の表示
  private showUpdateNotification(updateInfo: UpdateInfo) {
    const dialog = new Dialog({
      title: "プラグイン更新のお知らせ",
      content: `
        <div class="update-notification">
          <div class="update-header">
            <h3>新しいバージョンが利用可能です</h3>
            <div class="version-info">
              <span class="current-version">現在: v${this.plugin.i18n.version}</span>
              <span class="arrow">→</span>
              <span class="new-version">新版: ${updateInfo.version}</span>
            </div>
          </div>
          
          <div class="release-notes">
            <h4>更新内容:</h4>
            <div class="release-content">${this.formatReleaseNotes(updateInfo.releaseNotes)}</div>
          </div>
          
          <div class="update-actions">
            <label class="auto-update-option">
              <input type="checkbox" id="auto-update"> 今後は自動的に更新する
            </label>
          </div>
        </div>
      `,
      width: "500px",
      destroyCallback: () => {
        const autoUpdate = dialog.element.querySelector('#auto-update') as HTMLInputElement;
        if (autoUpdate?.checked) {
          this.settings.set('updates.autoInstall', true);
        }
      }
    });

    // 更新ボタンの追加
    const actionArea = dialog.element.querySelector('.b3-dialog__action');
    if (actionArea) {
      const updateButton = document.createElement('button');
      updateButton.className = 'b3-button b3-button--text';
      updateButton.textContent = '今すぐ更新';
      updateButton.addEventListener('click', () => {
        this.performUpdate(updateInfo);
        dialog.destroy();
      });
      
      actionArea.insertBefore(updateButton, actionArea.firstChild);
    }
  }

  // 更新の実行
  private async performUpdate(updateInfo: UpdateInfo) {
    try {
      showMessage('プラグインを更新しています...', 3000, 'info');
      
      // 更新パッケージのダウンロードと展開は
      // SiYuanのプラグインマネージャーに委譲
      await fetchPost('/bazaar/installBazaarPackage', {
        repoURL: this.plugin.i18n.url,
        repoHash: updateInfo.version.replace('v', ''),
        update: true
      });
      
      showMessage('プラグインの更新が完了しました。再起動してください。', 5000, 'success');
    } catch (error) {
      console.error('更新エラー:', error);
      showMessage(`更新に失敗しました: ${error.message}`, 5000, 'error');
    }
  }

  private formatReleaseNotes(notes: string): string {
    // Markdownの簡易変換
    return notes
      .replace(/^##\s+(.+)$/gm, '<h4>$1</h4>')
      .replace(/^-\s+(.+)$/gm, '<li>$1</li>')
      .replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>');
  }
}

interface UpdateInfo {
  version: string;
  releaseNotes: string;
  downloadUrl: string;
  publishedAt: Date;

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

## 🎯 プラグイン開発のチェックリスト

### 開発開始前の準備

- [ ] 開発環境の構築
  - [ ] Node.js (v16以上) のインストール
  - [ ] TypeScript の設定
  - [ ] VSCode + SiYuan 拡張機能
- [ ] プロジェクト構成の設計
  - [ ] ディレクトリ構造の決定
  - [ ] パッケージ依存関係の定義
  - [ ] ビルド設定の構築

### コア機能開発

- [ ] 基本プラグインクラスの実装
- [ ] API クライアントの設定
- [ ] 設定管理システムの実装
- [ ] エラーハンドリングの実装
- [ ] ログシステムの実装

### UI/UX 開発

- [ ] ユーザーインターフェースの設計
- [ ] ダイアログ・パネルの実装
- [ ] キーボードショートカットの設定
- [ ] コンテキストメニューの拡張
- [ ] 多言語対応の実装

### 品質保証

- [ ] ユニットテストの作成
- [ ] 統合テストの実装
- [ ] パフォーマンステストの実行
- [ ] メモリリーク検証
- [ ] エラーケースのテスト

### 配布準備

- [ ] ドキュメントの作成
- [ ] プラグイン設定ファイルの最終調整
- [ ] バージョン管理の実装
- [ ] 更新システムの実装
- [ ] 配布パッケージのビルド

## 🔧 よくある問題と解決方法

### 1. API 呼び出しが失敗する

**問題**: `TypeError: Cannot read property 'siyuan' of undefined`

**解決方法**:
```typescript
// ❌ 問題のあるコード
const token = window.siyuan.config.api.token;

// ✅ 修正されたコード
const token = window.siyuan?.config?.api?.token || '';
if (!token) {
  console.error('API token not available');
  return;
}
```

### 2. プラグインが正常に読み込まれない

**問題**: プラグインが SiYuan で認識されない

**解決方法**:
1. `plugin.json` の形式を確認
2. 必須フィールドが全て記入されているか確認
3. ファイルパスとディレクトリ構造を確認

```json
{
  "name": "my-plugin",
  "author": "Your Name", 
  "url": "https://github.com/username/my-plugin",
  "version": "1.0.0",
  "minAppVersion": "2.10.0"
}
```

### 3. イベントリスナーが動作しない

**問題**: ブロックイベントが捕捉されない

**解決方法**:
```typescript
// ❌ 問題のあるコード
this.eventBus.on("click-blockicon", this.handleBlockClick);

// ✅ 修正されたコード  
this.eventBus.on("click-blockicon", ({ detail }) => {
  this.handleBlockClick(detail);
});
```

### 4. メモリリークの発生

**問題**: プラグインが大量のメモリを消費する

**解決方法**:
```typescript
class MemoryEfficientPlugin extends Plugin {
  private intervals: number[] = [];
  private eventListeners: (() => void)[] = [];

  async onload() {
    // インターバルIDを保存
    const intervalId = setInterval(() => {
      this.doPeriodicTask();
    }, 5000);
    this.intervals.push(intervalId);

    // イベントリスナーのクリーンアップ関数を保存
    const cleanup = this.eventBus.on('some-event', this.handler);
    this.eventListeners.push(cleanup);
  }

  async onunload() {
    // 全てのインターバルをクリア
    this.intervals.forEach(id => clearInterval(id));
    this.intervals = [];

    // 全てのイベントリスナーを削除
    this.eventListeners.forEach(cleanup => cleanup());
    this.eventListeners = [];
  }
}
```

### 5. パフォーマンスの問題

**問題**: プラグインが SiYuan を重くする

**解決方法**:
```typescript
// ❌ 非効率なコード
for (const blockId of blockIds) {
  await this.processBlock(blockId);
}

// ✅ 効率的なコード
const batchSize = 10;
for (let i = 0; i < blockIds.length; i += batchSize) {
  const batch = blockIds.slice(i, i + batchSize);
  await Promise.all(batch.map(id => this.processBlock(id)));
}
```

## 📚 推奨学習リソース

### 公式ドキュメント

1. **[SiYuan API リファレンス](API_ja_JP.md)** - 完全な API 仕様
2. **[プラグイン開発ガイド](https://github.com/siyuan-note/petal)** - 公式開発ガイド
3. **[サンプルプラグイン](https://github.com/siyuan-note/plugin-sample)** - 基本テンプレート

### コミュニティリソース

1. **[SiYuan フォーラム](https://liuyun.io/)** - コミュニティディスカッション
2. **[Discord サーバー](https://discord.gg/bzfCBwMzdP)** - リアルタイムサポート
3. **[GitHub Discussions](https://github.com/siyuan-note/siyuan/discussions)** - 技術的な議論

### 技術スタック学習

1. **TypeScript**:
   - [TypeScript Handbook](https://www.typescriptlang.org/docs/)
   - [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

2. **Web APIs**:
   - [MDN Web Docs](https://developer.mozilla.org/en-US/)
   - [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)

3. **Node.js エコシステム**:
   - [Node.js Documentation](https://nodejs.org/en/docs/)
   - [npm Documentation](https://docs.npmjs.com/)

## 🎉 おめでとうございます！

このガイドを通じて、あなたは SiYuan プラグイン開発の包括的な知識を獲得しました。

### 次のステップ

1. **実際にプラグインを作成**: 小さなプラグインから始めて、徐々に機能を拡張していきましょう。

2. **コミュニティに参加**: 他の開発者との交流を通じて、新しいアイデアやベストプラクティスを学びましょう。

3. **オープンソースに貢献**: 自分のプラグインを公開して、SiYuan エコシステムに貢献しましょう。

4. **継続的な学習**: SiYuan は継続的に発展しています。新しい機能や API の更新に注目しましょう。

### 成功の鍵

- **ユーザー中心設計**: 常にエンドユーザーの体験を考慮して開発する
- **品質への拘り**: テストとドキュメント化を怠らない
- **コミュニティとの連携**: フィードバックを受け入れ、改善し続ける
- **持続可能な開発**: 保守しやすいコードを書く

---

## 参考リンク

### 開発リソース

- **[プラグイン開発ベストプラクティス](PLUGIN_BEST_PRACTICES_ja_JP.md)** - 高品質なプラグイン開発のガイドライン
- **[プラグインショーケース](PLUGIN_SHOWCASE_ja_JP.md)** - 優秀なプラグイン事例
- **[完全なAPIリファレンス](API_ja_JP.md)** - 全エンドポイントの詳細

### API ガイド

- **[初心者向けAPIガイド](API_BEGINNERS_ja_JP.md)** - API使用の基本
- **[高度なAPIパターン](API_ADVANCED_PATTERNS_ja_JP.md)** - 複雑な操作パターン
- **[カテゴリ別APIリファレンス](API_REFERENCE_BY_CATEGORY_ja_JP.md)** - 機能別API一覧
- **[未記載エンドポイント補完](API_MISSING_ENDPOINTS_ja_JP.md)** - 追加APIエンドポイント

### 外部リソース

- **[SiYuan 公式サイト](https://b3log.org/siyuan/)** - アプリケーションの詳細
- **[GitHub リポジトリ](https://github.com/siyuan-note/siyuan)** - ソースコードと Issues
- **[Bazaar マーケットプレイス](https://github.com/siyuan-note/bazaar)** - プラグイン配布

---

## 更新履歴

- **2024年1月**: 包括的なプラグイン開発ガイドとして大幅に拡張
  - 実践的なコード例を大幅に追加
  - UI/UX 開発セクションを新設
  - パフォーマンス最適化ガイドを強化
  - デバッグ・テスト手法を詳述
  - 外部サービス連携パターンを追加

*このガイドは SiYuan API の発展と共に継続的に更新されます。新しい機能やベストプラクティスが確立され次第、追加・改善していきます。*

---

**Happy Coding! 🚀**

皆さんの創造性豊かなプラグインが SiYuan エコシステムを豊かにすることを楽しみにしています。
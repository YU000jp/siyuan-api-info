# SiYuan 高度なAPIパターン

SiYuan APIを活用した高度なプログラミングパターンとベストプラクティスを紹介します。

**対象読者**: SiYuan APIの基本操作をマスターした開発者

**前提知識**:
- SiYuan APIの基本操作
- JavaScript/TypeScriptの中級以上の知識
- 非同期プログラミング、データベース設計の理解

**関連ドキュメント**:
- [初心者向けAPIガイド](API_BEGINNERS_ja_JP.md) - API使用の基本
- [プラグイン開発者向けAPIガイド](API_PLUGIN_DEVELOPERS_ja_JP.md) - プラグイン開発特化
- [完全なAPIリファレンス](API_ja_JP.md) - 全エンドポイント

---

## 目次

- [アーキテクチャパターン](#アーキテクチャパターン)
- [パフォーマンス最適化](#パフォーマンス最適化)
- [エラーハンドリング戦略](#エラーハンドリング戦略)
- [データ同期とキャッシュ](#データ同期とキャッシュ)
- [バッチ処理とトランザクション](#バッチ処理とトランザクション)
- [リアルタイム更新](#リアルタイム更新)
- [セキュリティベストプラクティス](#セキュリティベストプラクティス)
- [テストとモニタリング](#テストとモニタリング)
- [スケーラビリティ考慮事項](#スケーラビリティ考慮事項)
- [実践的な応用例](#実践的な応用例)

---

## アーキテクチャパターン

### 1. Repository パターン

データアクセスを抽象化して、APIの詳細をビジネスロジックから分離します。

```typescript
interface BlockRepository {
  findById(id: string): Promise<Block | null>;
  findByType(type: string): Promise<Block[]>;
  save(block: Block): Promise<Block>;
  delete(id: string): Promise<void>;
}

class SiYuanBlockRepository implements BlockRepository {
  constructor(private apiClient: SiYuanAPIClient) {}

  async findById(id: string): Promise<Block | null> {
    try {
      const result = await this.apiClient.call('/block/getBlockInfo', { id });
      return this.mapToBlock(result.data);
    } catch (error) {
      if (error.code === 404) return null;
      throw error;
    }
  }

  async findByType(type: string): Promise<Block[]> {
    const result = await this.apiClient.call('/query/sql', {
      stmt: `SELECT * FROM blocks WHERE type = ? ORDER BY created DESC`,
      args: [type]
    });
    return result.data.map(this.mapToBlock);
  }

  async save(block: Block): Promise<Block> {
    if (block.id) {
      return await this.update(block);
    } else {
      return await this.create(block);
    }
  }

  private async create(block: Block): Promise<Block> {
    const result = await this.apiClient.call('/block/insertBlock', {
      dataType: 'markdown',
      data: block.content,
      parentID: block.parentId
    });
    return { ...block, id: result.data[0].doOperations[0].id };
  }

  private async update(block: Block): Promise<Block> {
    await this.apiClient.call('/block/updateBlock', {
      dataType: 'markdown',
      data: block.content,
      id: block.id
    });
    return block;
  }

  private mapToBlock(data: any): Block {
    return {
      id: data.id,
      type: data.type,
      content: data.content,
      parentId: data.parent_id,
      created: new Date(data.created),
      updated: new Date(data.updated)
    };
  }
}
```

### 2. Service Layer パターン

複雑なビジネスロジックを管理するためのサービス層を実装します。

```typescript
class DocumentService {
  constructor(
    private blockRepo: BlockRepository,
    private notebookRepo: NotebookRepository,
    private eventEmitter: EventEmitter
  ) {}

  async createDocumentWithTemplate(
    notebookId: string, 
    template: DocumentTemplate,
    variables: Record<string, string>
  ): Promise<Document> {
    const notebook = await this.notebookRepo.findById(notebookId);
    if (!notebook) {
      throw new Error('ノートブックが見つかりません');
    }

    const processedContent = this.processTemplate(template, variables);
    const document = await this.createDocument(notebookId, processedContent);
    
    // イベントを発火
    this.eventEmitter.emit('documentCreated', { document, template });
    
    return document;
  }

  async generateDocumentFromAI(prompt: string, notebookId: string): Promise<Document> {
    const aiContent = await this.callAIService(prompt);
    const structuredContent = this.parseAIContent(aiContent);
    
    return await this.createDocument(notebookId, structuredContent);
  }

  private processTemplate(template: DocumentTemplate, variables: Record<string, string>): string {
    let content = template.content;
    for (const [key, value] of Object.entries(variables)) {
      content = content.replace(new RegExp(`{{${key}}}`, 'g'), value);
    }
    return content;
  }
}
```

### 3. Command パターン

複雑な操作を実行可能/取り消し可能なコマンドとして実装します。

```typescript
interface Command {
  execute(): Promise<void>;
  undo(): Promise<void>;
  canUndo(): boolean;
}

class MoveBlockCommand implements Command {
  private originalParentId?: string;
  private originalIndex?: number;

  constructor(
    private blockId: string,
    private targetParentId: string,
    private apiClient: SiYuanAPIClient
  ) {}

  async execute(): Promise<void> {
    // 元の位置を記録
    const blockInfo = await this.apiClient.call('/block/getBlockInfo', {
      id: this.blockId
    });
    this.originalParentId = blockInfo.data.parent_id;
    this.originalIndex = blockInfo.data.index;

    // ブロックを移動
    await this.apiClient.call('/block/moveBlock', {
      id: this.blockId,
      parentID: this.targetParentId
    });
  }

  async undo(): Promise<void> {
    if (!this.canUndo()) {
      throw new Error('元に戻せません');
    }

    await this.apiClient.call('/block/moveBlock', {
      id: this.blockId,
      parentID: this.originalParentId,
      previousID: this.getPreviousBlockId()
    });
  }

  canUndo(): boolean {
    return !!this.originalParentId;
  }

  private async getPreviousBlockId(): Promise<string | undefined> {
    // 元のインデックス位置に基づいて前のブロックIDを計算
    // 実装の詳細は省略
    return undefined;
  }
}

class CommandManager {
  private history: Command[] = [];
  private currentIndex = -1;

  async executeCommand(command: Command): Promise<void> {
    await command.execute();
    
    // 現在位置以降の履歴を削除
    this.history = this.history.slice(0, this.currentIndex + 1);
    this.history.push(command);
    this.currentIndex++;
  }

  async undo(): Promise<void> {
    if (this.currentIndex >= 0) {
      const command = this.history[this.currentIndex];
      if (command.canUndo()) {
        await command.undo();
        this.currentIndex--;
      }
    }
  }

  async redo(): Promise<void> {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++;
      const command = this.history[this.currentIndex];
      await command.execute();
    }
  }
}
```

---

## パフォーマンス最適化

### 1. バッチ処理と並列実行

```typescript
class BatchProcessor {
  private readonly batchSize = 100;
  private readonly concurrency = 5;

  async processBulkBlockUpdates(updates: BlockUpdate[]): Promise<void> {
    // バッチに分割
    const batches = this.chunkArray(updates, this.batchSize);
    
    // 並列でバッチを処理
    await this.processBatchesConcurrently(batches, this.concurrency);
  }

  private async processBatchesConcurrently(
    batches: BlockUpdate[][], 
    concurrency: number
  ): Promise<void> {
    const semaphore = new Semaphore(concurrency);
    
    const promises = batches.map(async (batch) => {
      await semaphore.acquire();
      try {
        await this.processBatch(batch);
      } finally {
        semaphore.release();
      }
    });

    await Promise.all(promises);
  }

  private async processBatch(updates: BlockUpdate[]): Promise<void> {
    // SQLトランザクションでバッチ更新
    const sqlStatements = updates.map(update => ({
      stmt: 'UPDATE blocks SET content = ?, updated = ? WHERE id = ?',
      args: [update.content, Date.now().toString(), update.id]
    }));

    await this.apiClient.call('/query/sqlTransaction', {
      statements: sqlStatements
    });
  }

  private chunkArray<T>(array: T[], size: number): T[][] {
    const chunks: T[][] = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size));
    }
    return chunks;
  }
}

class Semaphore {
  private permits: number;
  private queue: (() => void)[] = [];

  constructor(permits: number) {
    this.permits = permits;
  }

  async acquire(): Promise<void> {
    return new Promise((resolve) => {
      if (this.permits > 0) {
        this.permits--;
        resolve();
      } else {
        this.queue.push(resolve);
      }
    });
  }

  release(): void {
    if (this.queue.length > 0) {
      const resolve = this.queue.shift()!;
      resolve();
    } else {
      this.permits++;
    }
  }
}
```

### 2. インテリジェントキャッシュ

```typescript
class SmartCache<T> {
  private cache = new Map<string, CacheEntry<T>>();
  private accessLog = new Map<string, number>();

  constructor(
    private maxSize = 1000,
    private ttl = 5 * 60 * 1000, // 5分
    private evictionPolicy: EvictionPolicy = 'LRU'
  ) {}

  async get(
    key: string, 
    fetcher: () => Promise<T>,
    options: CacheOptions = {}
  ): Promise<T> {
    const entry = this.cache.get(key);
    
    if (entry && !this.isExpired(entry)) {
      this.recordAccess(key);
      return entry.value;
    }

    // キャッシュミス：データを取得
    const value = await fetcher();
    
    // キャッシュに保存
    this.set(key, value, options);
    
    return value;
  }

  private set(key: string, value: T, options: CacheOptions): void {
    // キャッシュサイズ制限チェック
    if (this.cache.size >= this.maxSize) {
      this.evict();
    }

    const ttl = options.ttl || this.ttl;
    this.cache.set(key, {
      value,
      expiry: Date.now() + ttl,
      created: Date.now()
    });
    
    this.recordAccess(key);
  }

  private evict(): void {
    switch (this.evictionPolicy) {
      case 'LRU':
        this.evictLRU();
        break;
      case 'LFU':
        this.evictLFU();
        break;
    }
  }

  private evictLRU(): void {
    let oldestKey: string | null = null;
    let oldestAccess = Infinity;

    for (const [key, accessTime] of this.accessLog) {
      if (accessTime < oldestAccess) {
        oldestAccess = accessTime;
        oldestKey = key;
      }
    }

    if (oldestKey) {
      this.cache.delete(oldestKey);
      this.accessLog.delete(oldestKey);
    }
  }

  private recordAccess(key: string): void {
    this.accessLog.set(key, Date.now());
  }

  private isExpired(entry: CacheEntry<T>): boolean {
    return Date.now() > entry.expiry;
  }
}

interface CacheEntry<T> {
  value: T;
  expiry: number;
  created: number;
}

interface CacheOptions {
  ttl?: number;
}

type EvictionPolicy = 'LRU' | 'LFU';
```

### 3. 遅延読み込みとページネーション

```typescript
class LazyBlockLoader {
  private loadedBlocks = new Map<string, Block>();
  private loadingPromises = new Map<string, Promise<Block>>();

  async getBlock(id: string): Promise<Block> {
    // 既に読み込み済み
    if (this.loadedBlocks.has(id)) {
      return this.loadedBlocks.get(id)!;
    }

    // 読み込み中
    if (this.loadingPromises.has(id)) {
      return await this.loadingPromises.get(id)!;
    }

    // 新規読み込み
    const promise = this.loadBlockFromAPI(id);
    this.loadingPromises.set(id, promise);

    try {
      const block = await promise;
      this.loadedBlocks.set(id, block);
      return block;
    } finally {
      this.loadingPromises.delete(id);
    }
  }

  async getBlocksInRange(
    startIndex: number, 
    count: number,
    parentId?: string
  ): Promise<Block[]> {
    const sql = `
      SELECT * FROM blocks 
      ${parentId ? 'WHERE parent_id = ?' : ''}
      ORDER BY created DESC 
      LIMIT ? OFFSET ?
    `;
    
    const args = parentId ? [parentId, count, startIndex] : [count, startIndex];
    
    const result = await this.apiClient.call('/query/sql', {
      stmt: sql,
      args
    });

    return result.data.map(this.mapToBlock);
  }

  // 無限スクロール用のページネーション
  async *getPaginatedBlocks(
    pageSize = 50,
    parentId?: string
  ): AsyncGenerator<Block[], void, unknown> {
    let offset = 0;
    
    while (true) {
      const blocks = await this.getBlocksInRange(offset, pageSize, parentId);
      
      if (blocks.length === 0) {
        break;
      }
      
      yield blocks;
      offset += pageSize;
      
      // 最後のページ
      if (blocks.length < pageSize) {
        break;
      }
    }
  }
}
```

---

## エラーハンドリング戦略

### 1. 階層化されたエラーハンドリング

```typescript
// カスタムエラークラス
class SiYuanAPIError extends Error {
  constructor(
    public code: number,
    message: string,
    public endpoint?: string,
    public requestData?: any
  ) {
    super(message);
    this.name = 'SiYuanAPIError';
  }
}

class RetryableError extends SiYuanAPIError {
  constructor(code: number, message: string, public retryAfter?: number) {
    super(code, message);
    this.name = 'RetryableError';
  }
}

class ValidationError extends Error {
  constructor(
    message: string,
    public field: string,
    public value: any
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

// エラーハンドラーファクトリー
class ErrorHandler {
  private strategies = new Map<string, ErrorStrategy>();

  constructor() {
    this.registerDefaultStrategies();
  }

  private registerDefaultStrategies(): void {
    this.strategies.set('NetworkError', new NetworkErrorStrategy());
    this.strategies.set('RateLimitError', new RateLimitErrorStrategy());
    this.strategies.set('ValidationError', new ValidationErrorStrategy());
    this.strategies.set('AuthenticationError', new AuthErrorStrategy());
  }

  async handleError(error: Error, context: ErrorContext): Promise<any> {
    const strategy = this.getStrategy(error);
    return await strategy.handle(error, context);
  }

  private getStrategy(error: Error): ErrorStrategy {
    const strategyName = this.determineStrategyName(error);
    return this.strategies.get(strategyName) || new DefaultErrorStrategy();
  }

  private determineStrategyName(error: Error): string {
    if (error instanceof ValidationError) return 'ValidationError';
    if (error instanceof SiYuanAPIError) {
      if (error.code === 401) return 'AuthenticationError';
      if (error.code === 429) return 'RateLimitError';
    }
    return 'NetworkError';
  }
}

interface ErrorStrategy {
  handle(error: Error, context: ErrorContext): Promise<any>;
}

class RateLimitErrorStrategy implements ErrorStrategy {
  async handle(error: RetryableError, context: ErrorContext): Promise<any> {
    const retryAfter = error.retryAfter || 1000;
    
    console.warn(`Rate limit exceeded. Retrying after ${retryAfter}ms`);
    
    await new Promise(resolve => setTimeout(resolve, retryAfter));
    
    // 元のリクエストを再実行
    return await context.retry();
  }
}
```

### 2. リトライメカニズム

```typescript
class RetryManager {
  async executeWithRetry<T>(
    operation: () => Promise<T>,
    options: RetryOptions = {}
  ): Promise<T> {
    const {
      maxRetries = 3,
      baseDelay = 1000,
      maxDelay = 10000,
      backoffFactor = 2,
      jitter = true
    } = options;

    let lastError: Error;

    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error as Error;

        if (attempt === maxRetries || !this.isRetryableError(error)) {
          throw error;
        }

        const delay = this.calculateDelay(attempt, baseDelay, maxDelay, backoffFactor, jitter);
        console.warn(`Attempt ${attempt + 1} failed, retrying in ${delay}ms:`, error);
        
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }

    throw lastError!;
  }

  private isRetryableError(error: any): boolean {
    if (error instanceof RetryableError) return true;
    if (error instanceof SiYuanAPIError) {
      // 5xx エラーは再試行可能
      return error.code >= 500 && error.code < 600;
    }
    return false;
  }

  private calculateDelay(
    attempt: number,
    baseDelay: number,
    maxDelay: number,
    backoffFactor: number,
    jitter: boolean
  ): number {
    let delay = Math.min(baseDelay * Math.pow(backoffFactor, attempt), maxDelay);
    
    if (jitter) {
      delay *= (0.5 + Math.random() * 0.5); // ±25% のジッター
    }
    
    return Math.floor(delay);
  }
}

interface RetryOptions {
  maxRetries?: number;
  baseDelay?: number;
  maxDelay?: number;
  backoffFactor?: number;
  jitter?: boolean;
}
```

---

## データ同期とキャッシュ

### 1. リアルタイムデータ同期

```typescript
class RealtimeSync {
  private websocket: WebSocket | null = null;
  private subscribers = new Map<string, Set<SyncCallback>>();
  private lastSyncTimestamp = 0;

  constructor(private apiClient: SiYuanAPIClient) {}

  async initialize(): Promise<void> {
    // WebSocket接続を確立
    await this.establishWebSocketConnection();
    
    // 初期同期を実行
    await this.performInitialSync();
    
    // 定期的な健全性チェックを開始
    this.startHealthCheck();
  }

  private async establishWebSocketConnection(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.websocket = new WebSocket('ws://localhost:6806/ws');
      
      this.websocket.onopen = () => {
        console.log('WebSocket connection established');
        resolve();
      };

      this.websocket.onmessage = (event) => {
        this.handleWebSocketMessage(event);
      };

      this.websocket.onerror = (error) => {
        console.error('WebSocket error:', error);
        reject(error);
      };

      this.websocket.onclose = () => {
        console.warn('WebSocket connection closed');
        this.scheduleReconnection();
      };
    });
  }

  private handleWebSocketMessage(event: MessageEvent): void {
    try {
      const message = JSON.parse(event.data);
      this.processSyncMessage(message);
    } catch (error) {
      console.error('Failed to parse WebSocket message:', error);
    }
  }

  private processeSyncMessage(message: SyncMessage): void {
    const { type, blockId, data } = message;
    
    // サブスクライバーに通知
    const callbacks = this.subscribers.get(blockId) || new Set();
    callbacks.forEach(callback => {
      try {
        callback(type, blockId, data);
      } catch (error) {
        console.error('Subscriber callback error:', error);
      }
    });

    // グローバルリスナーにも通知
    const globalCallbacks = this.subscribers.get('*') || new Set();
    globalCallbacks.forEach(callback => {
      try {
        callback(type, blockId, data);
      } catch (error) {
        console.error('Global subscriber callback error:', error);
      }
    });
  }

  subscribe(blockId: string, callback: SyncCallback): () => void {
    if (!this.subscribers.has(blockId)) {
      this.subscribers.set(blockId, new Set());
    }
    
    this.subscribers.get(blockId)!.add(callback);
    
    // アンサブスクライブ関数を返す
    return () => {
      const callbacks = this.subscribers.get(blockId);
      if (callbacks) {
        callbacks.delete(callback);
        if (callbacks.size === 0) {
          this.subscribers.delete(blockId);
        }
      }
    };
  }

  async forceSync(): Promise<void> {
    const timestamp = Date.now();
    
    // 最後の同期以降の変更を取得
    const changes = await this.apiClient.call('/sync/getChanges', {
      since: this.lastSyncTimestamp
    });

    // 変更を適用
    for (const change of changes.data) {
      this.processeSyncMessage(change);
    }

    this.lastSyncTimestamp = timestamp;
  }
}

type SyncCallback = (type: string, blockId: string, data: any) => void;

interface SyncMessage {
  type: 'created' | 'updated' | 'deleted';
  blockId: string;
  data: any;
  timestamp: number;
}
```

### 2. 分散キャッシュシステム

```typescript
class DistributedCache {
  private localCache = new Map<string, CacheEntry>();
  private remoteCache: RemoteCacheClient;

  constructor(remoteConfig: RemoteCacheConfig) {
    this.remoteCache = new RemoteCacheClient(remoteConfig);
  }

  async get<T>(key: string): Promise<T | null> {
    // L1: ローカルキャッシュから取得
    const localEntry = this.localCache.get(key);
    if (localEntry && !this.isExpired(localEntry)) {
      return localEntry.value;
    }

    // L2: リモートキャッシュから取得
    const remoteValue = await this.remoteCache.get<T>(key);
    if (remoteValue !== null) {
      // ローカルキャッシュに保存
      this.localCache.set(key, {
        value: remoteValue,
        expiry: Date.now() + 60000, // 1分
        version: await this.getVersion(key)
      });
      return remoteValue;
    }

    return null;
  }

  async set<T>(key: string, value: T, ttl: number = 300000): Promise<void> {
    const version = await this.incrementVersion(key);
    
    // ローカルキャッシュに保存
    this.localCache.set(key, {
      value,
      expiry: Date.now() + Math.min(ttl, 60000),
      version
    });

    // リモートキャッシュに保存
    await this.remoteCache.set(key, value, ttl, version);

    // 他のインスタンスに無効化通知
    await this.notifyInvalidation(key, version);
  }

  async invalidate(key: string): Promise<void> {
    this.localCache.delete(key);
    await this.remoteCache.delete(key);
    await this.notifyInvalidation(key);
  }

  private async notifyInvalidation(key: string, version?: number): Promise<void> {
    // 他のキャッシュインスタンスに無効化を通知
    // Redis Pub/Sub や WebSocket を使用
    await this.remoteCache.publish('cache:invalidate', {
      key,
      version,
      timestamp: Date.now()
    });
  }

  handleInvalidationNotification(message: InvalidationMessage): void {
    const entry = this.localCache.get(message.key);
    if (entry && (!message.version || entry.version < message.version)) {
      this.localCache.delete(message.key);
    }
  }
}
```

---

## バッチ処理とトランザクション

### 1. 高度なバッチ処理

```typescript
class AdvancedBatchProcessor {
  private readonly maxBatchSize = 1000;
  private readonly maxConcurrency = 10;
  private processingQueue: BatchItem[] = [];
  private isProcessing = false;

  async addToBatch<T>(operation: BatchOperation<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      const item: BatchItem = {
        operation,
        resolve,
        reject,
        timestamp: Date.now()
      };

      this.processingQueue.push(item);
      this.scheduleProcessing();
    });
  }

  private scheduleProcessing(): void {
    if (this.isProcessing) return;

    // デバウンス：短時間での複数の追加をまとめて処理
    setTimeout(() => {
      this.processBatch();
    }, 10);
  }

  private async processBatch(): Promise<void> {
    if (this.isProcessing || this.processingQueue.length === 0) return;

    this.isProcessing = true;

    try {
      // バッチを作成
      const currentBatch = this.processingQueue.splice(0, this.maxBatchSize);
      
      // 操作タイプ別にグループ化
      const groupedOperations = this.groupOperationsByType(currentBatch);
      
      // 各グループを並列で処理
      const processingPromises = Array.from(groupedOperations.entries()).map(
        ([type, operations]) => this.processOperationGroup(type, operations)
      );

      await Promise.all(processingPromises);
      
    } catch (error) {
      console.error('Batch processing error:', error);
      // エラーが発生した場合、残りのアイテムを reject
      this.processingQueue.forEach(item => item.reject(error));
      this.processingQueue = [];
    } finally {
      this.isProcessing = false;
      
      // まだ処理待ちがあれば再スケジュール
      if (this.processingQueue.length > 0) {
        this.scheduleProcessing();
      }
    }
  }

  private async processOperationGroup(
    type: string,
    items: BatchItem[]
  ): Promise<void> {
    switch (type) {
      case 'blockUpdate':
        await this.processBlockUpdateBatch(items);
        break;
      case 'blockInsert':
        await this.processBlockInsertBatch(items);
        break;
      case 'blockDelete':
        await this.processBlockDeleteBatch(items);
        break;
      default:
        // 個別処理
        await this.processIndividualOperations(items);
    }
  }

  private async processBlockUpdateBatch(items: BatchItem[]): Promise<void> {
    // SQLトランザクションでまとめて更新
    const statements = items.map(item => ({
      stmt: 'UPDATE blocks SET content = ?, updated = ? WHERE id = ?',
      args: [
        item.operation.data.content,
        Date.now().toString(),
        item.operation.data.id
      ]
    }));

    try {
      const result = await this.apiClient.call('/query/sqlTransaction', {
        statements
      });

      // 各アイテムに結果を通知
      items.forEach((item, index) => {
        item.resolve(result.data[index]);
      });
    } catch (error) {
      items.forEach(item => item.reject(error));
    }
  }
}

interface BatchOperation<T> {
  type: string;
  data: any;
  execute(): Promise<T>;
}

interface BatchItem {
  operation: BatchOperation<any>;
  resolve: (value: any) => void;
  reject: (error: Error) => void;
  timestamp: number;
}
```

### 2. 分散トランザクション

```typescript
class DistributedTransaction {
  private operations: TransactionOperation[] = [];
  private compensations: CompensationOperation[] = [];
  private status: TransactionStatus = 'pending';

  async addOperation(operation: TransactionOperation): Promise<void> {
    this.operations.push(operation);
  }

  async execute(): Promise<TransactionResult> {
    this.status = 'executing';
    const executedOperations: TransactionOperation[] = [];

    try {
      // フェーズ1: 準備フェーズ
      await this.preparePhase();

      // フェーズ2: 実行フェーズ
      for (const operation of this.operations) {
        await operation.execute();
        executedOperations.push(operation);
        
        // 補償操作を記録
        if (operation.getCompensation) {
          this.compensations.unshift(operation.getCompensation());
        }
      }

      // フェーズ3: コミットフェーズ
      await this.commitPhase();
      
      this.status = 'committed';
      return { success: true, operations: executedOperations };

    } catch (error) {
      console.error('Transaction failed:', error);
      
      // ロールバック実行
      await this.rollback();
      
      this.status = 'aborted';
      return { 
        success: false, 
        error, 
        operations: executedOperations 
      };
    }
  }

  private async preparePhase(): Promise<void> {
    const preparePromises = this.operations.map(op => 
      op.prepare ? op.prepare() : Promise.resolve()
    );
    
    await Promise.all(preparePromises);
  }

  private async commitPhase(): Promise<void> {
    const commitPromises = this.operations.map(op =>
      op.commit ? op.commit() : Promise.resolve()
    );
    
    await Promise.all(commitPromises);
  }

  private async rollback(): Promise<void> {
    for (const compensation of this.compensations) {
      try {
        await compensation.execute();
      } catch (error) {
        console.error('Compensation failed:', error);
        // 補償に失敗しても続行
      }
    }
  }
}

interface TransactionOperation {
  prepare?(): Promise<void>;
  execute(): Promise<any>;
  commit?(): Promise<void>;
  getCompensation?(): CompensationOperation;
}

interface CompensationOperation {
  execute(): Promise<void>;
}

class BlockUpdateTransaction implements TransactionOperation {
  constructor(
    private blockId: string,
    private newContent: string,
    private apiClient: SiYuanAPIClient
  ) {}

  private originalContent?: string;

  async prepare(): Promise<void> {
    // 現在の内容を保存（ロールバック用）
    const result = await this.apiClient.call('/block/getBlockInfo', {
      id: this.blockId
    });
    this.originalContent = result.data.content;
  }

  async execute(): Promise<void> {
    await this.apiClient.call('/block/updateBlock', {
      dataType: 'markdown',
      data: this.newContent,
      id: this.blockId
    });
  }

  getCompensation(): CompensationOperation {
    return {
      execute: async () => {
        if (this.originalContent !== undefined) {
          await this.apiClient.call('/block/updateBlock', {
            dataType: 'markdown',
            data: this.originalContent,
            id: this.blockId
          });
        }
      }
    };
  }
}

type TransactionStatus = 'pending' | 'executing' | 'committed' | 'aborted';

interface TransactionResult {
  success: boolean;
  operations: TransactionOperation[];
  error?: Error;
}
```

---

## 実践的な応用例

### 1. インテリジェントな文書分析システム

```typescript
class DocumentAnalyzer {
  constructor(
    private apiClient: SiYuanAPIClient,
    private nlpService: NLPService,
    private cache: SmartCache<any>
  ) {}

  async analyzeDocument(documentId: string): Promise<DocumentAnalysis> {
    const cacheKey = `analysis:${documentId}`;
    
    return await this.cache.get(cacheKey, async () => {
      // ドキュメントの全ブロックを取得
      const blocks = await this.getDocumentBlocks(documentId);
      
      // 並列で各種分析を実行
      const [
        readability,
        sentiments,
        keyPhrases,
        structure,
        links
      ] = await Promise.all([
        this.analyzeReadability(blocks),
        this.analyzeSentiments(blocks),
        this.extractKeyPhrases(blocks),
        this.analyzeStructure(blocks),
        this.analyzeLinks(blocks)
      ]);

      return {
        documentId,
        readability,
        sentiments,
        keyPhrases,
        structure,
        links,
        analyzedAt: new Date()
      };
    }, { ttl: 30 * 60 * 1000 }); // 30分キャッシュ
  }

  private async analyzeReadability(blocks: Block[]): Promise<ReadabilityScore> {
    const textBlocks = blocks.filter(b => b.type === 'p');
    const fullText = textBlocks.map(b => b.content).join(' ');
    
    return await this.nlpService.calculateReadability(fullText);
  }

  private async analyzeSentiments(blocks: Block[]): Promise<SentimentAnalysis[]> {
    const textBlocks = blocks.filter(b => b.type === 'p');
    
    // バッチでセンチメント分析
    const batchProcessor = new BatchProcessor();
    const sentimentPromises = textBlocks.map(block =>
      batchProcessor.addToBatch({
        type: 'sentiment',
        data: { text: block.content, blockId: block.id },
        execute: () => this.nlpService.analyzeSentiment(block.content)
      })
    );

    return await Promise.all(sentimentPromises);
  }

  async generateDocumentSuggestions(
    documentId: string
  ): Promise<DocumentSuggestion[]> {
    const analysis = await this.analyzeDocument(documentId);
    const suggestions: DocumentSuggestion[] = [];

    // 可読性に基づく提案
    if (analysis.readability.score < 60) {
      suggestions.push({
        type: 'readability',
        priority: 'high',
        message: '文章が複雑すぎます。短い文章に分割することを検討してください。',
        actionable: true,
        autoFix: async () => this.splitLongSentences(documentId)
      });
    }

    // 構造に基づく提案
    if (analysis.structure.headingDepth > 4) {
      suggestions.push({
        type: 'structure',
        priority: 'medium',
        message: '見出しの階層が深すぎます。文書構造を見直してください。',
        actionable: false
      });
    }

    // センチメントに基づく提案
    const negativeSentiments = analysis.sentiments.filter(s => s.score < -0.5);
    if (negativeSentiments.length > analysis.sentiments.length * 0.3) {
      suggestions.push({
        type: 'tone',
        priority: 'low',
        message: 'ネガティブな表現が多く含まれています。トーンの調整を検討してください。',
        actionable: true,
        autoFix: async () => this.suggestToneImprovement(documentId, negativeSentiments)
      });
    }

    return suggestions;
  }

  private async splitLongSentences(documentId: string): Promise<void> {
    const blocks = await this.getDocumentBlocks(documentId);
    const transaction = new DistributedTransaction();

    for (const block of blocks) {
      if (block.type === 'p' && this.isLongSentence(block.content)) {
        const splitSentences = this.splitSentence(block.content);
        
        // 元のブロックを更新
        transaction.addOperation(new BlockUpdateTransaction(
          block.id,
          splitSentences[0],
          this.apiClient
        ));

        // 追加の文を新しいブロックとして挿入
        for (let i = 1; i < splitSentences.length; i++) {
          transaction.addOperation(new BlockInsertTransaction(
            block.parentId,
            splitSentences[i],
            block.id, // 前のブロックの後に挿入
            this.apiClient
          ));
        }
      }
    }

    await transaction.execute();
  }
}

interface DocumentAnalysis {
  documentId: string;
  readability: ReadabilityScore;
  sentiments: SentimentAnalysis[];
  keyPhrases: KeyPhrase[];
  structure: StructureAnalysis;
  links: LinkAnalysis;
  analyzedAt: Date;
}

interface DocumentSuggestion {
  type: string;
  priority: 'high' | 'medium' | 'low';
  message: string;
  actionable: boolean;
  autoFix?: () => Promise<void>;
}
```

### 2. AIアシスタント統合システム

```typescript
class AIAssistantIntegration {
  private conversationHistory = new Map<string, ConversationContext>();

  constructor(
    private apiClient: SiYuanAPIClient,
    private aiService: AIService,
    private documentAnalyzer: DocumentAnalyzer
  ) {}

  async processNaturalLanguageCommand(
    command: string,
    context: CommandContext
  ): Promise<CommandResult> {
    // 意図を分析
    const intent = await this.aiService.analyzeIntent(command);
    
    // 文脈を構築
    const fullContext = await this.buildContext(context, intent);
    
    // コマンドを実行
    switch (intent.action) {
      case 'create_document':
        return await this.createDocumentFromNL(intent, fullContext);
      
      case 'search_content':
        return await this.searchContentFromNL(intent, fullContext);
      
      case 'analyze_document':
        return await this.analyzeDocumentFromNL(intent, fullContext);
      
      case 'summarize':
        return await this.summarizeContentFromNL(intent, fullContext);
        
      case 'generate_outline':
        return await this.generateOutlineFromNL(intent, fullContext);
        
      default:
        return await this.handleGenericQuery(intent, fullContext);
    }
  }

  private async createDocumentFromNL(
    intent: NLIntent,
    context: CommandContext
  ): Promise<CommandResult> {
    const { topic, style, length } = intent.parameters;
    
    // AIを使用してコンテンツを生成
    const generatedContent = await this.aiService.generateContent({
      topic,
      style: style || 'professional',
      length: length || 'medium',
      context: context.relatedDocuments
    });

    // ドキュメントを作成
    const result = await this.apiClient.call('/filetree/createDocWithMd', {
      notebook: context.currentNotebook,
      path: `/${topic}.md`,
      markdown: generatedContent
    });

    // 会話履歴を更新
    this.updateConversationHistory(context.userId, {
      action: 'create_document',
      result: result.data,
      feedback: null
    });

    return {
      success: true,
      message: `「${topic}」に関するドキュメントを作成しました`,
      data: result.data,
      followUp: [
        '内容を修正したい箇所はありますか？',
        '関連する情報を追加しますか？',
        'このドキュメントをベースに別のバージョンを作成しますか？'
      ]
    };
  }

  private async searchContentFromNL(
    intent: NLIntent,
    context: CommandContext
  ): Promise<CommandResult> {
    const { query, timeRange, contentType } = intent.parameters;
    
    // セマンティック検索を実行
    const semanticResults = await this.performSemanticSearch(query, {
      timeRange,
      contentType,
      context: context.currentNotebook
    });

    // 結果をランク付け
    const rankedResults = await this.rankSearchResults(
      semanticResults,
      query,
      context
    );

    return {
      success: true,
      message: `「${query}」に関連する ${rankedResults.length} 件の結果を見つけました`,
      data: rankedResults,
      followUp: [
        'より具体的な検索条件を指定しますか？',
        '結果をフィルタリングしますか？',
        '関連する新しいドキュメントを作成しますか？'
      ]
    };
  }

  private async performSemanticSearch(
    query: string,
    options: SearchOptions
  ): Promise<SearchResult[]> {
    // クエリの埋め込みベクトルを生成
    const queryEmbedding = await this.aiService.generateEmbedding(query);
    
    // ドキュメントの埋め込みベクトルと比較
    const documents = await this.getAllDocuments(options);
    const similarities: Array<{ document: any, similarity: number }> = [];

    for (const doc of documents) {
      const docEmbedding = await this.getOrGenerateEmbedding(doc);
      const similarity = this.cosineSimilarity(queryEmbedding, docEmbedding);
      
      if (similarity > 0.7) { // しきい値
        similarities.push({ document: doc, similarity });
      }
    }

    // 類似度でソート
    similarities.sort((a, b) => b.similarity - a.similarity);
    
    return similarities.map(s => ({
      document: s.document,
      score: s.similarity,
      relevantPhrases: this.extractRelevantPhrases(s.document.content, query)
    }));
  }

  async startConversationalSession(
    userId: string,
    initialContext: CommandContext
  ): Promise<ConversationSession> {
    const sessionId = this.generateSessionId();
    
    const session: ConversationSession = {
      id: sessionId,
      userId,
      context: initialContext,
      history: [],
      startedAt: new Date(),
      isActive: true
    };

    this.conversationHistory.set(sessionId, {
      session,
      lastActivity: new Date()
    });

    return session;
  }

  async continueConversation(
    sessionId: string,
    message: string
  ): Promise<ConversationResponse> {
    const conversationContext = this.conversationHistory.get(sessionId);
    if (!conversationContext) {
      throw new Error('Conversation session not found');
    }

    // 文脈を更新
    conversationContext.session.history.push({
      role: 'user',
      content: message,
      timestamp: new Date()
    });

    // AIレスポンスを生成
    const response = await this.aiService.generateConversationalResponse({
      message,
      history: conversationContext.session.history,
      context: conversationContext.session.context
    });

    // アクションが含まれている場合は実行
    if (response.actions) {
      const actionResults = await this.executeActions(
        response.actions,
        conversationContext.session.context
      );
      response.actionResults = actionResults;
    }

    // 履歴を更新
    conversationContext.session.history.push({
      role: 'assistant',
      content: response.message,
      actions: response.actions,
      timestamp: new Date()
    });

    conversationContext.lastActivity = new Date();

    return response;
  }
}

interface NLIntent {
  action: string;
  confidence: number;
  parameters: Record<string, any>;
}

interface CommandContext {
  userId: string;
  currentNotebook: string;
  currentDocument?: string;
  relatedDocuments: string[];
  userPreferences: UserPreferences;
}

interface ConversationSession {
  id: string;
  userId: string;
  context: CommandContext;
  history: ConversationMessage[];
  startedAt: Date;
  isActive: boolean;
}
```

---

## まとめ

このガイドでは、SiYuan APIを活用した高度なプログラミングパターンを紹介しました。これらのパターンを適切に組み合わせることで、スケーラブルで保守性の高いSiYuanアプリケーションを構築できます。

### 重要なポイント

1. **アーキテクチャ設計**: Repository パターンや Service Layer パターンを使用してコードを整理
2. **パフォーマンス**: バッチ処理、キャッシュ、並列実行を適切に活用
3. **エラー処理**: 階層化されたエラーハンドリングとリトライメカニズム
4. **データ同期**: リアルタイム同期と分散キャッシュによる一貫性の確保
5. **トランザクション**: 分散トランザクションによる整合性の保持

### 次のステップ

- [プラグイン開発ベストプラクティス](PLUGIN_BEST_PRACTICES_ja_JP.md) - プラグイン特有の開発手法
- [実際のプラグイン例](PLUGIN_SHOWCASE_ja_JP.md) - 具体的な実装例
- [完全なAPIリファレンス](API_ja_JP.md) - 全エンドポイントの詳細

---

*このガイドは継続的に更新されます。新しいパターンやベストプラクティスが確立されたら、追加していきます。*
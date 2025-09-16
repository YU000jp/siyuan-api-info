# SiYuan Advanced API Patterns

Advanced programming patterns and best practices for leveraging the SiYuan API effectively.

**Target Audience**: Developers who have mastered basic SiYuan API operations

**Prerequisites**:
- Basic SiYuan API operations
- Intermediate or advanced JavaScript/TypeScript knowledge
- Understanding of asynchronous programming and database design

**Related Documentation**:
- [Beginner API Guide](API_BEGINNERS.md) - Basic API usage
- [Plugin Developer API Guide](API_PLUGIN_DEVELOPERS.md) - Plugin development focused
- [Complete API Reference](API.md) - All endpoints

---

## Table of Contents

- [Architecture Patterns](#architecture-patterns)
- [Performance Optimization](#performance-optimization)
- [Error Handling Strategy](#error-handling-strategy)
- [Data Synchronization and Caching](#data-synchronization-and-caching)
- [Batch Processing and Transactions](#batch-processing-and-transactions)
- [Real-time Updates](#real-time-updates)
- [Security Best Practices](#security-best-practices)
- [Testing and Monitoring](#testing-and-monitoring)
- [Scalability Considerations](#scalability-considerations)
- [Practical Applications](#practical-applications)

---

## Architecture Patterns

### 1. Repository Pattern

Abstract data access to separate API details from business logic.

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
      const result = await this.apiClient.call('/query/sql', {
        stmt: 'SELECT * FROM blocks WHERE id = ?',
        params: [id]
      });
      return result.length > 0 ? this.mapToBlock(result[0]) : null;
    } catch (error) {
      throw new BlockNotFoundError(id, error);
    }
  }

  async findByType(type: string): Promise<Block[]> {
    const result = await this.apiClient.call('/query/sql', {
      stmt: 'SELECT * FROM blocks WHERE type = ? ORDER BY created',
      params: [type]
    });
    return result.map(row => this.mapToBlock(row));
  }

  private mapToBlock(row: any): Block {
    return new Block(
      row.id,
      row.content,
      row.type,
      new Date(row.created),
      new Date(row.updated)
    );
  }
}
```

**Benefits:**
- Testable business logic
- Centralized error handling
- Easy to swap implementations
- Clear separation of concerns

### 2. Service Layer Pattern

Encapsulate complex business operations in service classes.

```typescript
class NoteOrganizationService {
  constructor(
    private blockRepo: BlockRepository,
    private notebookRepo: NotebookRepository,
    private logger: Logger
  ) {}

  async reorganizeByTags(): Promise<ReorganizationResult> {
    this.logger.info('Starting note reorganization by tags');
    
    const taggedBlocks = await this.findAllTaggedBlocks();
    const organizationPlan = this.createOrganizationPlan(taggedBlocks);
    
    const results = await this.executeOrganizationPlan(organizationPlan);
    
    this.logger.info(`Reorganization completed: ${results.moved} blocks moved`);
    return results;
  }

  private async findAllTaggedBlocks(): Promise<TaggedBlock[]> {
    return await this.blockRepo.findWithCustomAttributes('tags');
  }

  private createOrganizationPlan(blocks: TaggedBlock[]): OrganizationPlan {
    const plan = new OrganizationPlan();
    
    for (const block of blocks) {
      const targetNotebook = this.determineTargetNotebook(block.tags);
      plan.addMove(block, targetNotebook);
    }
    
    return plan;
  }
}
```

### 3. Factory Pattern for API Clients

Create specialized API clients for different use cases.

```typescript
class SiYuanAPIFactory {
  static createForPlugin(pluginId: string): PluginAPIClient {
    return new PluginAPIClient(pluginId, this.getDefaultConfig());
  }

  static createForAutomation(): AutomationAPIClient {
    return new AutomationAPIClient(this.getAutomationConfig());
  }

  static createForIntegration(service: string): IntegrationAPIClient {
    return new IntegrationAPIClient(service, this.getIntegrationConfig());
  }

  private static getDefaultConfig(): APIConfig {
    return {
      baseURL: 'http://localhost:6806',
      timeout: 30000,
      retries: 3,
      rateLimiting: { requestsPerSecond: 10 }
    };
  }
}

// Usage
const pluginClient = SiYuanAPIFactory.createForPlugin('my-plugin');
const automationClient = SiYuanAPIFactory.createForAutomation();
```

---

## Performance Optimization

### 1. Request Batching

Combine multiple API calls to reduce network overhead.

```typescript
class BatchAPIClient {
  private batchQueue: APIRequest[] = [];
  private batchTimeout?: NodeJS.Timeout;
  
  async call(endpoint: string, data: any): Promise<any> {
    return new Promise((resolve, reject) => {
      const request: APIRequest = {
        endpoint,
        data,
        resolve,
        reject,
        timestamp: Date.now()
      };
      
      this.batchQueue.push(request);
      this.scheduleBatchExecution();
    });
  }

  private scheduleBatchExecution() {
    if (this.batchTimeout) return;
    
    this.batchTimeout = setTimeout(() => {
      this.executeBatch();
    }, 50); // 50ms batch window
  }

  private async executeBatch() {
    if (this.batchQueue.length === 0) return;
    
    const requests = [...this.batchQueue];
    this.batchQueue = [];
    this.batchTimeout = undefined;

    // Group by endpoint type for optimal batching
    const groups = this.groupRequestsByType(requests);
    
    for (const [type, groupRequests] of Object.entries(groups)) {
      await this.executeBatchGroup(type, groupRequests);
    }
  }

  private async executeBatchGroup(type: string, requests: APIRequest[]) {
    switch (type) {
      case 'query':
        await this.executeSQLBatch(requests);
        break;
      case 'block':
        await this.executeBlockBatch(requests);
        break;
      default:
        await this.executeSequential(requests);
    }
  }
}
```

### 2. Intelligent Caching

Implement multi-level caching with invalidation strategies.

```typescript
class SmartCache {
  private l1Cache = new Map<string, CacheEntry>(); // Memory cache
  private l2Cache?: IDBDatabase; // IndexedDB cache
  private invalidationRules = new Map<string, InvalidationRule>();

  async get<T>(key: string, fetcher: () => Promise<T>): Promise<T> {
    // Check L1 cache
    const l1Entry = this.l1Cache.get(key);
    if (l1Entry && !this.isExpired(l1Entry)) {
      return l1Entry.value;
    }

    // Check L2 cache
    const l2Entry = await this.getFromL2Cache(key);
    if (l2Entry && !this.isExpired(l2Entry)) {
      // Promote to L1
      this.l1Cache.set(key, l2Entry);
      return l2Entry.value;
    }

    // Fetch fresh data
    const value = await fetcher();
    const entry = this.createCacheEntry(key, value);
    
    // Store in both caches
    this.l1Cache.set(key, entry);
    await this.setL2Cache(key, entry);
    
    return value;
  }

  setupInvalidation(pattern: string, rule: InvalidationRule) {
    this.invalidationRules.set(pattern, rule);
  }

  async invalidate(event: CacheInvalidationEvent) {
    for (const [pattern, rule] of this.invalidationRules) {
      if (rule.matches(event)) {
        await this.invalidatePattern(pattern);
      }
    }
  }
}

// Usage
const cache = new SmartCache();

cache.setupInvalidation('blocks:*', {
  matches: (event) => event.type === 'block_updated',
  patterns: ['blocks:*', 'search:*']
});

const blocks = await cache.get('blocks:notebook:123', async () => {
  return await api.call('/query/sql', {
    stmt: 'SELECT * FROM blocks WHERE box = ?',
    params: ['123']
  });
});
```

### 3. Connection Pool Management

Manage multiple API connections efficiently.

```typescript
class ConnectionPool {
  private connections: Map<string, APIConnection> = new Map();
  private connectionQueue: Connection[] = [];
  private maxConnections = 5;
  private minConnections = 2;

  async getConnection(priority = 'normal'): Promise<APIConnection> {
    const available = this.findAvailableConnection();
    if (available) {
      return available;
    }

    if (this.connections.size < this.maxConnections) {
      return await this.createNewConnection();
    }

    return await this.waitForConnection(priority);
  }

  private async createNewConnection(): Promise<APIConnection> {
    const connection = new APIConnection({
      baseURL: 'http://localhost:6806',
      keepAlive: true,
      timeout: 30000
    });

    await connection.connect();
    this.connections.set(connection.id, connection);
    
    // Set up connection health monitoring
    this.monitorConnectionHealth(connection);
    
    return connection;
  }

  private monitorConnectionHealth(connection: APIConnection) {
    const healthCheck = setInterval(async () => {
      try {
        await connection.ping();
      } catch (error) {
        this.handleUnhealthyConnection(connection);
        clearInterval(healthCheck);
      }
    }, 30000);
  }
}
```

---

## Error Handling Strategy

### 1. Hierarchical Error Classification

Classify errors for appropriate handling strategies.

```typescript
abstract class SiYuanAPIError extends Error {
  abstract readonly category: ErrorCategory;
  abstract readonly retryable: boolean;
  abstract readonly statusCode?: number;
}

class NetworkError extends SiYuanAPIError {
  readonly category = ErrorCategory.NETWORK;
  readonly retryable = true;
  
  constructor(message: string, public readonly originalError: Error) {
    super(`Network error: ${message}`);
  }
}

class AuthenticationError extends SiYuanAPIError {
  readonly category = ErrorCategory.AUTHENTICATION;
  readonly retryable = false;
  readonly statusCode = 401;
  
  constructor(message = 'Invalid API token') {
    super(message);
  }
}

class RateLimitError extends SiYuanAPIError {
  readonly category = ErrorCategory.RATE_LIMIT;
  readonly retryable = true;
  readonly statusCode = 429;
  
  constructor(public readonly retryAfter: number) {
    super(`Rate limit exceeded, retry after ${retryAfter}ms`);
  }
}

// Error handler with strategy pattern
class ErrorHandler {
  private strategies = new Map<ErrorCategory, ErrorHandlingStrategy>();

  constructor() {
    this.strategies.set(ErrorCategory.NETWORK, new NetworkErrorStrategy());
    this.strategies.set(ErrorCategory.RATE_LIMIT, new RateLimitErrorStrategy());
    this.strategies.set(ErrorCategory.AUTHENTICATION, new AuthErrorStrategy());
  }

  async handle(error: SiYuanAPIError, context: ErrorContext): Promise<ErrorHandlingResult> {
    const strategy = this.strategies.get(error.category);
    if (!strategy) {
      return { action: 'fail', error };
    }

    return await strategy.handle(error, context);
  }
}
```

### 2. Retry Logic with Exponential Backoff

Implement intelligent retry mechanisms.

```typescript
class RetryManager {
  async executeWithRetry<T>(
    operation: () => Promise<T>,
    options: RetryOptions = {}
  ): Promise<T> {
    const config = { ...this.defaultOptions, ...options };
    let lastError: Error;

    for (let attempt = 0; attempt < config.maxRetries; attempt++) {
      try {
        return await this.executeWithTimeout(operation, config.timeout);
      } catch (error) {
        lastError = error;
        
        if (!this.isRetryableError(error) || attempt === config.maxRetries - 1) {
          throw error;
        }

        const backoffDelay = this.calculateBackoffDelay(attempt, config);
        await this.delay(backoffDelay);
        
        this.logRetryAttempt(attempt, error, backoffDelay);
      }
    }

    throw lastError!;
  }

  private calculateBackoffDelay(attempt: number, config: RetryOptions): number {
    const baseDelay = config.baseDelay || 1000;
    const exponentialDelay = baseDelay * Math.pow(2, attempt);
    const jitter = Math.random() * 0.1 * exponentialDelay;
    
    return Math.min(exponentialDelay + jitter, config.maxDelay || 30000);
  }

  private isRetryableError(error: any): boolean {
    if (error instanceof SiYuanAPIError) {
      return error.retryable;
    }
    
    // Network errors are generally retryable
    return error.code === 'ECONNRESET' || 
           error.code === 'ENOTFOUND' || 
           error.code === 'TIMEOUT';
  }
}
```

---

## Data Synchronization and Caching

### 1. Event-Driven Cache Invalidation

Implement real-time cache invalidation based on SiYuan events.

```typescript
class EventDrivenCache {
  private cache = new Map<string, any>();
  private subscriptions = new Map<string, Set<string>>();

  constructor(private eventBus: EventBus) {
    this.setupEventHandlers();
  }

  private setupEventHandlers() {
    this.eventBus.on('block.updated', (event: BlockUpdatedEvent) => {
      this.invalidateBlockCache(event.blockId);
      this.invalidateParentCaches(event.blockId);
    });

    this.eventBus.on('document.created', (event: DocumentCreatedEvent) => {
      this.invalidateNotebookCache(event.notebookId);
      this.invalidateSearchCaches();
    });

    this.eventBus.on('notebook.closed', (event: NotebookClosedEvent) => {
      this.clearNotebookCaches(event.notebookId);
    });
  }

  async getCachedBlocks(notebookId: string): Promise<Block[]> {
    const cacheKey = `blocks:${notebookId}`;
    
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }

    const blocks = await this.fetchBlocksFromAPI(notebookId);
    this.cache.set(cacheKey, blocks);
    
    // Subscribe to related events
    this.subscribeToEvents(cacheKey, [
      `block.updated.${notebookId}`,
      `document.created.${notebookId}`,
      `notebook.closed.${notebookId}`
    ]);

    return blocks;
  }
}
```

### 2. Optimistic Updates

Implement optimistic UI updates with rollback capabilities.

```typescript
class OptimisticUpdateManager {
  private pendingUpdates = new Map<string, PendingUpdate>();
  private rollbackStack: RollbackOperation[] = [];

  async updateBlockOptimistically(
    blockId: string, 
    newContent: string
  ): Promise<void> {
    const originalContent = await this.getBlockContent(blockId);
    
    // Apply optimistic update immediately
    this.applyOptimisticUpdate(blockId, newContent);
    
    // Track the update for potential rollback
    const pendingUpdate: PendingUpdate = {
      id: blockId,
      originalValue: originalContent,
      newValue: newContent,
      timestamp: Date.now()
    };
    
    this.pendingUpdates.set(blockId, pendingUpdate);

    try {
      // Send actual API request
      await this.apiClient.updateBlock(blockId, newContent);
      
      // Success - confirm the update
      this.confirmUpdate(blockId);
    } catch (error) {
      // Failure - rollback the optimistic update
      await this.rollbackUpdate(blockId);
      throw error;
    }
  }

  private async rollbackUpdate(blockId: string): Promise<void> {
    const pendingUpdate = this.pendingUpdates.get(blockId);
    if (!pendingUpdate) return;

    // Revert to original content
    this.applyOptimisticUpdate(blockId, pendingUpdate.originalValue);
    this.pendingUpdates.delete(blockId);

    // Notify UI of rollback
    this.eventBus.emit('update.rolledback', {
      blockId,
      reason: 'api_failure'
    });
  }
}
```

---

## Batch Processing and Transactions

### 1. Transaction-like Operations

Implement atomic operations across multiple API calls.

```typescript
class TransactionManager {
  private activeTransactions = new Map<string, Transaction>();

  async executeTransaction<T>(
    operations: TransactionOperation[],
    options: TransactionOptions = {}
  ): Promise<T> {
    const transaction = new Transaction(operations, options);
    const transactionId = this.generateTransactionId();
    
    this.activeTransactions.set(transactionId, transaction);

    try {
      // Execute all operations
      const results = await transaction.execute();
      
      // All operations succeeded
      await transaction.commit();
      return results;
      
    } catch (error) {
      // Rollback all operations
      await transaction.rollback();
      throw new TransactionError(transactionId, error);
      
    } finally {
      this.activeTransactions.delete(transactionId);
    }
  }
}

class Transaction {
  private executedOperations: ExecutedOperation[] = [];
  private compensations: CompensationOperation[] = [];

  async execute(): Promise<any> {
    for (const operation of this.operations) {
      try {
        const result = await this.executeOperation(operation);
        
        this.executedOperations.push({
          operation,
          result,
          timestamp: Date.now()
        });
        
        // Prepare compensation for rollback
        const compensation = this.createCompensation(operation, result);
        this.compensations.unshift(compensation); // LIFO for rollback
        
      } catch (error) {
        // Partial failure - prepare for rollback
        throw new OperationError(operation, error);
      }
    }
    
    return this.executedOperations.map(op => op.result);
  }

  async rollback(): Promise<void> {
    for (const compensation of this.compensations) {
      try {
        await compensation.execute();
      } catch (error) {
        // Log compensation failure but continue rollback
        console.error('Compensation failed:', compensation, error);
      }
    }
  }
}

// Usage
const transactionManager = new TransactionManager();

await transactionManager.executeTransaction([
  {
    type: 'create_notebook',
    data: { name: 'New Project' }
  },
  {
    type: 'create_document',
    data: { notebook: '{{notebook_id}}', path: '/README' }
  },
  {
    type: 'insert_blocks',
    data: { 
      document: '{{document_id}}',
      blocks: [
        { type: 'h', content: '# Project Overview' },
        { type: 'p', content: 'This is a new project.' }
      ]
    }
  }
]);
```

### 2. Bulk Operations

Optimize bulk operations for large datasets.

```typescript
class BulkOperationProcessor {
  async processBulkBlocks(
    blocks: BlockData[],
    operation: BulkOperation,
    options: BulkOptions = {}
  ): Promise<BulkResult> {
    const config = { ...this.defaultBulkOptions, ...options };
    const chunks = this.chunkArray(blocks, config.batchSize);
    const results: BulkChunkResult[] = [];

    // Process chunks with concurrency control
    const semaphore = new Semaphore(config.concurrency);
    
    const chunkPromises = chunks.map(async (chunk, index) => {
      await semaphore.acquire();
      
      try {
        const chunkResult = await this.processBulkChunk(
          chunk, 
          operation, 
          index
        );
        results.push(chunkResult);
        
        // Progress reporting
        this.reportProgress(index + 1, chunks.length);
        
      } finally {
        semaphore.release();
      }
    });

    await Promise.all(chunkPromises);
    
    return this.consolidateResults(results);
  }

  private async processBulkChunk(
    chunk: BlockData[],
    operation: BulkOperation,
    chunkIndex: number
  ): Promise<BulkChunkResult> {
    const chunkResult: BulkChunkResult = {
      chunkIndex,
      processed: 0,
      succeeded: 0,
      failed: 0,
      errors: []
    };

    for (const block of chunk) {
      try {
        await this.processIndividualBlock(block, operation);
        chunkResult.succeeded++;
      } catch (error) {
        chunkResult.failed++;
        chunkResult.errors.push({
          blockId: block.id,
          error: error.message
        });
      }
      chunkResult.processed++;
    }

    return chunkResult;
  }
}
```

---

## Real-time Updates

### 1. WebSocket Integration

Implement real-time updates using WebSocket connections.

```typescript
class RealTimeUpdateManager {
  private websocket?: WebSocket;
  private subscriptions = new Map<string, Set<UpdateCallback>>();
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;

  async connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.websocket = new WebSocket('ws://localhost:6806/ws');
      
      this.websocket.onopen = () => {
        this.reconnectAttempts = 0;
        this.sendAuthenticationMessage();
        resolve();
      };

      this.websocket.onmessage = (event) => {
        this.handleMessage(JSON.parse(event.data));
      };

      this.websocket.onclose = () => {
        this.handleDisconnection();
      };

      this.websocket.onerror = (error) => {
        reject(error);
      };
    });
  }

  subscribe(eventType: string, callback: UpdateCallback): () => void {
    if (!this.subscriptions.has(eventType)) {
      this.subscriptions.set(eventType, new Set());
    }
    
    this.subscriptions.get(eventType)!.add(callback);
    
    // Send subscription message to server
    this.sendSubscriptionMessage(eventType);
    
    // Return unsubscribe function
    return () => {
      this.subscriptions.get(eventType)?.delete(callback);
    };
  }

  private handleMessage(message: WebSocketMessage) {
    const callbacks = this.subscriptions.get(message.type);
    if (!callbacks) return;

    for (const callback of callbacks) {
      try {
        callback(message.data);
      } catch (error) {
        console.error('Error in update callback:', error);
      }
    }
  }

  private async handleDisconnection() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      const delay = Math.pow(2, this.reconnectAttempts) * 1000;
      
      setTimeout(() => {
        this.connect().catch(console.error);
      }, delay);
    }
  }
}

// Usage
const updateManager = new RealTimeUpdateManager();
await updateManager.connect();

const unsubscribe = updateManager.subscribe('block.updated', (data) => {
  // Update UI in real-time
  updateBlockInUI(data.blockId, data.newContent);
});
```

### 2. Change Tracking

Track and merge changes from multiple sources.

```typescript
class ChangeTracker {
  private changes = new Map<string, ChangeRecord>();
  private conflictResolver: ConflictResolver;

  trackChange(change: Change): void {
    const existing = this.changes.get(change.entityId);
    
    if (!existing) {
      this.changes.set(change.entityId, {
        entityId: change.entityId,
        changes: [change],
        lastModified: change.timestamp
      });
      return;
    }

    // Check for conflicts
    const conflict = this.detectConflict(existing, change);
    if (conflict) {
      const resolution = this.conflictResolver.resolve(conflict);
      this.applyResolution(existing, change, resolution);
    } else {
      existing.changes.push(change);
      existing.lastModified = change.timestamp;
    }
  }

  private detectConflict(existing: ChangeRecord, newChange: Change): Conflict | null {
    const lastChange = existing.changes[existing.changes.length - 1];
    
    // Conflict if changes overlap and have different origins
    if (lastChange.field === newChange.field && 
        lastChange.origin !== newChange.origin &&
        Math.abs(lastChange.timestamp - newChange.timestamp) < 5000) {
      
      return new Conflict(existing, newChange, ConflictType.CONCURRENT_EDIT);
    }

    return null;
  }
}
```

---

## Security Best Practices

### 1. Token Management

Secure API token handling and rotation.

```typescript
class SecureTokenManager {
  private encryptedToken?: string;
  private tokenExpiry?: number;
  private refreshCallback?: () => Promise<string>;

  async setToken(token: string, expiryMs?: number): Promise<void> {
    // Encrypt token before storing
    this.encryptedToken = await this.encrypt(token);
    this.tokenExpiry = expiryMs ? Date.now() + expiryMs : undefined;
    
    // Schedule automatic refresh if expiry is set
    if (this.tokenExpiry && this.refreshCallback) {
      this.scheduleTokenRefresh();
    }
  }

  async getToken(): Promise<string> {
    if (!this.encryptedToken) {
      throw new Error('No token available');
    }

    // Check if token is expired
    if (this.tokenExpiry && Date.now() > this.tokenExpiry) {
      if (this.refreshCallback) {
        const newToken = await this.refreshCallback();
        await this.setToken(newToken);
      } else {
        throw new Error('Token expired and no refresh callback available');
      }
    }

    return await this.decrypt(this.encryptedToken);
  }

  private async encrypt(data: string): Promise<string> {
    const key = await this.getEncryptionKey();
    const iv = crypto.getRandomValues(new Uint8Array(16));
    
    const encoded = new TextEncoder().encode(data);
    const encrypted = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv },
      key,
      encoded
    );

    return this.arrayBufferToBase64(encrypted) + ':' + this.arrayBufferToBase64(iv);
  }

  private scheduleTokenRefresh(): void {
    const refreshTime = this.tokenExpiry! - 60000; // Refresh 1 minute before expiry
    const delay = Math.max(0, refreshTime - Date.now());
    
    setTimeout(async () => {
      try {
        if (this.refreshCallback) {
          const newToken = await this.refreshCallback();
          await this.setToken(newToken);
        }
      } catch (error) {
        console.error('Token refresh failed:', error);
      }
    }, delay);
  }
}
```

### 2. Request Validation

Validate and sanitize API requests.

```typescript
class RequestValidator {
  private schemas = new Map<string, JSONSchema>();

  constructor() {
    this.loadValidationSchemas();
  }

  validateRequest(endpoint: string, data: any): ValidationResult {
    const schema = this.schemas.get(endpoint);
    if (!schema) {
      return { valid: true, data: this.sanitizeGeneric(data) };
    }

    const validation = this.validateAgainstSchema(data, schema);
    if (!validation.valid) {
      return validation;
    }

    return {
      valid: true,
      data: this.sanitizeForEndpoint(endpoint, validation.data)
    };
  }

  private sanitizeForEndpoint(endpoint: string, data: any): any {
    switch (endpoint) {
      case '/block/insertBlock':
        return this.sanitizeBlockData(data);
      case '/query/sql':
        return this.sanitizeSQLQuery(data);
      default:
        return this.sanitizeGeneric(data);
    }
  }

  private sanitizeBlockData(data: any): any {
    return {
      ...data,
      // Remove potentially dangerous HTML
      data: this.sanitizeHTML(data.data),
      // Validate block type
      dataType: this.validateBlockType(data.dataType)
    };
  }

  private sanitizeSQLQuery(data: any): any {
    const stmt = data.stmt;
    
    // Basic SQL injection prevention
    if (this.containsUnsafeSQL(stmt)) {
      throw new ValidationError('Unsafe SQL detected');
    }

    // Limit query complexity
    if (this.isQueryTooComplex(stmt)) {
      throw new ValidationError('Query too complex');
    }

    return data;
  }
}
```

---

## Testing and Monitoring

### 1. API Testing Framework

Comprehensive testing for API operations.

```typescript
class APITestFramework {
  private testSuite: TestSuite;
  private mockAPI: MockSiYuanAPI;

  async runTests(): Promise<TestResults> {
    const results = new TestResults();
    
    for (const testCase of this.testSuite.getTestCases()) {
      const result = await this.runTestCase(testCase);
      results.add(result);
    }

    return results;
  }

  async runTestCase(testCase: TestCase): Promise<TestResult> {
    const startTime = Date.now();
    
    try {
      // Setup test environment
      await this.setupTestEnvironment(testCase);
      
      // Execute test
      const result = await testCase.execute(this.mockAPI);
      
      // Validate result
      const validation = await testCase.validate(result);
      
      return new TestResult(
        testCase.name,
        TestStatus.PASSED,
        Date.now() - startTime,
        validation
      );
      
    } catch (error) {
      return new TestResult(
        testCase.name,
        TestStatus.FAILED,
        Date.now() - startTime,
        { error: error.message }
      );
    } finally {
      await this.cleanupTestEnvironment(testCase);
    }
  }
}

// Example test case
class CreateBlockTestCase extends TestCase {
  async execute(api: MockSiYuanAPI): Promise<any> {
    return await api.call('/block/insertBlock', {
      dataType: 'markdown',
      data: 'Test block content',
      parentID: 'test-parent-id'
    });
  }

  async validate(result: any): Promise<ValidationResult> {
    return {
      valid: result.code === 0 && result.data.id,
      message: result.code === 0 ? 'Block created successfully' : 'Failed to create block'
    };
  }
}
```

### 2. Performance Monitoring

Monitor API performance and detect issues.

```typescript
class PerformanceMonitor {
  private metrics = new Map<string, MetricCollection>();
  private alertThresholds = new Map<string, AlertThreshold>();

  recordAPICall(endpoint: string, duration: number, success: boolean): void {
    const metric = this.getOrCreateMetric(endpoint);
    
    metric.addDataPoint({
      timestamp: Date.now(),
      duration,
      success,
      endpoint
    });

    // Check for performance alerts
    this.checkAlertThresholds(endpoint, metric);
  }

  private checkAlertThresholds(endpoint: string, metric: MetricCollection): void {
    const threshold = this.alertThresholds.get(endpoint);
    if (!threshold) return;

    const recentData = metric.getRecentDataPoints(threshold.timeWindow);
    const avgDuration = this.calculateAverageDuration(recentData);
    const errorRate = this.calculateErrorRate(recentData);

    if (avgDuration > threshold.maxAvgDuration) {
      this.triggerAlert(new PerformanceAlert(
        AlertType.SLOW_RESPONSE,
        endpoint,
        `Average response time ${avgDuration}ms exceeds threshold ${threshold.maxAvgDuration}ms`
      ));
    }

    if (errorRate > threshold.maxErrorRate) {
      this.triggerAlert(new PerformanceAlert(
        AlertType.HIGH_ERROR_RATE,
        endpoint,
        `Error rate ${errorRate}% exceeds threshold ${threshold.maxErrorRate}%`
      ));
    }
  }
}
```

---

## Scalability Considerations

### 1. Load Balancing

Distribute API calls across multiple SiYuan instances.

```typescript
class LoadBalancer {
  private instances: SiYuanInstance[] = [];
  private currentIndex = 0;
  private healthChecker: HealthChecker;

  constructor(instances: InstanceConfig[]) {
    this.instances = instances.map(config => new SiYuanInstance(config));
    this.healthChecker = new HealthChecker(this.instances);
    this.startHealthMonitoring();
  }

  async executeRequest(request: APIRequest): Promise<APIResponse> {
    const maxAttempts = this.instances.length;
    let lastError: Error;

    for (let attempt = 0; attempt < maxAttempts; attempt++) {
      const instance = this.selectInstance(request);
      
      try {
        return await instance.execute(request);
      } catch (error) {
        lastError = error;
        this.markInstanceUnhealthy(instance, error);
      }
    }

    throw new NoHealthyInstancesError(lastError);
  }

  private selectInstance(request: APIRequest): SiYuanInstance {
    const healthyInstances = this.instances.filter(i => i.isHealthy);
    
    if (healthyInstances.length === 0) {
      throw new NoHealthyInstancesError();
    }

    // Round-robin selection
    const instance = healthyInstances[this.currentIndex % healthyInstances.length];
    this.currentIndex++;
    
    return instance;
  }
}
```

### 2. Resource Management

Efficient resource usage for large-scale operations.

```typescript
class ResourceManager {
  private memoryPool: MemoryPool;
  private connectionPool: ConnectionPool;
  private taskQueue: PriorityQueue<Task>;

  async executeTask<T>(task: Task<T>): Promise<T> {
    // Check resource availability
    const resources = await this.allocateResources(task.requirements);
    
    try {
      // Execute with resource constraints
      return await this.executeWithResources(task, resources);
    } finally {
      // Always clean up resources
      await this.deallocateResources(resources);
    }
  }

  private async allocateResources(requirements: ResourceRequirements): Promise<AllocatedResources> {
    const memory = await this.memoryPool.allocate(requirements.memory);
    const connections = await this.connectionPool.allocate(requirements.connections);
    
    return new AllocatedResources(memory, connections);
  }
}
```

---

## Practical Applications

### 1. Intelligent Content Migration

Migrate content between notebooks with conflict resolution.

```typescript
class ContentMigrator {
  async migrateContent(
    sourceNotebook: string,
    targetNotebook: string,
    options: MigrationOptions = {}
  ): Promise<MigrationResult> {
    
    const migrationPlan = await this.createMigrationPlan(
      sourceNotebook, 
      targetNotebook, 
      options
    );
    
    const result = new MigrationResult();
    
    for (const item of migrationPlan.items) {
      try {
        const migratedItem = await this.migrateItem(item, options);
        result.addSuccess(migratedItem);
      } catch (error) {
        result.addError(item, error);
      }
    }
    
    return result;
  }

  private async migrateItem(
    item: MigrationItem,
    options: MigrationOptions
  ): Promise<MigratedItem> {
    
    // Check for conflicts in target
    const conflicts = await this.detectConflicts(item);
    
    if (conflicts.length > 0) {
      const resolution = await this.resolveConflicts(conflicts, options);
      item = this.applyResolution(item, resolution);
    }
    
    // Perform the actual migration
    return await this.performMigration(item);
  }
}
```

### 2. Advanced Search and Analytics

Implement sophisticated search and analytics capabilities.

```typescript
class AdvancedSearchEngine {
  async semanticSearch(query: string, options: SearchOptions = {}): Promise<SearchResults> {
    // Combine multiple search strategies
    const results = await Promise.all([
      this.fullTextSearch(query, options),
      this.structuralSearch(query, options),
      this.relationshipSearch(query, options)
    ]);
    
    // Merge and rank results
    return this.mergeSearchResults(results, query);
  }

  private async relationshipSearch(
    query: string,
    options: SearchOptions
  ): Promise<SearchResults> {
    
    // Find blocks with relationships to query terms
    const relationshipQuery = `
      WITH RECURSIVE related_blocks AS (
        -- Base case: blocks matching query directly
        SELECT id, content, 1 as depth
        FROM blocks 
        WHERE content MATCH ?
        
        UNION ALL
        
        -- Recursive case: blocks related to found blocks
        SELECT b.id, b.content, rb.depth + 1
        FROM blocks b
        JOIN refs r ON (b.id = r.block_id OR b.id = r.def_block_id)
        JOIN related_blocks rb ON (
          (r.def_block_id = rb.id OR r.block_id = rb.id) 
          AND rb.depth < ?
        )
      )
      SELECT DISTINCT id, content, MIN(depth) as relevance_depth
      FROM related_blocks
      GROUP BY id
      ORDER BY relevance_depth, id
    `;
    
    const results = await this.apiClient.query(relationshipQuery, [
      query,
      options.maxRelationshipDepth || 3
    ]);
    
    return new SearchResults(results, SearchType.RELATIONSHIP);
  }
}
```

---

*This guide provides advanced patterns for building sophisticated applications with the SiYuan API. These patterns help create more robust, performant, and maintainable integrations.*
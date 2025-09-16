# SiYuan プラグイン開発ベストプラクティス

このドキュメントでは、高品質で保守性の高いSiYuanプラグインを開発するためのベストプラクティスを紹介します。

## 目次

- [プロジェクト構成](#プロジェクト構成)
- [コーディング規約](#コーディング規約)
- [パフォーマンス最適化](#パフォーマンス最適化)
- [エラーハンドリング](#エラーハンドリング)
- [セキュリティ](#セキュリティ)
- [テストとデバッグ](#テストとデバッグ)
- [多言語対応](#多言語対応)
- [アクセシビリティ](#アクセシビリティ)
- [配布とメンテナンス](#配布とメンテナンス)

---

## プロジェクト構成

### 推奨ディレクトリ構造

```
my-plugin/
├── src/                    # ソースコード
│   ├── index.ts           # メインエントリーポイント
│   ├── components/        # UIコンポーネント
│   ├── services/          # APIサービス
│   ├── utils/             # ユーティリティ関数
│   ├── types/             # TypeScript型定義
│   └── styles/            # CSS/SCSS ファイル
├── i18n/                  # 多言語ファイル
│   ├── en_US.json
│   ├── ja_JP.json
│   └── zh_CN.json
├── assets/                # 静的ファイル
│   ├── icon.png
│   └── screenshots/
├── tests/                 # テストファイル
│   ├── unit/
│   └── integration/
├── docs/                  # ドキュメント
│   ├── README.md
│   ├── API.md
│   └── CHANGELOG.md
├── dist/                  # ビルド成果物（git除外）
├── plugin.json           # プラグイン設定
├── package.json          # Node.js設定
├── tsconfig.json         # TypeScript設定
├── webpack.config.js     # Webpack設定
├── .gitignore
└── README.md
```

### package.json の設定例

```json
{
  "name": "siyuan-plugin-example",
  "version": "1.0.0",
  "description": "Example SiYuan plugin with best practices",
  "main": "dist/index.js",
  "scripts": {
    "build": "webpack --mode production",
    "dev": "webpack --mode development --watch",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/**/*.ts",
    "lint:fix": "eslint src/**/*.ts --fix",
    "type-check": "tsc --noEmit",
    "format": "prettier --write src/**/*.{ts,js,json,css,md}",
    "prepare": "husky install"
  },
  "keywords": ["siyuan", "plugin", "productivity"],
  "author": "Your Name <your.email@example.com>",
  "license": "MIT",
  "devDependencies": {
    "@types/jest": "^29.5.0",
    "@typescript-eslint/eslint-plugin": "^5.57.0",
    "@typescript-eslint/parser": "^5.57.0",
    "eslint": "^8.37.0",
    "eslint-config-prettier": "^8.8.0",
    "eslint-plugin-prettier": "^4.2.1",
    "husky": "^8.0.3",
    "jest": "^29.5.0",
    "lint-staged": "^13.2.0",
    "prettier": "^2.8.7",
    "ts-jest": "^29.1.0",
    "ts-loader": "^9.4.2",
    "typescript": "^5.0.2",
    "webpack": "^5.76.3",
    "webpack-cli": "^5.0.1"
  },
  "lint-staged": {
    "*.{ts,js}": ["eslint --fix", "prettier --write"],
    "*.{json,md,css}": ["prettier --write"]
  }
}
```

---

## コーディング規約

### TypeScript の活用

```typescript
// 良い例: 型安全なプラグイン実装
import { Plugin, showMessage, Dialog } from "siyuan";

interface PluginConfig {
    apiEndpoint: string;
    maxRetries: number;
    timeout: number;
}

interface ApiResponse<T = unknown> {
    code: number;
    msg: string;
    data: T;
}

interface NotebookInfo {
    id: string;
    name: string;
    icon: string;
    sort: number;
    closed: boolean;
}

class ApiClient {
    private readonly baseUrl: string;
    private readonly token: string;
    private readonly timeout: number;

    constructor(baseUrl: string, token: string, timeout: number = 10000) {
        this.baseUrl = baseUrl;
        this.token = token;
        this.timeout = timeout;
    }

    async request<T>(endpoint: string, data: Record<string, unknown> = {}): Promise<ApiResponse<T>> {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.timeout);

        try {
            const response = await fetch(`${this.baseUrl}/api${endpoint}`, {
                method: "POST",
                headers: {
                    "Authorization": `Token ${this.token}`,
                    "Content-Type": "application/json"
                },
                body: JSON.stringify(data),
                signal: controller.signal
            });

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }

            return await response.json();
        } finally {
            clearTimeout(timeoutId);
        }
    }

    async getNotebooks(): Promise<NotebookInfo[]> {
        const response = await this.request<{ notebooks: NotebookInfo[] }>("/notebook/lsNotebooks");
        
        if (response.code !== 0) {
            throw new Error(`API Error: ${response.msg}`);
        }

        return response.data.notebooks;
    }
}

export default class ExamplePlugin extends Plugin {
    private config: PluginConfig;
    private apiClient: ApiClient;

    async onload(): Promise<void> {
        this.loadConfig();
        this.initializeApiClient();
        this.setupUI();
        this.registerCommands();
    }

    private loadConfig(): void {
        this.config = {
            apiEndpoint: this.data[STORAGE_NAME]?.apiEndpoint || "http://127.0.0.1:6806",
            maxRetries: this.data[STORAGE_NAME]?.maxRetries || 3,
            timeout: this.data[STORAGE_NAME]?.timeout || 10000
        };
    }

    private initializeApiClient(): void {
        const token = window.siyuan.config.api?.token || "";
        this.apiClient = new ApiClient(this.config.apiEndpoint, token, this.config.timeout);
    }

    // ... その他のメソッド
}
```

### エラーハンドリングのベストプラクティス

```typescript
// カスタムエラークラス
class PluginError extends Error {
    constructor(
        message: string,
        public readonly code: string,
        public readonly recoverable: boolean = true
    ) {
        super(message);
        this.name = "PluginError";
    }
}

class ApiError extends PluginError {
    constructor(message: string, public readonly statusCode: number) {
        super(message, "API_ERROR", statusCode < 500);
    }
}

// エラーハンドリングのラッパー関数
async function withErrorHandling<T>(
    operation: () => Promise<T>,
    fallback?: T,
    onError?: (error: Error) => void
): Promise<T | undefined> {
    try {
        return await operation();
    } catch (error) {
        console.error("Operation failed:", error);
        
        if (onError) {
            onError(error);
        }

        if (error instanceof PluginError) {
            if (error.recoverable) {
                showMessage(`操作が失敗しました: ${error.message}`, 3000, "error");
            } else {
                showMessage(`回復不可能なエラー: ${error.message}`, 5000, "error");
            }
        } else {
            showMessage(`予期しないエラーが発生しました: ${error.message}`, 3000, "error");
        }

        return fallback;
    }
}

// 使用例
class RobustPlugin extends Plugin {
    async performComplexOperation(): Promise<void> {
        await withErrorHandling(
            async () => {
                const notebooks = await this.apiClient.getNotebooks();
                await this.processNotebooks(notebooks);
            },
            undefined,
            (error) => {
                // カスタムエラー処理
                this.logError("ComplexOperation", error);
            }
        );
    }

    private logError(operation: string, error: Error): void {
        const errorInfo = {
            operation,
            error: error.message,
            stack: error.stack,
            timestamp: new Date().toISOString(),
            plugin: this.name,
            version: this.version
        };

        console.error("Plugin Error:", errorInfo);
        
        // 必要に応じて外部ログサービスに送信
        // this.sendErrorToService(errorInfo);
    }
}
```

---

## パフォーマンス最適化

### API呼び出しの最適化

```typescript
// 悪い例: 大量の個別API呼び出し
async function getBadBlockInfo(blockIds: string[]): Promise<BlockInfo[]> {
    const results: BlockInfo[] = [];
    
    for (const id of blockIds) {
        const response = await apiCall("/block/getBlockInfo", { id });
        results.push(response.data);
    }
    
    return results;
}

// 良い例: バッチ処理とキャッシュの活用
class OptimizedApiClient {
    private cache = new Map<string, { data: any; timestamp: number }>();
    private readonly cacheTimeout = 5 * 60 * 1000; // 5分

    async getBatchBlockInfo(blockIds: string[]): Promise<BlockInfo[]> {
        // キャッシュから取得可能なものを分離
        const now = Date.now();
        const cached: BlockInfo[] = [];
        const needsFetch: string[] = [];

        for (const id of blockIds) {
            const cacheEntry = this.cache.get(id);
            if (cacheEntry && (now - cacheEntry.timestamp) < this.cacheTimeout) {
                cached.push(cacheEntry.data);
            } else {
                needsFetch.push(id);
            }
        }

        // 必要なものだけをバッチ取得
        let fetched: BlockInfo[] = [];
        if (needsFetch.length > 0) {
            const chunks = this.chunkArray(needsFetch, 10); // 10個ずつ処理
            
            const fetchPromises = chunks.map(async (chunk) => {
                const response = await this.apiRequest("/block/getBlockInfos", { ids: chunk });
                return response.data;
            });

            const results = await Promise.all(fetchPromises);
            fetched = results.flat();

            // キャッシュに保存
            fetched.forEach(info => {
                this.cache.set(info.id, { data: info, timestamp: now });
            });
        }

        return [...cached, ...fetched];
    }

    private chunkArray<T>(array: T[], size: number): T[][] {
        const chunks: T[][] = [];
        for (let i = 0; i < array.length; i += size) {
            chunks.push(array.slice(i, i + size));
        }
        return chunks;
    }
}
```

### UI パフォーマンス最適化

```typescript
// 仮想スクロールの実装例
class VirtualScrollList {
    private container: HTMLElement;
    private itemHeight: number;
    private visibleItems: number;
    private data: any[];
    private renderItem: (item: any, index: number) => string;

    constructor(
        container: HTMLElement,
        itemHeight: number,
        renderItem: (item: any, index: number) => string
    ) {
        this.container = container;
        this.itemHeight = itemHeight;
        this.visibleItems = Math.ceil(container.clientHeight / itemHeight) + 2;
        this.renderItem = renderItem;
        this.data = [];

        this.setupScrollListener();
    }

    setData(data: any[]): void {
        this.data = data;
        this.render();
    }

    private setupScrollListener(): void {
        let scrollTimeout: number;
        
        this.container.addEventListener("scroll", () => {
            if (scrollTimeout) {
                cancelAnimationFrame(scrollTimeout);
            }
            
            scrollTimeout = requestAnimationFrame(() => {
                this.render();
            });
        });
    }

    private render(): void {
        const scrollTop = this.container.scrollTop;
        const startIndex = Math.floor(scrollTop / this.itemHeight);
        const endIndex = Math.min(startIndex + this.visibleItems, this.data.length);

        const visibleData = this.data.slice(startIndex, endIndex);
        const html = visibleData.map((item, index) => 
            this.renderItem(item, startIndex + index)
        ).join("");

        this.container.innerHTML = `
            <div style="height: ${this.data.length * this.itemHeight}px; position: relative;">
                <div style="transform: translateY(${startIndex * this.itemHeight}px);">
                    ${html}
                </div>
            </div>
        `;
    }
}

// 使用例
class PerformantListPlugin extends Plugin {
    private virtualList: VirtualScrollList;

    async onload(): Promise<void> {
        const listContainer = document.createElement("div");
        listContainer.style.height = "400px";
        listContainer.style.overflow = "auto";

        this.virtualList = new VirtualScrollList(
            listContainer,
            50, // 各アイテムの高さ
            (item, index) => `
                <div class="list-item" style="height: 50px;">
                    <h3>${item.title}</h3>
                    <p>${item.description}</p>
                </div>
            `
        );

        // 大量のデータを設定
        const data = await this.loadLargeDataset();
        this.virtualList.setData(data);
    }
}
```

---

## セキュリティ

### 入力検証とサニタイゼーション

```typescript
// セキュリティユーティリティ
class SecurityUtils {
    // HTMLエスケープ
    static escapeHtml(text: string): string {
        const div = document.createElement("div");
        div.textContent = text;
        return div.innerHTML;
    }

    // SQLインジェクション対策
    static sanitizeSqlInput(input: string): string {
        return input.replace(/['";\\]/g, "\\$&");
    }

    // パス・トラバーサル対策
    static sanitizePath(path: string): string {
        return path.replace(/\.\./g, "").replace(/[<>:"|?*]/g, "");
    }

    // URL検証
    static isValidUrl(url: string): boolean {
        try {
            const urlObj = new URL(url);
            return ["http:", "https:"].includes(urlObj.protocol);
        } catch {
            return false;
        }
    }

    // XSS対策のためのCSP設定
    static setupContentSecurityPolicy(): void {
        const meta = document.createElement("meta");
        meta.httpEquiv = "Content-Security-Policy";
        meta.content = "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';";
        document.head.appendChild(meta);
    }
}

// 安全な入力フォームコンポーネント
class SecureForm {
    private form: HTMLFormElement;
    private validators: Map<string, (value: string) => boolean>;

    constructor(container: HTMLElement) {
        this.validators = new Map();
        this.createForm(container);
    }

    addValidator(fieldName: string, validator: (value: string) => boolean): void {
        this.validators.set(fieldName, validator);
    }

    private createForm(container: HTMLElement): void {
        this.form = document.createElement("form");
        this.form.addEventListener("submit", (e) => {
            e.preventDefault();
            this.handleSubmit();
        });

        container.appendChild(this.form);
    }

    private handleSubmit(): void {
        const formData = new FormData(this.form);
        const errors: string[] = [];

        for (const [name, value] of formData.entries()) {
            const validator = this.validators.get(name);
            const stringValue = value.toString();

            // 基本的なサニタイゼーション
            const sanitized = SecurityUtils.escapeHtml(stringValue);

            // カスタム検証
            if (validator && !validator(sanitized)) {
                errors.push(`Invalid input for field: ${name}`);
            }
        }

        if (errors.length > 0) {
            showMessage(`Validation errors: ${errors.join(", ")}`, 5000, "error");
            return;
        }

        this.processValidData(formData);
    }

    private processValidData(formData: FormData): void {
        // 検証済みデータの処理
    }
}
```

### APIキーとシークレットの管理

```typescript
// 安全な設定管理
class SecureConfigManager {
    private readonly encryptionKey: string;

    constructor() {
        // 実際の実装では、より安全な方法でキーを取得
        this.encryptionKey = this.generateEncryptionKey();
    }

    // 機密情報の暗号化保存
    async saveSecureConfig(key: string, value: string): Promise<void> {
        const encrypted = await this.encrypt(value);
        
        // SiYuanのストレージに暗号化された値を保存
        const currentData = this.data[STORAGE_NAME] || {};
        currentData[`secure_${key}`] = encrypted;
        this.saveData(STORAGE_NAME, currentData);
    }

    // 機密情報の復号化読み込み
    async loadSecureConfig(key: string): Promise<string | null> {
        const currentData = this.data[STORAGE_NAME] || {};
        const encrypted = currentData[`secure_${key}`];
        
        if (!encrypted) {
            return null;
        }

        return await this.decrypt(encrypted);
    }

    private async encrypt(text: string): Promise<string> {
        // Web Crypto APIを使用した暗号化
        const encoder = new TextEncoder();
        const data = encoder.encode(text);
        
        const key = await window.crypto.subtle.importKey(
            "raw",
            encoder.encode(this.encryptionKey.slice(0, 32)),
            { name: "AES-GCM" },
            false,
            ["encrypt"]
        );

        const iv = window.crypto.getRandomValues(new Uint8Array(12));
        const encrypted = await window.crypto.subtle.encrypt(
            { name: "AES-GCM", iv },
            key,
            data
        );

        // IVと暗号化データを結合してBase64エンコード
        const combined = new Uint8Array(iv.length + encrypted.byteLength);
        combined.set(iv);
        combined.set(new Uint8Array(encrypted), iv.length);
        
        return btoa(String.fromCharCode(...combined));
    }

    private async decrypt(encryptedData: string): Promise<string> {
        const combined = new Uint8Array(
            atob(encryptedData).split("").map(char => char.charCodeAt(0))
        );
        
        const iv = combined.slice(0, 12);
        const encrypted = combined.slice(12);

        const encoder = new TextEncoder();
        const key = await window.crypto.subtle.importKey(
            "raw",
            encoder.encode(this.encryptionKey.slice(0, 32)),
            { name: "AES-GCM" },
            false,
            ["decrypt"]
        );

        const decrypted = await window.crypto.subtle.decrypt(
            { name: "AES-GCM", iv },
            key,
            encrypted
        );

        return new TextDecoder().decode(decrypted);
    }

    private generateEncryptionKey(): string {
        // プロダクションでは、より安全な方法でキーを生成・管理
        return window.siyuan.config.system?.id || "default-key";
    }

    // 機密情報の安全な削除
    async clearSecureConfig(key: string): Promise<void> {
        const currentData = this.data[STORAGE_NAME] || {};
        delete currentData[`secure_${key}`];
        this.saveData(STORAGE_NAME, currentData);
    }
}

// 使用例
class SecurePlugin extends Plugin {
    private configManager: SecureConfigManager;

    async onload(): Promise<void> {
        this.configManager = new SecureConfigManager();

        // APIキーの設定UI
        this.setting.addItem({
            key: "apiKey",
            value: "",
            type: "textinput",
            title: "API Key",
            description: "Enter your API key (will be encrypted)",
            action: {
                callback: async () => {
                    const apiKey = this.setting.get("apiKey");
                    if (apiKey) {
                        await this.configManager.saveSecureConfig("apiKey", apiKey);
                        showMessage("API key saved securely", 2000, "info");
                    }
                }
            }
        });
    }

    async getApiKey(): Promise<string | null> {
        return await this.configManager.loadSecureConfig("apiKey");
    }
}
```

---

## テストとデバッグ

### ユニットテストの実装

```typescript
// tests/unit/api-client.test.ts
import { ApiClient } from "../../src/services/api-client";

// モッククラス
class MockFetch {
    private responses: Map<string, any> = new Map();

    mockResponse(url: string, response: any): void {
        this.responses.set(url, response);
    }

    async fetch(url: string, options: RequestInit): Promise<Response> {
        const response = this.responses.get(url);
        
        if (!response) {
            throw new Error(`No mock response for ${url}`);
        }

        return {
            ok: response.ok !== false,
            status: response.status || 200,
            statusText: response.statusText || "OK",
            json: async () => response.data || response
        } as Response;
    }
}

describe("ApiClient", () => {
    let apiClient: ApiClient;
    let mockFetch: MockFetch;

    beforeEach(() => {
        mockFetch = new MockFetch();
        global.fetch = mockFetch.fetch.bind(mockFetch) as any;
        
        apiClient = new ApiClient("http://localhost:6806", "test-token");
    });

    describe("getNotebooks", () => {
        it("should return notebooks when API call succeeds", async () => {
            const mockNotebooks = [
                { id: "1", name: "Test Notebook", icon: "📓", sort: 1, closed: false }
            ];

            mockFetch.mockResponse("http://localhost:6806/api/notebook/lsNotebooks", {
                code: 0,
                msg: "",
                data: { notebooks: mockNotebooks }
            });

            const result = await apiClient.getNotebooks();
            
            expect(result).toEqual(mockNotebooks);
        });

        it("should throw error when API call fails", async () => {
            mockFetch.mockResponse("http://localhost:6806/api/notebook/lsNotebooks", {
                code: -1,
                msg: "API Error",
                data: null
            });

            await expect(apiClient.getNotebooks()).rejects.toThrow("API Error");
        });
    });

    describe("request timeout", () => {
        it("should timeout after specified duration", async () => {
            const slowClient = new ApiClient("http://localhost:6806", "test-token", 100);
            
            // 遅いレスポンスをモック
            mockFetch.mockResponse = () => {
                return new Promise(resolve => setTimeout(resolve, 200));
            };

            await expect(slowClient.getNotebooks()).rejects.toThrow();
        });
    });
});
```

### 統合テストの実装

```typescript
// tests/integration/plugin.test.ts
import { JSDOM } from "jsdom";
import ExamplePlugin from "../../src/index";

describe("ExamplePlugin Integration", () => {
    let plugin: ExamplePlugin;
    let mockSiYuan: any;

    beforeEach(() => {
        // DOM環境のセットアップ
        const dom = new JSDOM(`<!DOCTYPE html><html><body></body></html>`);
        global.document = dom.window.document;
        global.window = dom.window as any;

        // SiYuan環境のモック
        mockSiYuan = {
            config: {
                api: { token: "test-token" },
                system: { id: "test-system" }
            }
        };
        
        global.window.siyuan = mockSiYuan;

        // Plugin インスタンスの作成
        plugin = new ExamplePlugin({
            app: {} as any,
            name: "test-plugin",
            displayName: "Test Plugin",
            i18n: {}
        });
    });

    afterEach(async () => {
        if (plugin.onunload) {
            await plugin.onunload();
        }
    });

    it("should initialize correctly", async () => {
        await plugin.onload();
        
        // 初期化の検証
        expect(plugin.name).toBe("test-plugin");
        // UIが正しく作成されているかチェック
        // API clientが初期化されているかチェック
    });

    it("should handle API errors gracefully", async () => {
        // APIエラーをシミュレート
        global.fetch = jest.fn().mockRejectedValue(new Error("Network error"));
        
        await plugin.onload();
        
        // エラーハンドリングが適切に行われているかチェック
        // ユーザーへの適切なメッセージ表示をチェック
    });
});
```

### デバッグツールの実装

```typescript
// デバッグ用のユーティリティクラス
class DebugUtils {
    private static readonly DEBUG_KEY = "plugin-debug";
    private static isEnabled = false;

    static enable(): void {
        this.isEnabled = true;
        localStorage.setItem(this.DEBUG_KEY, "true");
        console.log("Plugin debug mode enabled");
    }

    static disable(): void {
        this.isEnabled = false;
        localStorage.removeItem(this.DEBUG_KEY);
        console.log("Plugin debug mode disabled");
    }

    static isDebugEnabled(): boolean {
        if (this.isEnabled) return true;
        return localStorage.getItem(this.DEBUG_KEY) === "true";
    }

    static log(component: string, message: string, data?: any): void {
        if (!this.isDebugEnabled()) return;

        const timestamp = new Date().toISOString();
        const prefix = `[DEBUG ${timestamp}] ${component}:`;
        
        if (data) {
            console.log(prefix, message, data);
        } else {
            console.log(prefix, message);
        }
    }

    static measurePerformance<T>(
        operation: string, 
        fn: () => T | Promise<T>
    ): T | Promise<T> {
        if (!this.isDebugEnabled()) {
            return fn();
        }

        const start = performance.now();
        const result = fn();

        if (result instanceof Promise) {
            return result.finally(() => {
                const end = performance.now();
                this.log("PERF", `${operation} took ${(end - start).toFixed(2)}ms`);
            });
        } else {
            const end = performance.now();
            this.log("PERF", `${operation} took ${(end - start).toFixed(2)}ms`);
            return result;
        }
    }

    static createDebugPanel(): HTMLElement {
        const panel = document.createElement("div");
        panel.style.cssText = `
            position: fixed;
            top: 10px;
            right: 10px;
            width: 300px;
            background: #1e1e1e;
            color: #fff;
            padding: 10px;
            border-radius: 5px;
            font-family: monospace;
            font-size: 12px;
            z-index: 10000;
            max-height: 400px;
            overflow-y: auto;
        `;

        panel.innerHTML = `
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px;">
                <strong>Debug Panel</strong>
                <button id="clearDebugLog" style="background: #333; color: #fff; border: none; padding: 2px 6px; border-radius: 3px;">Clear</button>
            </div>
            <div id="debugLog"></div>
        `;

        const clearButton = panel.querySelector("#clearDebugLog");
        const debugLog = panel.querySelector("#debugLog");

        clearButton?.addEventListener("click", () => {
            if (debugLog) debugLog.innerHTML = "";
        });

        // console.log をインターセプトしてデバッグパネルに表示
        const originalLog = console.log;
        console.log = (...args) => {
            originalLog.apply(console, args);
            
            if (this.isDebugEnabled() && debugLog && args[0]?.includes?.("[DEBUG")) {
                const logEntry = document.createElement("div");
                logEntry.style.cssText = "margin-bottom: 5px; padding: 2px; background: #333; border-radius: 2px;";
                logEntry.textContent = args.join(" ");
                debugLog.appendChild(logEntry);
                debugLog.scrollTop = debugLog.scrollHeight;
            }
        };

        return panel;
    }
}

// プラグインでのデバッグ機能の使用例
export default class DebuggablePlugin extends Plugin {
    private debugPanel?: HTMLElement;

    async onload(): Promise<void> {
        // デバッグモードの場合はデバッグパネルを表示
        if (DebugUtils.isDebugEnabled()) {
            this.debugPanel = DebugUtils.createDebugPanel();
            document.body.appendChild(this.debugPanel);
        }

        // デバッグログの出力
        DebugUtils.log("Plugin", "Loading plugin", { name: this.name });

        // パフォーマンス測定付きの初期化
        await DebugUtils.measurePerformance("Plugin initialization", async () => {
            await this.initializeComponents();
        });

        // デバッグコマンドを追加
        this.addCommand({
            langKey: "toggleDebug",
            hotkey: "⌘⌥⇧D",
            callback: () => {
                if (DebugUtils.isDebugEnabled()) {
                    DebugUtils.disable();
                    this.debugPanel?.remove();
                    this.debugPanel = undefined;
                } else {
                    DebugUtils.enable();
                    this.debugPanel = DebugUtils.createDebugPanel();
                    document.body.appendChild(this.debugPanel);
                }
            }
        });

        DebugUtils.log("Plugin", "Plugin loaded successfully");
    }

    async onunload(): Promise<void> {
        DebugUtils.log("Plugin", "Unloading plugin");
        
        this.debugPanel?.remove();
        
        DebugUtils.log("Plugin", "Plugin unloaded successfully");
    }

    private async initializeComponents(): Promise<void> {
        DebugUtils.log("Plugin", "Initializing components");
        
        // 各コンポーネントの初期化をパフォーマンス測定
        await DebugUtils.measurePerformance("API Client init", () => {
            // API client の初期化
        });

        await DebugUtils.measurePerformance("UI Components init", () => {
            // UI コンポーネントの初期化
        });

        DebugUtils.log("Plugin", "Components initialized");
    }
}
```

---

## 多言語対応

### 多言語ファイルの構造

```json
// i18n/en_US.json
{
  "plugin": {
    "name": "Example Plugin",
    "description": "An example plugin demonstrating best practices"
  },
  "ui": {
    "dialog": {
      "title": "Example Dialog",
      "confirm": "Confirm",
      "cancel": "Cancel"
    },
    "menu": {
      "refresh": "Refresh",
      "settings": "Settings",
      "about": "About"
    },
    "notifications": {
      "success": "Operation completed successfully",
      "error": "An error occurred: {0}",
      "warning": "Warning: {0}"
    }
  },
  "settings": {
    "general": {
      "title": "General Settings",
      "apiEndpoint": "API Endpoint",
      "timeout": "Request Timeout (ms)",
      "retries": "Max Retries"
    },
    "advanced": {
      "title": "Advanced Settings",
      "debugMode": "Enable Debug Mode",
      "logLevel": "Log Level"
    }
  },
  "errors": {
    "networkError": "Network connection failed",
    "authError": "Authentication failed",
    "validationError": "Invalid input: {0}"
  }
}
```

```json
// i18n/ja_JP.json
{
  "plugin": {
    "name": "サンプルプラグイン",
    "description": "ベストプラクティスを示すサンプルプラグイン"
  },
  "ui": {
    "dialog": {
      "title": "サンプルダイアログ",
      "confirm": "確認",
      "cancel": "キャンセル"
    },
    "menu": {
      "refresh": "更新",
      "settings": "設定",
      "about": "について"
    },
    "notifications": {
      "success": "操作が正常に完了しました",
      "error": "エラーが発生しました: {0}",
      "warning": "警告: {0}"
    }
  },
  "settings": {
    "general": {
      "title": "一般設定",
      "apiEndpoint": "APIエンドポイント",
      "timeout": "リクエストタイムアウト (ms)",
      "retries": "最大再試行回数"
    },
    "advanced": {
      "title": "高度な設定",
      "debugMode": "デバッグモードを有効にする",
      "logLevel": "ログレベル"
    }
  },
  "errors": {
    "networkError": "ネットワーク接続に失敗しました",
    "authError": "認証に失敗しました",
    "validationError": "入力が無効です: {0}"
  }
}
```

### 多言語対応のユーティリティ

```typescript
// i18n utility class
class I18n {
    private messages: Record<string, any> = {};
    private currentLocale: string = "en_US";

    constructor(messages: Record<string, any>, locale?: string) {
        this.messages = messages;
        if (locale) {
            this.currentLocale = locale;
        }
    }

    // キー指定でメッセージを取得
    t(key: string, ...args: any[]): string {
        const keys = key.split(".");
        let message: any = this.messages;

        for (const k of keys) {
            if (message && typeof message === "object" && k in message) {
                message = message[k];
            } else {
                console.warn(`Missing translation key: ${key}`);
                return key; // キーが見つからない場合はキー自体を返す
            }
        }

        if (typeof message !== "string") {
            console.warn(`Translation key ${key} is not a string`);
            return key;
        }

        // プレースホルダーの置換
        return this.formatMessage(message, args);
    }

    // メッセージフォーマット（プレースホルダー {0}, {1}, ... を置換）
    private formatMessage(message: string, args: any[]): string {
        return message.replace(/\{(\d+)\}/g, (match, index) => {
            const argIndex = parseInt(index, 10);
            return args[argIndex] !== undefined ? String(args[argIndex]) : match;
        });
    }

    // 複数形対応
    plural(key: string, count: number, ...args: any[]): string {
        const pluralKey = count === 1 ? `${key}.singular` : `${key}.plural`;
        return this.t(pluralKey, count, ...args);
    }

    // 日付のローカライズ
    formatDate(date: Date, format: "short" | "long" = "short"): string {
        const options: Intl.DateTimeFormatOptions = format === "short" 
            ? { year: "numeric", month: "2-digit", day: "2-digit" }
            : { year: "numeric", month: "long", day: "numeric", weekday: "long" };

        return new Intl.DateTimeFormat(this.currentLocale.replace("_", "-"), options).format(date);
    }

    // 数値のローカライズ
    formatNumber(number: number, options?: Intl.NumberFormatOptions): string {
        return new Intl.NumberFormat(this.currentLocale.replace("_", "-"), options).format(number);
    }

    // 通貨のローカライズ
    formatCurrency(amount: number, currency: string): string {
        return new Intl.NumberFormat(this.currentLocale.replace("_", "-"), {
            style: "currency",
            currency
        }).format(amount);
    }
}

// プラグインでの多言語対応実装
export default class I18nPlugin extends Plugin {
    private i18n: I18n;

    async onload(): Promise<void> {
        // 現在のロケールを取得
        const locale = window.siyuan.config.lang || "en_US";
        
        // 多言語メッセージを初期化
        this.i18n = new I18n(this.i18nMessages, locale);

        // UIを構築
        this.createLocalizedUI();

        // 設定項目を追加
        this.addLocalizedSettings();
    }

    private createLocalizedUI(): void {
        this.addTopBar({
            icon: "iconLanguage",
            title: this.i18n.t("plugin.name"),
            callback: () => this.showLocalizedDialog()
        });
    }

    private showLocalizedDialog(): void {
        const dialog = new Dialog({
            title: this.i18n.t("ui.dialog.title"),
            content: `
                <div class="localized-dialog">
                    <p>${this.i18n.t("plugin.description")}</p>
                    <div class="dialog-actions">
                        <button class="b3-button b3-button--cancel" id="cancelBtn">
                            ${this.i18n.t("ui.dialog.cancel")}
                        </button>
                        <button class="b3-button" id="confirmBtn">
                            ${this.i18n.t("ui.dialog.confirm")}
                        </button>
                    </div>
                </div>
            `,
            width: 400,
            height: 200
        });

        // イベントハンドラー
        dialog.element.querySelector("#confirmBtn")?.addEventListener("click", () => {
            showMessage(this.i18n.t("ui.notifications.success"), 2000, "info");
            dialog.destroy();
        });

        dialog.element.querySelector("#cancelBtn")?.addEventListener("click", () => {
            dialog.destroy();
        });
    }

    private addLocalizedSettings(): void {
        this.setting.addItem({
            key: "generalSettings",
            value: "",
            type: "hint",
            title: this.i18n.t("settings.general.title"),
            description: ""
        });

        this.setting.addItem({
            key: "apiEndpoint",
            value: "http://127.0.0.1:6806",
            type: "textinput",
            title: this.i18n.t("settings.general.apiEndpoint"),
            description: this.i18n.t("settings.general.apiEndpoint")
        });

        this.setting.addItem({
            key: "timeout",
            value: 10000,
            type: "number",
            title: this.i18n.t("settings.general.timeout"),
            description: this.i18n.t("settings.general.timeout")
        });
    }

    // エラーメッセージの多言語対応
    private handleError(error: Error, context?: string): void {
        let message: string;

        if (error.name === "NetworkError") {
            message = this.i18n.t("errors.networkError");
        } else if (error.name === "AuthenticationError") {
            message = this.i18n.t("errors.authError");
        } else {
            message = this.i18n.t("errors.validationError", error.message);
        }

        showMessage(message, 3000, "error");
        
        // デバッグ情報をログに出力
        console.error(`Plugin error in ${context}:`, error);
    }

    // 動的な翻訳（設定変更時などに対応）
    private updateLocale(newLocale: string): void {
        this.i18n = new I18n(this.i18nMessages, newLocale);
        
        // UIを再構築
        this.recreateUI();
        
        showMessage(this.i18n.t("ui.notifications.success"), 2000, "info");
    }

    private recreateUI(): void {
        // 既存のUIを削除
        this.removeExistingUI();
        
        // 新しいロケールでUIを再作成
        this.createLocalizedUI();
    }

    private removeExistingUI(): void {
        // プラグインが作成したUI要素をクリーンアップ
        // 実装は具体的なUI構造に依存
    }
}
```

これらのベストプラクティスに従うことで、保守性が高く、セキュアで、ユーザーフレンドリーなSiYuanプラグインを開発できます。

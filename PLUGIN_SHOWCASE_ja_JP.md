# SiYuan プラグイン開発ショーケース

このドキュメントは、SiYuan APIを使用した実践的なプラグイン開発例を提供します。各言語での実装例とUI・パネル操作による多様なプラグインの可能性を示します。

## 目次

- [基本的なプラグイン構造](#基本的なプラグイン構造)
- [言語別実装例](#言語別実装例)
  - [JavaScript/TypeScript](#javascripttypescript)
  - [Python](#python)
  - [Go](#go)
- [UI操作プラグイン例](#ui操作プラグイン例)
  - [パネル管理プラグイン](#パネル管理プラグイン)
  - [ダイアログ表示プラグイン](#ダイアログ表示プラグイン)
  - [メニュー拡張プラグイン](#メニュー拡張プラグイン)
- [データ統合プラグイン例](#データ統合プラグイン例)
  - [データベース操作プラグイン](#データベース操作プラグイン)
  - [検索拡張プラグイン](#検索拡張プラグイン)
  - [同期連携プラグイン](#同期連携プラグイン)
- [高度なプラグインパターン](#高度なプラグインパターン)
  - [AI統合プラグイン](#ai統合プラグイン)
  - [外部サービス連携](#外部サービス連携)
  - [ワークフロー自動化](#ワークフロー自動化)
- [ベストプラクティス](#ベストプラクティス)

---

## 基本的なプラグイン構造

すべてのSiYuanプラグインは以下の基本構造を持ちます：

```
plugin-name/
├── plugin.json          # プラグイン設定
├── index.js             # メインエントリーポイント
├── i18n/               # 多言語対応
│   ├── en_US.json
│   ├── ja_JP.json
│   └── zh_CN.json
├── README.md           # ドキュメント
└── icon.png           # プラグインアイコン
```

### plugin.json の例

```json
{
  "name": "sample-plugin",
  "author": "YourName",
  "url": "https://github.com/yourusername/sample-plugin",
  "version": "1.0.0",
  "minAppVersion": "2.8.0",
  "backends": ["windows", "linux", "darwin"],
  "frontends": ["desktop", "mobile"],
  "displayName": {
    "en_US": "Sample Plugin",
    "ja_JP": "サンプルプラグイン",
    "zh_CN": "示例插件"
  },
  "description": {
    "en_US": "A sample plugin demonstrating SiYuan API usage",
    "ja_JP": "SiYuan APIの使用方法を示すサンプルプラグイン",
    "zh_CN": "演示SiYuan API使用的示例插件"
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
    "custom": ""
  }
}
```

---

## 言語別実装例

### JavaScript/TypeScript

#### 基本的なJavaScriptプラグイン

```javascript
import {
    Plugin,
    showMessage,
    confirm,
    Dialog,
    Menu,
    openTab,
    adaptHotkey
} from "siyuan";

const API_BASE = "/api";
const API_TOKEN = window.siyuan.config.api?.token || "";

export default class SamplePlugin extends Plugin {
    
    async onload() {
        // アイコンをトップバーに追加
        const topBarElement = this.addTopBar({
            icon: "iconEmoji",
            title: this.i18n.pluginTitle,
            position: "right",
            callback: () => {
                this.showMainDialog();
            }
        });

        // コマンドを追加
        this.addCommand({
            langKey: "showDialog",
            hotkey: "⌘⌥D",
            callback: () => {
                this.showMainDialog();
            }
        });

        // 設定項目を追加
        this.setting.addItem({
            key: "apiEndpoint",
            value: "https://api.example.com",
            type: "textinput",
            title: this.i18n.setting.apiEndpoint,
            description: this.i18n.setting.apiEndpointDesc,
        });

        console.log("Sample Plugin loaded");
    }

    async onunload() {
        console.log("Sample Plugin unloaded");
    }

    private async showMainDialog() {
        const dialog = new Dialog({
            title: this.i18n.dialogTitle,
            content: `<div class="b3-dialog__content">
                <div class="fn__flex-column">
                    <label class="fn__flex">
                        <span class="fn__flex-center fn__size200">${this.i18n.inputLabel}</span>
                        <input class="b3-text-field fn__flex-1" id="pluginInput" value="">
                    </label>
                    <div class="fn__hr"></div>
                    <div class="fn__flex fn__flex-wrap" id="pluginButtons">
                        <button class="b3-button b3-button--outline fn__flex-center fn__size200" id="searchButton">
                            ${this.i18n.searchButton}
                        </button>
                        <button class="b3-button b3-button--outline fn__flex-center fn__size200" id="createButton">
                            ${this.i18n.createButton}
                        </button>
                    </div>
                </div>
            </div>`,
            width: 520,
            height: 300,
        });

        // ボタンイベントハンドラー
        dialog.element.querySelector("#searchButton").addEventListener("click", () => {
            this.handleSearch();
        });

        dialog.element.querySelector("#createButton").addEventListener("click", () => {
            this.handleCreate();
        });
    }

    private async handleSearch() {
        const query = (document.getElementById("pluginInput") as HTMLInputElement).value;
        
        try {
            const response = await this.apiRequest("/search/fullTextSearchBlock", {
                query: query,
                types: {
                    paragraph: true,
                    heading: true,
                    document: true
                }
            });

            if (response.code === 0) {
                this.displaySearchResults(response.data.blocks);
            } else {
                showMessage(this.i18n.searchError + response.msg, 3000, "error");
            }
        } catch (error) {
            console.error("Search error:", error);
            showMessage(this.i18n.searchError + error.message, 3000, "error");
        }
    }

    private async handleCreate() {
        const content = (document.getElementById("pluginInput") as HTMLInputElement).value;
        
        try {
            // 新しいドキュメントを作成
            const createResponse = await this.apiRequest("/filetree/createDocWithMd", {
                notebook: this.getCurrentNotebook(),
                path: `/新しいドキュメント-${Date.now()}`,
                markdown: `# 新しいドキュメント\n\n${content}`
            });

            if (createResponse.code === 0) {
                showMessage(this.i18n.createSuccess, 3000, "info");
                
                // 作成したドキュメントを開く
                openTab({
                    app: this.app,
                    doc: {
                        id: createResponse.data,
                        action: ["cb-get-focus"]
                    }
                });
            } else {
                showMessage(this.i18n.createError + createResponse.msg, 3000, "error");
            }
        } catch (error) {
            console.error("Create error:", error);
            showMessage(this.i18n.createError + error.message, 3000, "error");
        }
    }

    private async apiRequest(endpoint: string, data: any) {
        const response = await fetch(API_BASE + endpoint, {
            method: "POST",
            headers: {
                "Authorization": `Token ${API_TOKEN}`,
                "Content-Type": "application/json"
            },
            body: JSON.stringify(data)
        });

        return response.json();
    }

    private getCurrentNotebook(): string {
        // 現在のノートブックIDを取得
        const activeTab = document.querySelector('.layout__tab--active');
        return activeTab?.getAttribute('data-id') || '';
    }

    private displaySearchResults(blocks: any[]) {
        const resultsHtml = blocks.map(block => `
            <div class="search-result" data-id="${block.id}">
                <div class="search-result__title">${block.name || block.content.slice(0, 50)}</div>
                <div class="search-result__path">${block.hPath}</div>
            </div>
        `).join('');

        // 結果を表示するダイアログを作成
        new Dialog({
            title: this.i18n.searchResults,
            content: `<div class="search-results-container">${resultsHtml}</div>`,
            width: 600,
            height: 400,
        });
    }
}
```

#### TypeScriptによる型安全なプラグイン

```typescript
import {
    Plugin,
    showMessage,
    Dialog,
    IOperation,
    IMenuItem
} from "siyuan";

interface ApiResponse<T = any> {
    code: number;
    msg: string;
    data: T;
}

interface BlockInfo {
    id: string;
    parentID: string;
    rootID: string;
    content: string;
    type: string;
    subType: string;
}

interface NotebookInfo {
    id: string;
    name: string;
    icon: string;
    sort: number;
    closed: boolean;
}

export default class TypedPlugin extends Plugin {
    private apiToken: string;
    private notebooks: NotebookInfo[] = [];

    async onload(): Promise<void> {
        this.apiToken = window.siyuan.config.api?.token || "";
        
        // ノートブック一覧を読み込み
        await this.loadNotebooks();

        // カスタムドックパネルを追加
        this.addDock({
            config: {
                position: "LeftBottom",
                size: {width: 300, height: 200},
                icon: "iconFiles",
                title: this.i18n.dockTitle,
            },
            data: {},
            type: "notebook-manager",
            init: (dock) => {
                this.initNotebookManager(dock.element);
            }
        });
    }

    private async loadNotebooks(): Promise<void> {
        try {
            const response = await this.apiCall<NotebookInfo[]>("/notebook/lsNotebooks", {});
            if (response.code === 0) {
                this.notebooks = response.data.notebooks || [];
            }
        } catch (error) {
            console.error("Failed to load notebooks:", error);
        }
    }

    private async apiCall<T>(endpoint: string, data: Record<string, any>): Promise<ApiResponse<T>> {
        const response = await fetch(`/api${endpoint}`, {
            method: "POST",
            headers: {
                "Authorization": `Token ${this.apiToken}`,
                "Content-Type": "application/json"
            },
            body: JSON.stringify(data)
        });

        return response.json();
    }

    private initNotebookManager(element: HTMLElement): void {
        element.innerHTML = `
            <div class="notebook-manager">
                <div class="notebook-manager__header">
                    <h3>${this.i18n.notebookManager}</h3>
                    <button class="b3-button b3-button--small" id="refreshNotebooks">
                        ${this.i18n.refresh}
                    </button>
                </div>
                <div class="notebook-manager__list" id="notebookList">
                    ${this.renderNotebookList()}
                </div>
            </div>
        `;

        // イベントリスナーを追加
        element.querySelector("#refreshNotebooks")?.addEventListener("click", () => {
            this.refreshNotebooks();
        });

        // ノートブックアイテムのクリックイベント
        element.addEventListener("click", (e) => {
            const target = e.target as HTMLElement;
            const notebookItem = target.closest(".notebook-item");
            if (notebookItem) {
                const notebookId = notebookItem.getAttribute("data-id");
                if (notebookId) {
                    this.openNotebook(notebookId);
                }
            }
        });
    }

    private renderNotebookList(): string {
        return this.notebooks.map(notebook => `
            <div class="notebook-item ${notebook.closed ? 'notebook-item--closed' : ''}" data-id="${notebook.id}">
                <div class="notebook-item__icon">${notebook.icon}</div>
                <div class="notebook-item__name">${notebook.name}</div>
                <div class="notebook-item__status">
                    ${notebook.closed ? this.i18n.closed : this.i18n.open}
                </div>
            </div>
        `).join('');
    }

    private async refreshNotebooks(): Promise<void> {
        await this.loadNotebooks();
        const listElement = document.querySelector("#notebookList");
        if (listElement) {
            listElement.innerHTML = this.renderNotebookList();
        }
        showMessage(this.i18n.refreshed, 2000, "info");
    }

    private async openNotebook(notebookId: string): Promise<void> {
        try {
            const response = await this.apiCall("/notebook/openNotebook", {
                notebook: notebookId
            });

            if (response.code === 0) {
                showMessage(this.i18n.notebookOpened, 2000, "info");
                await this.refreshNotebooks();
            } else {
                showMessage(this.i18n.openError + response.msg, 3000, "error");
            }
        } catch (error) {
            console.error("Failed to open notebook:", error);
            showMessage(this.i18n.openError + error.message, 3000, "error");
        }
    }
}
```

### Python

#### Python APIクライアントプラグイン

```python
import json
import requests
from typing import Dict, Any, List, Optional
import asyncio
import aiohttp

class SiYuanAPIClient:
    """SiYuan API クライアント"""
    
    def __init__(self, base_url: str = "http://127.0.0.1:6806", api_token: str = ""):
        self.base_url = base_url
        self.api_token = api_token
        self.session = requests.Session()
        self.session.headers.update({
            "Authorization": f"Token {api_token}",
            "Content-Type": "application/json"
        })

    def api_call(self, endpoint: str, data: Dict[str, Any] = None) -> Dict[str, Any]:
        """API エンドポイントを呼び出し"""
        if data is None:
            data = {}
            
        url = f"{self.base_url}/api{endpoint}"
        response = self.session.post(url, json=data)
        response.raise_for_status()
        return response.json()

    def get_notebooks(self) -> List[Dict[str, Any]]:
        """ノートブック一覧を取得"""
        response = self.api_call("/notebook/lsNotebooks")
        return response.get("data", {}).get("notebooks", [])

    def create_document(self, notebook_id: str, path: str, markdown: str) -> str:
        """ドキュメントを作成"""
        response = self.api_call("/filetree/createDocWithMd", {
            "notebook": notebook_id,
            "path": path,
            "markdown": markdown
        })
        return response.get("data")

    def search_blocks(self, query: str, types: Dict[str, bool] = None) -> List[Dict[str, Any]]:
        """ブロックを検索"""
        if types is None:
            types = {
                "paragraph": True,
                "heading": True,
                "document": True
            }
            
        response = self.api_call("/search/fullTextSearchBlock", {
            "query": query,
            "types": types
        })
        return response.get("data", {}).get("blocks", [])

    def insert_block(self, data_type: str, data: str, parent_id: str) -> str:
        """ブロックを挿入"""
        response = self.api_call("/block/insertBlock", {
            "dataType": data_type,
            "data": data,
            "parentID": parent_id
        })
        return response.get("data", [{}])[0].get("doOperations", [{}])[0].get("id")

    def update_block(self, block_id: str, data_type: str, data: str) -> None:
        """ブロックを更新"""
        self.api_call("/block/updateBlock", {
            "id": block_id,
            "dataType": data_type,
            "data": data
        })

    def get_block_kramdown(self, block_id: str) -> str:
        """ブロックのKramdownを取得"""
        response = self.api_call("/block/getBlockKramdown", {
            "id": block_id
        })
        return response.get("data", {}).get("kramdown", "")

class TaskManagerPlugin:
    """タスク管理プラグイン"""
    
    def __init__(self, api_client: SiYuanAPIClient):
        self.api = api_client
        self.task_notebook_id = None

    def setup(self):
        """プラグインの初期設定"""
        # タスク専用ノートブックを作成または取得
        notebooks = self.api.get_notebooks()
        task_notebook = next((nb for nb in notebooks if nb["name"] == "Tasks"), None)
        
        if not task_notebook:
            # タスクノートブックを作成
            self.task_notebook_id = self.api.api_call("/notebook/createNotebook", {
                "name": "Tasks"
            })["data"]
        else:
            self.task_notebook_id = task_notebook["id"]

    def create_task(self, title: str, description: str = "", priority: str = "medium") -> str:
        """新しいタスクを作成"""
        task_markdown = f"""# {title}

**Priority:** {priority}
**Status:** TODO
**Created:** {self._get_current_time()}

## Description
{description}

## Notes
- [ ] Task created
"""
        
        doc_path = f"/tasks/{title.replace(' ', '-').lower()}-{self._get_timestamp()}"
        return self.api.create_document(self.task_notebook_id, doc_path, task_markdown)

    def list_tasks(self, status: str = None) -> List[Dict[str, Any]]:
        """タスク一覧を取得"""
        query = "priority AND status"
        if status:
            query += f" AND status:{status}"
            
        blocks = self.api.search_blocks(query)
        return [block for block in blocks if block.get("box") == self.task_notebook_id]

    def update_task_status(self, task_id: str, new_status: str) -> None:
        """タスクのステータスを更新"""
        kramdown = self.api.get_block_kramdown(task_id)
        
        # ステータス行を更新
        lines = kramdown.split('\n')
        for i, line in enumerate(lines):
            if line.startswith("**Status:**"):
                lines[i] = f"**Status:** {new_status}"
                break
        
        updated_kramdown = '\n'.join(lines)
        self.api.update_block(task_id, "kramdown", updated_kramdown)

    def _get_current_time(self) -> str:
        """現在時刻を取得"""
        from datetime import datetime
        return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    def _get_timestamp(self) -> str:
        """タイムスタンプを取得"""
        from datetime import datetime
        return datetime.now().strftime("%Y%m%d%H%M%S")

# 使用例
def main():
    # API クライアントを初期化
    client = SiYuanAPIClient(api_token="your-api-token-here")
    
    # タスク管理プラグインを初期化
    task_manager = TaskManagerPlugin(client)
    task_manager.setup()
    
    # 新しいタスクを作成
    task_id = task_manager.create_task(
        title="プラグイン開発を完了する",
        description="SiYuan プラグインの開発とテストを行う",
        priority="high"
    )
    print(f"Created task: {task_id}")
    
    # タスク一覧を取得
    tasks = task_manager.list_tasks()
    print(f"Found {len(tasks)} tasks")
    
    # タスクのステータスを更新
    if tasks:
        task_manager.update_task_status(tasks[0]["id"], "IN_PROGRESS")
        print("Updated task status")

if __name__ == "__main__":
    main()
```

### Go

#### Go言語によるプラグインサーバー

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "time"
)

// APIResponse represents the standard SiYuan API response
type APIResponse struct {
    Code int         `json:"code"`
    Msg  string      `json:"msg"`
    Data interface{} `json:"data"`
}

// NotebookInfo represents notebook information
type NotebookInfo struct {
    ID     string `json:"id"`
    Name   string `json:"name"`
    Icon   string `json:"icon"`
    Sort   int    `json:"sort"`
    Closed bool   `json:"closed"`
}

// BlockInfo represents block information
type BlockInfo struct {
    ID       string `json:"id"`
    ParentID string `json:"parentID"`
    RootID   string `json:"rootID"`
    Content  string `json:"content"`
    Type     string `json:"type"`
    SubType  string `json:"subType"`
}

// SiYuanClient handles API communication with SiYuan
type SiYuanClient struct {
    BaseURL   string
    APIToken  string
    HTTPClient *http.Client
}

// NewSiYuanClient creates a new SiYuan API client
func NewSiYuanClient(baseURL, apiToken string) *SiYuanClient {
    return &SiYuanClient{
        BaseURL:   baseURL,
        APIToken:  apiToken,
        HTTPClient: &http.Client{Timeout: 30 * time.Second},
    }
}

// APICall makes a POST request to the SiYuan API
func (c *SiYuanClient) APICall(endpoint string, data interface{}) (*APIResponse, error) {
    jsonData, err := json.Marshal(data)
    if err != nil {
        return nil, fmt.Errorf("failed to marshal data: %w", err)
    }

    req, err := http.NewRequest("POST", c.BaseURL+"/api"+endpoint, bytes.NewBuffer(jsonData))
    if err != nil {
        return nil, fmt.Errorf("failed to create request: %w", err)
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Token "+c.APIToken)

    resp, err := c.HTTPClient.Do(req)
    if err != nil {
        return nil, fmt.Errorf("failed to make request: %w", err)
    }
    defer resp.Body.Close()

    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("failed to read response body: %w", err)
    }

    var apiResp APIResponse
    if err := json.Unmarshal(body, &apiResp); err != nil {
        return nil, fmt.Errorf("failed to unmarshal response: %w", err)
    }

    return &apiResp, nil
}

// GetNotebooks retrieves all notebooks
func (c *SiYuanClient) GetNotebooks() ([]*NotebookInfo, error) {
    resp, err := c.APICall("/notebook/lsNotebooks", map[string]interface{}{})
    if err != nil {
        return nil, err
    }

    if resp.Code != 0 {
        return nil, fmt.Errorf("API error: %s", resp.Msg)
    }

    data, ok := resp.Data.(map[string]interface{})
    if !ok {
        return nil, fmt.Errorf("unexpected response format")
    }

    notebooksData, ok := data["notebooks"].([]interface{})
    if !ok {
        return nil, fmt.Errorf("notebooks not found in response")
    }

    var notebooks []*NotebookInfo
    for _, nb := range notebooksData {
        nbMap, ok := nb.(map[string]interface{})
        if !ok {
            continue
        }

        notebook := &NotebookInfo{
            ID:     getString(nbMap, "id"),
            Name:   getString(nbMap, "name"),
            Icon:   getString(nbMap, "icon"),
            Sort:   getInt(nbMap, "sort"),
            Closed: getBool(nbMap, "closed"),
        }
        notebooks = append(notebooks, notebook)
    }

    return notebooks, nil
}

// SearchBlocks searches for blocks with the given query
func (c *SiYuanClient) SearchBlocks(query string, types map[string]bool) ([]*BlockInfo, error) {
    data := map[string]interface{}{
        "query": query,
        "types": types,
    }

    resp, err := c.APICall("/search/fullTextSearchBlock", data)
    if err != nil {
        return nil, err
    }

    if resp.Code != 0 {
        return nil, fmt.Errorf("API error: %s", resp.Msg)
    }

    dataMap, ok := resp.Data.(map[string]interface{})
    if !ok {
        return nil, fmt.Errorf("unexpected response format")
    }

    blocksData, ok := dataMap["blocks"].([]interface{})
    if !ok {
        return nil, fmt.Errorf("blocks not found in response")
    }

    var blocks []*BlockInfo
    for _, b := range blocksData {
        blockMap, ok := b.(map[string]interface{})
        if !ok {
            continue
        }

        block := &BlockInfo{
            ID:       getString(blockMap, "id"),
            ParentID: getString(blockMap, "parentID"),
            RootID:   getString(blockMap, "rootID"),
            Content:  getString(blockMap, "content"),
            Type:     getString(blockMap, "type"),
            SubType:  getString(blockMap, "subType"),
        }
        blocks = append(blocks, block)
    }

    return blocks, nil
}

// CreateDocument creates a new document with markdown content
func (c *SiYuanClient) CreateDocument(notebookID, path, markdown string) (string, error) {
    data := map[string]interface{}{
        "notebook": notebookID,
        "path":     path,
        "markdown": markdown,
    }

    resp, err := c.APICall("/filetree/createDocWithMd", data)
    if err != nil {
        return "", err
    }

    if resp.Code != 0 {
        return "", fmt.Errorf("API error: %s", resp.Msg)
    }

    docID, ok := resp.Data.(string)
    if !ok {
        return "", fmt.Errorf("unexpected response format")
    }

    return docID, nil
}

// PluginServer represents a simple plugin server
type PluginServer struct {
    client *SiYuanClient
    port   string
}

// NewPluginServer creates a new plugin server
func NewPluginServer(client *SiYuanClient, port string) *PluginServer {
    return &PluginServer{
        client: client,
        port:   port,
    }
}

// Start starts the plugin server
func (s *PluginServer) Start() {
    http.HandleFunc("/api/notebooks", s.handleGetNotebooks)
    http.HandleFunc("/api/search", s.handleSearch)
    http.HandleFunc("/api/create-doc", s.handleCreateDocument)

    fmt.Printf("Plugin server starting on port %s\n", s.port)
    if err := http.ListenAndServe(":"+s.port, nil); err != nil {
        fmt.Printf("Server error: %v\n", err)
    }
}

func (s *PluginServer) handleGetNotebooks(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    notebooks, err := s.client.GetNotebooks()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(notebooks)
}

func (s *PluginServer) handleSearch(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    var searchReq struct {
        Query string            `json:"query"`
        Types map[string]bool   `json:"types"`
    }

    if err := json.NewDecoder(r.Body).Decode(&searchReq); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }

    if searchReq.Types == nil {
        searchReq.Types = map[string]bool{
            "paragraph": true,
            "heading":   true,
            "document":  true,
        }
    }

    blocks, err := s.client.SearchBlocks(searchReq.Query, searchReq.Types)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(blocks)
}

func (s *PluginServer) handleCreateDocument(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    var createReq struct {
        NotebookID string `json:"notebookId"`
        Path       string `json:"path"`
        Markdown   string `json:"markdown"`
    }

    if err := json.NewDecoder(r.Body).Decode(&createReq); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }

    docID, err := s.client.CreateDocument(createReq.NotebookID, createReq.Path, createReq.Markdown)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    response := map[string]string{"docId": docID}
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

// Helper functions
func getString(m map[string]interface{}, key string) string {
    if val, ok := m[key].(string); ok {
        return val
    }
    return ""
}

func getInt(m map[string]interface{}, key string) int {
    if val, ok := m[key].(float64); ok {
        return int(val)
    }
    return 0
}

func getBool(m map[string]interface{}, key string) bool {
    if val, ok := m[key].(bool); ok {
        return val
    }
    return false
}

func main() {
    // Initialize SiYuan client
    client := NewSiYuanClient("http://127.0.0.1:6806", "your-api-token-here")
    
    // Create and start plugin server
    server := NewPluginServer(client, "8080")
    server.Start()
}
```

---

## UI操作プラグイン例

### パネル管理プラグイン

```javascript
// パネル管理プラグイン - 複数のカスタムパネルを管理
export default class PanelManagerPlugin extends Plugin {
    
    private panels: Map<string, any> = new Map();

    async onload() {
        // 複数のカスタムパネルを作成
        this.createFileExplorerPanel();
        this.createTaskPanel();
        this.createCalendarPanel();

        // パネル制御メニューを追加
        this.addTopBar({
            icon: "iconLayout",
            title: "Panel Manager",
            callback: () => this.showPanelMenu()
        });
    }

    private createFileExplorerPanel() {
        const panel = this.addDock({
            config: {
                position: "LeftTop",
                size: {width: 300, height: 400},
                icon: "iconFolder",
                title: "File Explorer Plus",
                show: true
            },
            data: {},
            type: "file-explorer-plus",
            init: (dock) => {
                this.initFileExplorer(dock.element);
            }
        });
        this.panels.set("fileExplorer", panel);
    }

    private createTaskPanel() {
        const panel = this.addDock({
            config: {
                position: "RightTop",
                size: {width: 350, height: 500},
                icon: "iconList",
                title: "Task Manager",
                show: false
            },
            data: {},
            type: "task-manager",
            init: (dock) => {
                this.initTaskManager(dock.element);
            }
        });
        this.panels.set("taskManager", panel);
    }

    private createCalendarPanel() {
        const panel = this.addDock({
            config: {
                position: "RightBottom",
                size: {width: 300, height: 300},
                icon: "iconCalendar",
                title: "Calendar View",
                show: false
            },
            data: {},
            type: "calendar-view",
            init: (dock) => {
                this.initCalendarView(dock.element);
            }
        });
        this.panels.set("calendar", panel);
    }

    private initFileExplorer(element: HTMLElement) {
        element.innerHTML = `
            <div class="file-explorer-plus">
                <div class="explorer-toolbar">
                    <button class="b3-button b3-button--small" id="refreshFiles">
                        <svg><use xlink:href="#iconRefresh"></use></svg>
                    </button>
                    <button class="b3-button b3-button--small" id="newFolder">
                        <svg><use xlink:href="#iconFolder"></use></svg>
                    </button>
                    <input type="text" class="b3-text-field" placeholder="Search files..." id="fileSearch">
                </div>
                <div class="explorer-tree" id="fileTree">
                    <div class="tree-loading">Loading files...</div>
                </div>
            </div>
        `;

        this.loadFileTree(element);
        this.bindFileExplorerEvents(element);
    }

    private async loadFileTree(element: HTMLElement) {
        try {
            const notebooks = await this.apiRequest("/notebook/lsNotebooks", {});
            const treeHtml = this.renderFileTree(notebooks.data.notebooks);
            element.querySelector("#fileTree").innerHTML = treeHtml;
        } catch (error) {
            console.error("Failed to load file tree:", error);
        }
    }

    private renderFileTree(notebooks: any[]): string {
        return notebooks.map(notebook => `
            <div class="tree-node" data-type="notebook" data-id="${notebook.id}">
                <div class="tree-node__header">
                    <span class="tree-node__icon">${notebook.icon}</span>
                    <span class="tree-node__title">${notebook.name}</span>
                    <span class="tree-node__toggle">▶</span>
                </div>
                <div class="tree-node__children" style="display: none;">
                    <div class="tree-loading">Loading...</div>
                </div>
            </div>
        `).join('');
    }

    private initTaskManager(element: HTMLElement) {
        element.innerHTML = `
            <div class="task-manager">
                <div class="task-header">
                    <h3>My Tasks</h3>
                    <button class="b3-button b3-button--small" id="addTask">
                        <svg><use xlink:href="#iconAdd"></use></svg>
                        Add Task
                    </button>
                </div>
                <div class="task-filters">
                    <button class="task-filter active" data-filter="all">All</button>
                    <button class="task-filter" data-filter="todo">Todo</button>
                    <button class="task-filter" data-filter="doing">Doing</button>
                    <button class="task-filter" data-filter="done">Done</button>
                </div>
                <div class="task-list" id="taskList">
                    <div class="task-loading">Loading tasks...</div>
                </div>
            </div>
        `;

        this.loadTasks(element);
        this.bindTaskEvents(element);
    }

    private async loadTasks(element: HTMLElement) {
        // タスク専用の検索クエリ
        const taskQuery = 'content:"- [ ]" OR content:"- [x]"';
        
        try {
            const response = await this.apiRequest("/search/fullTextSearchBlock", {
                query: taskQuery,
                types: { listItem: true }
            });

            const tasks = this.parseTasks(response.data.blocks);
            element.querySelector("#taskList").innerHTML = this.renderTasks(tasks);
        } catch (error) {
            console.error("Failed to load tasks:", error);
        }
    }

    private parseTasks(blocks: any[]): any[] {
        return blocks.map(block => {
            const isCompleted = block.content.includes("- [x]");
            const taskText = block.content.replace(/^- \[[x ]\]\s*/, "");
            
            return {
                id: block.id,
                text: taskText,
                completed: isCompleted,
                rootId: block.rootID,
                path: block.hPath
            };
        });
    }

    private renderTasks(tasks: any[]): string {
        return tasks.map(task => `
            <div class="task-item ${task.completed ? 'completed' : ''}" data-id="${task.id}">
                <input type="checkbox" ${task.completed ? 'checked' : ''} class="task-checkbox">
                <span class="task-text">${task.text}</span>
                <div class="task-actions">
                    <button class="task-action" data-action="edit">
                        <svg><use xlink:href="#iconEdit"></use></svg>
                    </button>
                    <button class="task-action" data-action="delete">
                        <svg><use xlink:href="#iconTrashcan"></use></svg>
                    </button>
                </div>
            </div>
        `).join('');
    }

    private initCalendarView(element: HTMLElement) {
        element.innerHTML = `
            <div class="calendar-view">
                <div class="calendar-header">
                    <button class="calendar-nav" id="prevMonth">‹</button>
                    <h3 id="currentMonth">Loading...</h3>
                    <button class="calendar-nav" id="nextMonth">›</button>
                </div>
                <div class="calendar-grid" id="calendarGrid">
                    <div class="calendar-loading">Loading calendar...</div>
                </div>
            </div>
        `;

        this.renderCalendar(element);
        this.bindCalendarEvents(element);
    }

    private showPanelMenu() {
        const menu = new Menu("panelManager");
        
        this.panels.forEach((panel, key) => {
            menu.addItem({
                icon: "iconEye",
                label: `Toggle ${key}`,
                click: () => {
                    this.togglePanel(key);
                }
            });
        });

        menu.addSeparator();
        
        menu.addItem({
            icon: "iconRefresh",
            label: "Refresh All Panels",
            click: () => {
                this.refreshAllPanels();
            }
        });

        menu.addItem({
            icon: "iconSettings",
            label: "Panel Settings",
            click: () => {
                this.showPanelSettings();
            }
        });

        menu.popup();
    }

    private togglePanel(panelKey: string) {
        // パネルの表示/非表示を切り替え
        const panel = this.panels.get(panelKey);
        if (panel) {
            // パネルの表示状態をAPIで制御
            this.apiRequest("/ui/reloadUI", {});
        }
    }

    private refreshAllPanels() {
        // すべてのパネルをリフレッシュ
        this.panels.forEach((panel, key) => {
            const element = document.querySelector(`[data-type="${key}"]`);
            if (element) {
                this.refreshPanelContent(key, element as HTMLElement);
            }
        });
    }

    private refreshPanelContent(panelKey: string, element: HTMLElement) {
        switch (panelKey) {
            case "fileExplorer":
                this.loadFileTree(element);
                break;
            case "taskManager":
                this.loadTasks(element);
                break;
            case "calendar":
                this.renderCalendar(element);
                break;
        }
    }

    // その他のヘルパーメソッド...
    private bindFileExplorerEvents(element: HTMLElement) { /* ... */ }
    private bindTaskEvents(element: HTMLElement) { /* ... */ }
    private bindCalendarEvents(element: HTMLElement) { /* ... */ }
    private renderCalendar(element: HTMLElement) { /* ... */ }
    private showPanelSettings() { /* ... */ }
}
```

---

## データ統合プラグイン例

### データベース操作プラグイン

```javascript
// 高度なデータベース操作プラグイン
export default class DatabasePlugin extends Plugin {

    async onload() {
        // データベース管理用のトップバーアイコンを追加
        this.addTopBar({
            icon: "iconDatabase",
            title: "Database Manager",
            callback: () => this.showDatabaseManager()
        });

        // コマンドパレットにコマンドを追加
        this.addCommand({
            langKey: "createDatabase",
            hotkey: "⌘⌥B",
            callback: () => this.createDatabaseDialog()
        });
    }

    private async showDatabaseManager() {
        // データベース（属性ビュー）の一覧を取得
        const databases = await this.findAllDatabases();
        
        const dialog = new Dialog({
            title: "Database Manager",
            content: `
                <div class="database-manager">
                    <div class="database-toolbar">
                        <button class="b3-button" id="createDb">New Database</button>
                        <button class="b3-button b3-button--outline" id="importDb">Import</button>
                        <button class="b3-button b3-button--outline" id="exportDb">Export</button>
                    </div>
                    <div class="database-list">
                        ${this.renderDatabaseList(databases)}
                    </div>
                </div>
            `,
            width: 800,
            height: 600
        });

        this.bindDatabaseManagerEvents(dialog.element, databases);
    }

    private async findAllDatabases(): Promise<any[]> {
        // 属性ビューブロックを検索
        const response = await this.apiRequest("/search/fullTextSearchBlock", {
            query: "type:av",
            types: { av: true }
        });

        const databases = [];
        for (const block of response.data.blocks) {
            try {
                const avData = await this.apiRequest("/av/getAttributeView", {
                    id: block.id
                });
                
                if (avData.code === 0) {
                    databases.push({
                        ...block,
                        avData: avData.data
                    });
                }
            } catch (error) {
                console.warn(`Failed to get AV data for ${block.id}:`, error);
            }
        }

        return databases;
    }

    private renderDatabaseList(databases: any[]): string {
        if (databases.length === 0) {
            return '<div class="database-empty">No databases found. Create your first database!</div>';
        }

        return databases.map(db => `
            <div class="database-item" data-id="${db.id}">
                <div class="database-header">
                    <h3 class="database-title">${db.name || 'Untitled Database'}</h3>
                    <div class="database-actions">
                        <button class="b3-button b3-button--small" data-action="view">View</button>
                        <button class="b3-button b3-button--small" data-action="edit">Edit</button>
                        <button class="b3-button b3-button--small" data-action="clone">Clone</button>
                        <button class="b3-button b3-button--small b3-button--cancel" data-action="delete">Delete</button>
                    </div>
                </div>
                <div class="database-info">
                    <span class="database-stat">
                        Keys: ${db.avData?.keyValues?.length || 0}
                    </span>
                    <span class="database-stat">
                        Records: ${this.countRecords(db.avData)}
                    </span>
                    <span class="database-path">${db.hPath}</span>
                </div>
            </div>
        `).join('');
    }

    private countRecords(avData: any): number {
        if (!avData?.keyValues?.length) return 0;
        return avData.keyValues[0]?.values?.length || 0;
    }

    private bindDatabaseManagerEvents(element: HTMLElement, databases: any[]) {
        // 新しいデータベース作成
        element.querySelector("#createDb")?.addEventListener("click", () => {
            this.createDatabaseDialog();
        });

        // データベースアクション
        element.addEventListener("click", async (e) => {
            const target = e.target as HTMLElement;
            const action = target.getAttribute("data-action");
            const dbItem = target.closest(".database-item");
            
            if (!action || !dbItem) return;

            const dbId = dbItem.getAttribute("data-id");
            const database = databases.find(db => db.id === dbId);

            switch (action) {
                case "view":
                    this.viewDatabase(database);
                    break;
                case "edit":
                    this.editDatabase(database);
                    break;
                case "clone":
                    await this.cloneDatabase(database);
                    break;
                case "delete":
                    await this.deleteDatabase(database);
                    break;
            }
        });
    }

    private async createDatabaseDialog() {
        const dialog = new Dialog({
            title: "Create New Database",
            content: `
                <div class="database-creator">
                    <div class="form-group">
                        <label>Database Name:</label>
                        <input type="text" class="b3-text-field" id="dbName" placeholder="My Database">
                    </div>
                    <div class="form-group">
                        <label>Template:</label>
                        <select class="b3-select" id="dbTemplate">
                            <option value="blank">Blank Database</option>
                            <option value="tasks">Task Management</option>
                            <option value="contacts">Contact List</option>
                            <option value="inventory">Inventory</option>
                            <option value="projects">Project Tracker</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>Location:</label>
                        <select class="b3-select" id="dbNotebook">
                            <!-- Notebooks will be populated -->
                        </select>
                    </div>
                    <div class="form-actions">
                        <button class="b3-button b3-button--cancel" id="cancelCreate">Cancel</button>
                        <button class="b3-button" id="confirmCreate">Create Database</button>
                    </div>
                </div>
            `,
            width: 500,
            height: 400
        });

        // ノートブック一覧を読み込み
        await this.populateNotebookSelect(dialog.element.querySelector("#dbNotebook"));

        // イベントハンドラー
        dialog.element.querySelector("#confirmCreate")?.addEventListener("click", async () => {
            await this.handleCreateDatabase(dialog);
        });

        dialog.element.querySelector("#cancelCreate")?.addEventListener("click", () => {
            dialog.destroy();
        });
    }

    private async populateNotebookSelect(selectElement: HTMLSelectElement) {
        try {
            const response = await this.apiRequest("/notebook/lsNotebooks", {});
            const notebooks = response.data.notebooks;

            notebooks.forEach((notebook: any) => {
                const option = document.createElement("option");
                option.value = notebook.id;
                option.textContent = notebook.name;
                selectElement.appendChild(option);
            });
        } catch (error) {
            console.error("Failed to load notebooks:", error);
        }
    }

    private async handleCreateDatabase(dialog: Dialog) {
        const nameInput = dialog.element.querySelector("#dbName") as HTMLInputElement;
        const templateSelect = dialog.element.querySelector("#dbTemplate") as HTMLSelectElement;
        const notebookSelect = dialog.element.querySelector("#dbNotebook") as HTMLSelectElement;

        const dbName = nameInput.value.trim() || "New Database";
        const template = templateSelect.value;
        const notebookId = notebookSelect.value;

        try {
            // データベーステンプレートに基づいてマークダウンを生成
            const markdown = this.generateDatabaseMarkdown(dbName, template);
            
            // ドキュメントを作成
            const docResponse = await this.apiRequest("/filetree/createDocWithMd", {
                notebook: notebookId,
                path: `/${dbName}`,
                markdown: markdown
            });

            if (docResponse.code === 0) {
                showMessage("Database created successfully!", 2000, "info");
                dialog.destroy();
                
                // 作成したデータベースを開く
                openTab({
                    app: this.app,
                    doc: {
                        id: docResponse.data,
                        action: ["cb-get-focus"]
                    }
                });
            } else {
                showMessage("Failed to create database: " + docResponse.msg, 3000, "error");
            }
        } catch (error) {
            console.error("Database creation error:", error);
            showMessage("Failed to create database: " + error.message, 3000, "error");
        }
    }

    private generateDatabaseMarkdown(name: string, template: string): string {
        const baseMarkdown = `# ${name}\n\n`;
        
        switch (template) {
            case "tasks":
                return baseMarkdown + `
| Task | Status | Priority | Due Date | Assignee |
|------|--------|----------|----------|----------|
| Sample Task | Todo | High | 2024-01-15 | Me |
{: id="20210808180117-" updated="20210808180117"}
`;

            case "contacts":
                return baseMarkdown + `
| Name | Email | Phone | Company | Notes |
|------|-------|-------|---------|-------|
| John Doe | john@example.com | 555-0123 | Example Corp | Sample contact |
{: id="20210808180117-" updated="20210808180117"}
`;

            case "inventory":
                return baseMarkdown + `
| Item | Quantity | Category | Location | Cost |
|------|----------|----------|----------|------|
| Sample Item | 10 | Electronics | Warehouse A | $50.00 |
{: id="20210808180117-" updated="20210808180117"}
`;

            case "projects":
                return baseMarkdown + `
| Project | Status | Start Date | End Date | Progress |
|---------|--------|------------|----------|----------|
| Sample Project | Active | 2024-01-01 | 2024-06-30 | 25% |
{: id="20210808180117-" updated="20210808180117"}
`;

            default:
                return baseMarkdown + `
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Data 1 | Data 2 | Data 3 |
{: id="20210808180117-" updated="20210808180117"}
`;
        }
    }

    private async viewDatabase(database: any) {
        // データベースビューを表示
        openTab({
            app: this.app,
            doc: {
                id: database.id,
                action: ["cb-get-focus"]
            }
        });
    }

    private async editDatabase(database: any) {
        // データベース編集ダイアログを表示
        const dialog = new Dialog({
            title: `Edit Database: ${database.name}`,
            content: `
                <div class="database-editor">
                    <div class="editor-tabs">
                        <button class="editor-tab active" data-tab="structure">Structure</button>
                        <button class="editor-tab" data-tab="data">Data</button>
                        <button class="editor-tab" data-tab="settings">Settings</button>
                    </div>
                    <div class="editor-content">
                        <div class="editor-panel" id="structurePanel">
                            ${await this.renderStructureEditor(database)}
                        </div>
                        <div class="editor-panel" id="dataPanel" style="display: none;">
                            ${await this.renderDataEditor(database)}
                        </div>
                        <div class="editor-panel" id="settingsPanel" style="display: none;">
                            ${this.renderSettingsEditor(database)}
                        </div>
                    </div>
                </div>
            `,
            width: 900,
            height: 700
        });

        this.bindDatabaseEditorEvents(dialog.element, database);
    }

    private async renderStructureEditor(database: any): Promise<string> {
        const avData = database.avData;
        const keys = avData?.keyValues || [];

        return `
            <div class="structure-editor">
                <div class="structure-toolbar">
                    <button class="b3-button" id="addColumn">Add Column</button>
                    <button class="b3-button b3-button--outline" id="reorderColumns">Reorder</button>
                </div>
                <div class="columns-list">
                    ${keys.map(keyValue => `
                        <div class="column-item" data-key-id="${keyValue.key.id}">
                            <div class="column-header">
                                <input type="text" class="column-name" value="${keyValue.key.name}">
                                <select class="column-type">
                                    <option value="text" ${keyValue.key.type === 'text' ? 'selected' : ''}>Text</option>
                                    <option value="number" ${keyValue.key.type === 'number' ? 'selected' : ''}>Number</option>
                                    <option value="select" ${keyValue.key.type === 'select' ? 'selected' : ''}>Select</option>
                                    <option value="date" ${keyValue.key.type === 'date' ? 'selected' : ''}>Date</option>
                                    <option value="checkbox" ${keyValue.key.type === 'checkbox' ? 'selected' : ''}>Checkbox</option>
                                </select>
                                <button class="column-delete">×</button>
                            </div>
                            ${keyValue.key.type === 'select' ? this.renderSelectOptions(keyValue.key.options) : ''}
                        </div>
                    `).join('')}
                </div>
            </div>
        `;
    }

    private renderSelectOptions(options: any[]): string {
        if (!options) return '';
        
        return `
            <div class="select-options">
                <h4>Options:</h4>
                ${options.map(option => `
                    <div class="option-item">
                        <input type="text" value="${option.name}" class="option-name">
                        <input type="color" value="${option.color}" class="option-color">
                        <button class="option-delete">×</button>
                    </div>
                `).join('')}
                <button class="b3-button b3-button--small" onclick="addSelectOption(this)">Add Option</button>
            </div>
        `;
    }

    // ... 他のメソッドは省略（実装は同様のパターン）
}
```

この実装例では、以下の機能を提供しています：

1. **UI操作機能**
   - 複数のカスタムパネル管理
   - 動的なダイアログ生成
   - リアルタイムなUI更新

2. **データベース統合**
   - 属性ビュー（データベース）の作成・編集
   - テンプレートベースのデータベース生成
   - 構造化データの管理

3. **多言語対応**
   - JavaScript/TypeScript、Python、Go での実装例
   - 各言語での API クライアント実装

4. **実践的な機能**
   - ファイル管理
   - タスク管理
   - カレンダー表示
   - データベース操作

これらの例を参考に、様々な用途のプラグインを開発できます。
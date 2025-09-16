# SiYuan 高度なプラグインパターン

このドキュメントでは、SiYuan APIを活用した高度なプラグイン開発パターンとベストプラクティスを紹介します。

## 目次

- [AI統合プラグイン](#ai統合プラグイン)
- [外部サービス連携](#外部サービス連携)
- [ワークフロー自動化](#ワークフロー自動化)
- [プラグイン間通信](#プラグイン間通信)
- [パフォーマンス最適化](#パフォーマンス最適化)
- [エラーハンドリング](#エラーハンドリング)
- [テストとデバッグ](#テストとデバッグ)
- [配布とメンテナンス](#配布とメンテナンス)

---

## AI統合プラグイン

### ChatGPT統合プラグイン

```typescript
import { Plugin, showMessage, Dialog } from "siyuan";

interface AIResponse {
    choices: Array<{
        message: {
            content: string;
            role: string;
        };
    }>;
}

export default class AIAssistantPlugin extends Plugin {
    private apiKey: string = "";
    private conversationHistory: Array<any> = [];

    async onload() {
        // AI設定の読み込み
        await this.loadAISettings();

        // AIアシスタントボタンを追加
        this.addTopBar({
            icon: "iconSparkles",
            title: "AI Assistant",
            callback: () => this.showAIDialog()
        });

        // 選択テキストを分析するコマンド
        this.addCommand({
            langKey: "analyzeText",
            hotkey: "⌘⌥A",
            callback: () => this.analyzeSelectedText()
        });

        // 自動要約機能
        this.addCommand({
            langKey: "autoSummarize",
            hotkey: "⌘⌥S",
            callback: () => this.summarizeDocument()
        });
    }

    private async loadAISettings() {
        // 設定からAPIキーを読み込み
        this.apiKey = this.data[STORAGE_NAME]?.apiKey || "";
        
        // 設定項目を追加
        this.setting.addItem({
            key: "apiKey",
            value: this.apiKey,
            type: "textinput",
            title: "OpenAI API Key",
            description: "Enter your OpenAI API key for AI features",
        });

        this.setting.addItem({
            key: "model",
            value: this.data[STORAGE_NAME]?.model || "gpt-3.5-turbo",
            type: "select",
            title: "AI Model",
            description: "Select the AI model to use",
            options: {
                "gpt-3.5-turbo": "GPT-3.5 Turbo",
                "gpt-4": "GPT-4",
                "gpt-4-turbo": "GPT-4 Turbo"
            }
        });
    }

    private async showAIDialog() {
        const dialog = new Dialog({
            title: "AI Assistant",
            content: `
                <div class="ai-assistant">
                    <div class="chat-container">
                        <div class="chat-history" id="chatHistory">
                            ${this.renderChatHistory()}
                        </div>
                        <div class="chat-input-container">
                            <div class="input-toolbar">
                                <button class="b3-button b3-button--small" id="insertContext">
                                    Insert Current Document
                                </button>
                                <button class="b3-button b3-button--small" id="clearHistory">
                                    Clear History
                                </button>
                            </div>
                            <div class="input-area">
                                <textarea class="b3-text-field" id="userInput" 
                                         placeholder="Ask me anything about your notes..."
                                         rows="3"></textarea>
                                <button class="b3-button" id="sendMessage">Send</button>
                            </div>
                        </div>
                    </div>
                    <div class="ai-actions">
                        <button class="b3-button b3-button--outline" id="summarizeDoc">
                            Summarize Current Document
                        </button>
                        <button class="b3-button b3-button--outline" id="generateOutline">
                            Generate Outline
                        </button>
                        <button class="b3-button b3-button--outline" id="improveWriting">
                            Improve Writing
                        </button>
                    </div>
                </div>
            `,
            width: 700,
            height: 600
        });

        this.bindAIDialogEvents(dialog.element);
    }

    private bindAIDialogEvents(element: HTMLElement) {
        const userInput = element.querySelector("#userInput") as HTMLTextAreaElement;
        const sendButton = element.querySelector("#sendMessage") as HTMLButtonElement;
        const chatHistory = element.querySelector("#chatHistory") as HTMLElement;

        // メッセージ送信
        const sendMessage = async () => {
            const message = userInput.value.trim();
            if (!message) return;

            // ユーザーメッセージを表示
            this.addMessageToHistory("user", message);
            this.updateChatDisplay(chatHistory);
            userInput.value = "";
            sendButton.disabled = true;
            sendButton.textContent = "Thinking...";

            try {
                // AIに送信
                const response = await this.callAI(message);
                this.addMessageToHistory("assistant", response);
                this.updateChatDisplay(chatHistory);
            } catch (error) {
                console.error("AI request failed:", error);
                showMessage("AI request failed: " + error.message, 3000, "error");
            } finally {
                sendButton.disabled = false;
                sendButton.textContent = "Send";
            }
        };

        sendButton.addEventListener("click", sendMessage);
        userInput.addEventListener("keydown", (e) => {
            if (e.key === "Enter" && (e.ctrlKey || e.metaKey)) {
                sendMessage();
            }
        });

        // コンテキスト挿入
        element.querySelector("#insertContext")?.addEventListener("click", async () => {
            const currentDoc = await this.getCurrentDocumentContent();
            if (currentDoc) {
                userInput.value += `\n\nCurrent document context:\n${currentDoc.content}`;
            }
        });

        // 履歴クリア
        element.querySelector("#clearHistory")?.addEventListener("click", () => {
            this.conversationHistory = [];
            this.updateChatDisplay(chatHistory);
        });

        // クイックアクション
        element.querySelector("#summarizeDoc")?.addEventListener("click", () => {
            this.performQuickAction("summarize");
        });

        element.querySelector("#generateOutline")?.addEventListener("click", () => {
            this.performQuickAction("outline");
        });

        element.querySelector("#improveWriting")?.addEventListener("click", () => {
            this.performQuickAction("improve");
        });
    }

    private async callAI(message: string): Promise<string> {
        if (!this.apiKey) {
            throw new Error("OpenAI API key not configured");
        }

        // SiYuan内蔵のAI機能を使用
        try {
            const response = await this.apiRequest("/ai/chatGPT", {
                msg: message,
                context: this.buildContext()
            });

            if (response.code === 0 && response.data.choices?.length > 0) {
                return response.data.choices[0].message.content;
            } else {
                throw new Error(response.msg || "AI response error");
            }
        } catch (error) {
            // フォールバック: 直接OpenAI APIを呼び出し
            return await this.callOpenAIDirectly(message);
        }
    }

    private async callOpenAIDirectly(message: string): Promise<string> {
        const response = await fetch("https://api.openai.com/v1/chat/completions", {
            method: "POST",
            headers: {
                "Authorization": `Bearer ${this.apiKey}`,
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                model: this.data[STORAGE_NAME]?.model || "gpt-3.5-turbo",
                messages: [
                    ...this.conversationHistory,
                    { role: "user", content: message }
                ],
                max_tokens: 1000,
                temperature: 0.7
            })
        });

        if (!response.ok) {
            throw new Error(`OpenAI API error: ${response.statusText}`);
        }

        const data: AIResponse = await response.json();
        return data.choices[0].message.content;
    }

    private buildContext(): string {
        // 現在のドキュメントやノートブックの情報を含むコンテキストを構築
        return this.conversationHistory
            .map(msg => `${msg.role}: ${msg.content}`)
            .join("\n");
    }

    private addMessageToHistory(role: string, content: string) {
        this.conversationHistory.push({ role, content, timestamp: Date.now() });
        
        // 履歴が長くなりすぎないよう制限
        if (this.conversationHistory.length > 20) {
            this.conversationHistory.shift();
        }
    }

    private renderChatHistory(): string {
        return this.conversationHistory.map(msg => `
            <div class="chat-message ${msg.role}">
                <div class="message-header">
                    <span class="message-role">${msg.role === 'user' ? 'You' : 'AI'}</span>
                    <span class="message-time">${new Date(msg.timestamp).toLocaleTimeString()}</span>
                </div>
                <div class="message-content">${this.formatMessage(msg.content)}</div>
            </div>
        `).join('');
    }

    private formatMessage(content: string): string {
        // マークダウンを簡単にHTMLに変換
        return content
            .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
            .replace(/\*(.*?)\*/g, '<em>$1</em>')
            .replace(/`(.*?)`/g, '<code>$1</code>')
            .replace(/\n/g, '<br>');
    }

    private updateChatDisplay(chatHistory: HTMLElement) {
        chatHistory.innerHTML = this.renderChatHistory();
        chatHistory.scrollTop = chatHistory.scrollHeight;
    }

    private async performQuickAction(action: string) {
        const currentDoc = await this.getCurrentDocumentContent();
        if (!currentDoc) {
            showMessage("No document selected", 2000, "error");
            return;
        }

        let prompt = "";
        switch (action) {
            case "summarize":
                prompt = `Please provide a concise summary of the following document:\n\n${currentDoc.content}`;
                break;
            case "outline":
                prompt = `Please create a detailed outline for the following document:\n\n${currentDoc.content}`;
                break;
            case "improve":
                prompt = `Please improve the writing quality and clarity of the following text:\n\n${currentDoc.content}`;
                break;
        }

        try {
            const response = await this.callAI(prompt);
            
            // 結果を新しいドキュメントとして作成
            await this.createResultDocument(action, response, currentDoc.name);
            showMessage(`${action} completed and saved as new document`, 3000, "info");
        } catch (error) {
            console.error(`Failed to perform ${action}:`, error);
            showMessage(`Failed to perform ${action}: ${error.message}`, 3000, "error");
        }
    }

    private async getCurrentDocumentContent(): Promise<{content: string, name: string, id: string} | null> {
        // 現在アクティブなタブのドキュメント内容を取得
        const activeTab = document.querySelector('.layout__tab--active');
        if (!activeTab) return null;

        const docId = activeTab.getAttribute('data-id');
        if (!docId) return null;

        try {
            const response = await this.apiRequest("/block/getBlockKramdown", {
                id: docId
            });

            if (response.code === 0) {
                return {
                    content: response.data.kramdown,
                    name: activeTab.textContent?.trim() || "Untitled",
                    id: docId
                };
            }
        } catch (error) {
            console.error("Failed to get document content:", error);
        }

        return null;
    }

    private async createResultDocument(action: string, content: string, originalName: string) {
        const notebooks = await this.apiRequest("/notebook/lsNotebooks", {});
        const defaultNotebook = notebooks.data.notebooks[0]?.id;

        if (!defaultNotebook) {
            throw new Error("No notebook available");
        }

        const title = `${action.charAt(0).toUpperCase() + action.slice(1)} of ${originalName}`;
        const markdown = `# ${title}\n\n${content}`;

        await this.apiRequest("/filetree/createDocWithMd", {
            notebook: defaultNotebook,
            path: `/${title}`,
            markdown: markdown
        });
    }

    private async analyzeSelectedText() {
        // 選択されたテキストを取得して分析
        const selection = window.getSelection()?.toString();
        if (!selection) {
            showMessage("No text selected", 2000, "error");
            return;
        }

        try {
            const analysis = await this.callAI(
                `Please analyze the following text and provide insights about its content, style, and suggestions for improvement:\n\n${selection}`
            );

            // 分析結果をダイアログで表示
            new Dialog({
                title: "Text Analysis",
                content: `
                    <div class="text-analysis">
                        <h3>Original Text:</h3>
                        <div class="original-text">${selection}</div>
                        <h3>Analysis:</h3>
                        <div class="analysis-result">${this.formatMessage(analysis)}</div>
                    </div>
                `,
                width: 600,
                height: 500
            });
        } catch (error) {
            console.error("Text analysis failed:", error);
            showMessage("Text analysis failed: " + error.message, 3000, "error");
        }
    }

    private async summarizeDocument() {
        const currentDoc = await this.getCurrentDocumentContent();
        if (!currentDoc) {
            showMessage("No document to summarize", 2000, "error");
            return;
        }

        try {
            const summary = await this.callAI(
                `Please provide a comprehensive summary of the following document, including main points, key concepts, and conclusions:\n\n${currentDoc.content}`
            );

            // 要約を現在のドキュメントの先頭に挿入
            await this.insertSummaryBlock(currentDoc.id, summary);
            showMessage("Summary inserted at document top", 2000, "info");
        } catch (error) {
            console.error("Summarization failed:", error);
            showMessage("Summarization failed: " + error.message, 3000, "error");
        }
    }

    private async insertSummaryBlock(docId: string, summary: string) {
        const summaryMarkdown = `> **AI Summary**
> 
> ${summary.replace(/\n/g, '\n> ')}

---

`;

        await this.apiRequest("/block/prependBlock", {
            parentID: docId,
            dataType: "markdown",
            data: summaryMarkdown
        });
    }
}
```

---

## 外部サービス連携

### GitHub統合プラグイン

```typescript
import { Plugin, showMessage, Dialog, Menu } from "siyuan";

interface GitHubRepo {
    id: number;
    name: string;
    full_name: string;
    description: string;
    html_url: string;
    clone_url: string;
}

interface GitHubIssue {
    id: number;
    number: number;
    title: string;
    body: string;
    state: string;
    html_url: string;
    created_at: string;
    updated_at: string;
}

export default class GitHubIntegrationPlugin extends Plugin {
    private githubToken: string = "";
    private username: string = "";
    private selectedRepo: string = "";

    async onload() {
        // GitHub設定の読み込み
        await this.loadGitHubSettings();

        // GitHubパネルを追加
        this.addDock({
            config: {
                position: "RightTop",
                size: { width: 350, height: 500 },
                icon: "iconGithub",
                title: "GitHub Integration"
            },
            data: {},
            type: "github-panel",
            init: (dock) => {
                this.initGitHubPanel(dock.element);
            }
        });

        // コマンドを追加
        this.addCommand({
            langKey: "syncToGitHub",
            hotkey: "⌘⌥G",
            callback: () => this.syncCurrentDocumentToGitHub()
        });

        this.addCommand({
            langKey: "importFromGitHub",
            hotkey: "⌘⌥I",
            callback: () => this.importFromGitHub()
        });
    }

    private async loadGitHubSettings() {
        this.githubToken = this.data[STORAGE_NAME]?.githubToken || "";
        this.username = this.data[STORAGE_NAME]?.username || "";
        this.selectedRepo = this.data[STORAGE_NAME]?.selectedRepo || "";

        // 設定項目を追加
        this.setting.addItem({
            key: "githubToken",
            value: this.githubToken,
            type: "textinput",
            title: "GitHub Personal Access Token",
            description: "Enter your GitHub PAT for API access"
        });

        this.setting.addItem({
            key: "username",
            value: this.username,
            type: "textinput",
            title: "GitHub Username",
            description: "Your GitHub username"
        });
    }

    private async initGitHubPanel(element: HTMLElement) {
        element.innerHTML = `
            <div class="github-panel">
                <div class="github-header">
                    <h3>GitHub Integration</h3>
                    <button class="b3-button b3-button--small" id="refreshGitHub">
                        <svg><use xlink:href="#iconRefresh"></use></svg>
                    </button>
                </div>
                
                <div class="github-section">
                    <h4>Repository</h4>
                    <select class="b3-select" id="repoSelect">
                        <option value="">Select Repository...</option>
                    </select>
                </div>

                <div class="github-section">
                    <h4>Issues</h4>
                    <div class="issues-container" id="issuesContainer">
                        <div class="loading">Loading issues...</div>
                    </div>
                    <button class="b3-button b3-button--outline" id="createIssue">
                        Create Issue from Document
                    </button>
                </div>

                <div class="github-section">
                    <h4>Quick Actions</h4>
                    <div class="action-buttons">
                        <button class="b3-button b3-button--small" id="syncDoc">
                            Sync Current Doc
                        </button>
                        <button class="b3-button b3-button--small" id="importReadme">
                            Import README
                        </button>
                        <button class="b3-button b3-button--small" id="exportNotes">
                            Export to Repo
                        </button>
                    </div>
                </div>
            </div>
        `;

        await this.loadRepositories(element);
        this.bindGitHubPanelEvents(element);
    }

    private async loadRepositories(element: HTMLElement) {
        if (!this.githubToken || !this.username) {
            element.querySelector("#issuesContainer").innerHTML = 
                '<div class="error">Please configure GitHub credentials in settings</div>';
            return;
        }

        try {
            const repos = await this.fetchGitHubAPI<GitHubRepo[]>(`/user/repos?sort=updated&per_page=50`);
            const repoSelect = element.querySelector("#repoSelect") as HTMLSelectElement;
            
            repoSelect.innerHTML = '<option value="">Select Repository...</option>';
            repos.forEach(repo => {
                const option = document.createElement("option");
                option.value = repo.full_name;
                option.textContent = repo.name;
                repoSelect.appendChild(option);
            });

            if (this.selectedRepo) {
                repoSelect.value = this.selectedRepo;
                await this.loadIssues(element, this.selectedRepo);
            }
        } catch (error) {
            console.error("Failed to load repositories:", error);
            element.querySelector("#issuesContainer").innerHTML = 
                `<div class="error">Failed to load repositories: ${error.message}</div>`;
        }
    }

    private async loadIssues(element: HTMLElement, repoName: string) {
        const issuesContainer = element.querySelector("#issuesContainer");
        if (!issuesContainer) return;

        issuesContainer.innerHTML = '<div class="loading">Loading issues...</div>';

        try {
            const issues = await this.fetchGitHubAPI<GitHubIssue[]>(`/repos/${repoName}/issues?state=open&per_page=20`);
            
            issuesContainer.innerHTML = issues.length > 0 
                ? issues.map(issue => this.renderIssue(issue)).join('')
                : '<div class="no-issues">No open issues found</div>';
        } catch (error) {
            console.error("Failed to load issues:", error);
            issuesContainer.innerHTML = `<div class="error">Failed to load issues: ${error.message}</div>`;
        }
    }

    private renderIssue(issue: GitHubIssue): string {
        return `
            <div class="issue-item" data-issue-id="${issue.id}" data-issue-number="${issue.number}">
                <div class="issue-header">
                    <span class="issue-number">#${issue.number}</span>
                    <span class="issue-title">${issue.title}</span>
                </div>
                <div class="issue-meta">
                    <span class="issue-state ${issue.state}">${issue.state}</span>
                    <span class="issue-date">${new Date(issue.updated_at).toLocaleDateString()}</span>
                </div>
                <div class="issue-actions">
                    <button class="b3-button b3-button--small" data-action="view">View</button>
                    <button class="b3-button b3-button--small" data-action="import">Import</button>
                </div>
            </div>
        `;
    }

    private bindGitHubPanelEvents(element: HTMLElement) {
        // リポジトリ選択
        element.querySelector("#repoSelect")?.addEventListener("change", async (e) => {
            const target = e.target as HTMLSelectElement;
            this.selectedRepo = target.value;
            
            if (this.selectedRepo) {
                await this.loadIssues(element, this.selectedRepo);
                // 設定を保存
                this.saveData(STORAGE_NAME, { ...this.data[STORAGE_NAME], selectedRepo: this.selectedRepo });
            }
        });

        // リフレッシュ
        element.querySelector("#refreshGitHub")?.addEventListener("click", () => {
            this.loadRepositories(element);
        });

        // Issue作成
        element.querySelector("#createIssue")?.addEventListener("click", () => {
            this.createIssueFromDocument();
        });

        // Issueアクション
        element.addEventListener("click", async (e) => {
            const target = e.target as HTMLElement;
            const action = target.getAttribute("data-action");
            const issueItem = target.closest(".issue-item");
            
            if (!action || !issueItem) return;

            const issueNumber = issueItem.getAttribute("data-issue-number");
            
            switch (action) {
                case "view":
                    this.viewIssue(issueNumber);
                    break;
                case "import":
                    await this.importIssue(issueNumber);
                    break;
            }
        });

        // クイックアクション
        element.querySelector("#syncDoc")?.addEventListener("click", () => {
            this.syncCurrentDocumentToGitHub();
        });

        element.querySelector("#importReadme")?.addEventListener("click", () => {
            this.importReadme();
        });

        element.querySelector("#exportNotes")?.addEventListener("click", () => {
            this.exportNotesToRepo();
        });
    }

    private async fetchGitHubAPI<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
        const response = await fetch(`https://api.github.com${endpoint}`, {
            ...options,
            headers: {
                "Authorization": `token ${this.githubToken}`,
                "Accept": "application/vnd.github.v3+json",
                "Content-Type": "application/json",
                ...options.headers
            }
        });

        if (!response.ok) {
            throw new Error(`GitHub API error: ${response.statusText}`);
        }

        return response.json();
    }

    private async createIssueFromDocument() {
        if (!this.selectedRepo) {
            showMessage("Please select a repository first", 2000, "error");
            return;
        }

        const currentDoc = await this.getCurrentDocumentContent();
        if (!currentDoc) {
            showMessage("No document selected", 2000, "error");
            return;
        }

        const dialog = new Dialog({
            title: "Create GitHub Issue",
            content: `
                <div class="create-issue-dialog">
                    <div class="form-group">
                        <label>Issue Title:</label>
                        <input type="text" class="b3-text-field" id="issueTitle" value="${currentDoc.name}">
                    </div>
                    <div class="form-group">
                        <label>Issue Body:</label>
                        <textarea class="b3-text-field" id="issueBody" rows="10">${currentDoc.content}</textarea>
                    </div>
                    <div class="form-group">
                        <label>Labels (comma-separated):</label>
                        <input type="text" class="b3-text-field" id="issueLabels" placeholder="bug, enhancement">
                    </div>
                    <div class="form-actions">
                        <button class="b3-button b3-button--cancel" id="cancelIssue">Cancel</button>
                        <button class="b3-button" id="createIssueBtn">Create Issue</button>
                    </div>
                </div>
            `,
            width: 600,
            height: 500
        });

        dialog.element.querySelector("#createIssueBtn")?.addEventListener("click", async () => {
            await this.submitIssue(dialog);
        });

        dialog.element.querySelector("#cancelIssue")?.addEventListener("click", () => {
            dialog.destroy();
        });
    }

    private async submitIssue(dialog: Dialog) {
        const title = (dialog.element.querySelector("#issueTitle") as HTMLInputElement).value;
        const body = (dialog.element.querySelector("#issueBody") as HTMLTextAreaElement).value;
        const labelsInput = (dialog.element.querySelector("#issueLabels") as HTMLInputElement).value;
        
        const labels = labelsInput.split(",").map(l => l.trim()).filter(l => l);

        try {
            const issue = await this.fetchGitHubAPI(`/repos/${this.selectedRepo}/issues`, {
                method: "POST",
                body: JSON.stringify({
                    title,
                    body,
                    labels
                })
            });

            showMessage("Issue created successfully!", 2000, "info");
            dialog.destroy();
            
            // パネルを更新
            const githubPanel = document.querySelector('[data-type="github-panel"]');
            if (githubPanel) {
                await this.loadIssues(githubPanel as HTMLElement, this.selectedRepo);
            }
        } catch (error) {
            console.error("Failed to create issue:", error);
            showMessage("Failed to create issue: " + error.message, 3000, "error");
        }
    }

    private async importIssue(issueNumber: string) {
        if (!this.selectedRepo || !issueNumber) return;

        try {
            const issue = await this.fetchGitHubAPI<GitHubIssue>(`/repos/${this.selectedRepo}/issues/${issueNumber}`);
            
            // Issueをドキュメントとして作成
            const markdown = `# GitHub Issue #${issue.number}: ${issue.title}

**Repository:** ${this.selectedRepo}
**State:** ${issue.state}
**Created:** ${new Date(issue.created_at).toLocaleDateString()}
**Updated:** ${new Date(issue.updated_at).toLocaleDateString()}
**URL:** ${issue.html_url}

---

${issue.body || "No description provided."}
`;

            const notebooks = await this.apiRequest("/notebook/lsNotebooks", {});
            const defaultNotebook = notebooks.data.notebooks[0]?.id;

            if (defaultNotebook) {
                const docResponse = await this.apiRequest("/filetree/createDocWithMd", {
                    notebook: defaultNotebook,
                    path: `/GitHub-Issue-${issue.number}-${issue.title}`,
                    markdown: markdown
                });

                if (docResponse.code === 0) {
                    showMessage("Issue imported successfully!", 2000, "info");
                    
                    // 作成したドキュメントを開く
                    openTab({
                        app: this.app,
                        doc: {
                            id: docResponse.data,
                            action: ["cb-get-focus"]
                        }
                    });
                } else {
                    throw new Error(docResponse.msg);
                }
            }
        } catch (error) {
            console.error("Failed to import issue:", error);
            showMessage("Failed to import issue: " + error.message, 3000, "error");
        }
    }

    private viewIssue(issueNumber: string) {
        if (!this.selectedRepo || !issueNumber) return;
        
        const url = `https://github.com/${this.selectedRepo}/issues/${issueNumber}`;
        window.open(url, "_blank");
    }

    private async syncCurrentDocumentToGitHub() {
        if (!this.selectedRepo) {
            showMessage("Please select a repository first", 2000, "error");
            return;
        }

        const currentDoc = await this.getCurrentDocumentContent();
        if (!currentDoc) {
            showMessage("No document selected", 2000, "error");
            return;
        }

        try {
            // ファイルをリポジトリにコミット
            const fileName = `${currentDoc.name.replace(/[^a-zA-Z0-9]/g, "-")}.md`;
            
            const response = await this.fetchGitHubAPI(`/repos/${this.selectedRepo}/contents/${fileName}`, {
                method: "PUT",
                body: JSON.stringify({
                    message: `Update ${fileName} from SiYuan`,
                    content: btoa(unescape(encodeURIComponent(currentDoc.content))),
                    branch: "main"
                })
            });

            showMessage("Document synced to GitHub successfully!", 2000, "info");
        } catch (error) {
            console.error("Failed to sync document:", error);
            showMessage("Failed to sync document: " + error.message, 3000, "error");
        }
    }

    private async importReadme() {
        if (!this.selectedRepo) {
            showMessage("Please select a repository first", 2000, "error");
            return;
        }

        try {
            const readme = await this.fetchGitHubAPI(`/repos/${this.selectedRepo}/readme`);
            const content = atob(readme.content);
            
            const notebooks = await this.apiRequest("/notebook/lsNotebooks", {});
            const defaultNotebook = notebooks.data.notebooks[0]?.id;

            if (defaultNotebook) {
                const docResponse = await this.apiRequest("/filetree/createDocWithMd", {
                    notebook: defaultNotebook,
                    path: `/README-${this.selectedRepo.replace("/", "-")}`,
                    markdown: content
                });

                if (docResponse.code === 0) {
                    showMessage("README imported successfully!", 2000, "info");
                    
                    openTab({
                        app: this.app,
                        doc: {
                            id: docResponse.data,
                            action: ["cb-get-focus"]
                        }
                    });
                }
            }
        } catch (error) {
            console.error("Failed to import README:", error);
            showMessage("Failed to import README: " + error.message, 3000, "error");
        }
    }

    private async exportNotesToRepo() {
        if (!this.selectedRepo) {
            showMessage("Please select a repository first", 2000, "error");
            return;
        }

        const dialog = new Dialog({
            title: "Export Notes to GitHub",
            content: `
                <div class="export-dialog">
                    <p>This will export all documents from the selected notebook to your GitHub repository.</p>
                    <div class="form-group">
                        <label>Select Notebook:</label>
                        <select class="b3-select" id="notebookSelect">
                            <option value="">Loading notebooks...</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>Target Folder in Repository:</label>
                        <input type="text" class="b3-text-field" id="targetFolder" value="notes" placeholder="notes">
                    </div>
                    <div class="form-actions">
                        <button class="b3-button b3-button--cancel" id="cancelExport">Cancel</button>
                        <button class="b3-button" id="startExport">Start Export</button>
                    </div>
                </div>
            `,
            width: 500,
            height: 300
        });

        // ノートブック一覧を読み込み
        const notebooks = await this.apiRequest("/notebook/lsNotebooks", {});
        const notebookSelect = dialog.element.querySelector("#notebookSelect") as HTMLSelectElement;
        
        notebookSelect.innerHTML = "";
        notebooks.data.notebooks.forEach((notebook: any) => {
            const option = document.createElement("option");
            option.value = notebook.id;
            option.textContent = notebook.name;
            notebookSelect.appendChild(option);
        });

        dialog.element.querySelector("#startExport")?.addEventListener("click", async () => {
            await this.performNotesExport(dialog);
        });

        dialog.element.querySelector("#cancelExport")?.addEventListener("click", () => {
            dialog.destroy();
        });
    }

    private async performNotesExport(dialog: Dialog) {
        const notebookId = (dialog.element.querySelector("#notebookSelect") as HTMLSelectElement).value;
        const targetFolder = (dialog.element.querySelector("#targetFolder") as HTMLInputElement).value || "notes";
        
        if (!notebookId) {
            showMessage("Please select a notebook", 2000, "error");
            return;
        }

        try {
            // ノートブックのすべてのドキュメントを取得
            const docs = await this.apiRequest("/search/fullTextSearchBlock", {
                query: `box:"${notebookId}"`,
                types: { document: true }
            });

            const exportButton = dialog.element.querySelector("#startExport") as HTMLButtonElement;
            exportButton.disabled = true;
            exportButton.textContent = "Exporting...";

            let exported = 0;
            for (const doc of docs.data.blocks) {
                try {
                    const kramdown = await this.apiRequest("/block/getBlockKramdown", {
                        id: doc.id
                    });

                    const fileName = `${doc.name.replace(/[^a-zA-Z0-9]/g, "-")}.md`;
                    const filePath = `${targetFolder}/${fileName}`;

                    await this.fetchGitHubAPI(`/repos/${this.selectedRepo}/contents/${filePath}`, {
                        method: "PUT",
                        body: JSON.stringify({
                            message: `Export ${doc.name} from SiYuan`,
                            content: btoa(unescape(encodeURIComponent(kramdown.data.kramdown))),
                            branch: "main"
                        })
                    });

                    exported++;
                } catch (error) {
                    console.warn(`Failed to export document ${doc.name}:`, error);
                }
            }

            showMessage(`Successfully exported ${exported} documents!`, 3000, "info");
            dialog.destroy();
        } catch (error) {
            console.error("Export failed:", error);
            showMessage("Export failed: " + error.message, 3000, "error");
        }
    }

    // ヘルパーメソッド
    private async getCurrentDocumentContent(): Promise<{content: string, name: string, id: string} | null> {
        const activeTab = document.querySelector('.layout__tab--active');
        if (!activeTab) return null;

        const docId = activeTab.getAttribute('data-id');
        if (!docId) return null;

        try {
            const response = await this.apiRequest("/block/getBlockKramdown", {
                id: docId
            });

            if (response.code === 0) {
                return {
                    content: response.data.kramdown,
                    name: activeTab.textContent?.trim() || "Untitled",
                    id: docId
                };
            }
        } catch (error) {
            console.error("Failed to get document content:", error);
        }

        return null;
    }
}
```

これらの高度なプラグインパターンは以下の特徴を持ちます：

### AI統合プラグイン
- OpenAI ChatGPTとの連携
- 自動要約、アウトライン生成
- テキスト分析機能
- 会話履歴管理
- SiYuan内蔵AI機能の活用

### GitHub統合プラグイン  
- GitHub APIとの完全統合
- Issue管理とインポート/エクスポート
- リポジトリブラウジング
- 自動ドキュメント同期
- README インポート機能

これらのパターンを参考に、様々な外部サービスとの連携プラグインを開発できます。
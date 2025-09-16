# SiYuan Plugin Developer API Guide

A comprehensive guide covering API endpoints, implementation patterns, and best practices needed for developing SiYuan plugins.

**Target Audience**: Developers creating SiYuan plugins (beginner to advanced)

**Prerequisites**: 
- Basic knowledge of JavaScript/TypeScript
- Basic experience using SiYuan
- Understanding of plugin development concepts

**Related Documentation**:
- [Plugin Best Practices](PLUGIN_BEST_PRACTICES.md) - Development methods and code quality
- [Plugin Showcase](PLUGIN_SHOWCASE.md) - Examples of excellent plugins
- [Complete API Reference](API.md) - All API endpoints
- [Beginner API Guide](API_BEGINNERS.md) - Basic API usage

---

## 📋 Table of Contents

### 🚀 Getting Started
- [Development Environment Setup](#development-environment-setup)
- [Creating Plugin Projects](#creating-plugin-projects)
- [Basic Configuration and API Initialization](#basic-configuration-and-api-initialization)

### 🔧 Core Feature Development
- [Block Operations](#block-operations)
- [Document Operations](#document-operations)
- [Notebook Operations](#notebook-operations)
- [Database Operations (SQL)](#database-operations-sql)
- [Search Feature Implementation](#search-feature-implementation)

### 🎨 UI & Interaction
- [UI Updates and Synchronization](#ui-updates-and-synchronization)
- [Custom Dialogs and Panels](#custom-dialogs-and-panels)
- [Context Menu Extensions](#context-menu-extensions)
- [Keyboard Shortcuts](#keyboard-shortcuts)

### 📁 File & Resource Management
- [File and Asset Management](#file-and-asset-management)
- [Settings Storage and Management](#settings-storage-and-management)
- [Internationalization (i18n)](#internationalization-i18n)

### 🔗 Plugin Integration
- [Inter-Plugin Communication](#inter-plugin-communication)
- [Event System Usage](#event-system-usage)
- [External Service Integration](#external-service-integration)

### 📊 Practical Development Examples
- [Practical Plugin Examples](#practical-plugin-examples)
- [Performance Optimization](#performance-optimization)
- [Debugging and Testing](#debugging-and-testing)
- [Distribution and Updates](#distribution-and-updates)

---

## 🚀 Development Environment Setup

### Tools Required for Plugin Development

```bash
# Install Node.js and npm/pnpm
node --version  # v16+ recommended
npm --version   # or pnpm --version

# Install TypeScript globally
npm install -g typescript

# Recommended development tools
npm install -g @typescript-eslint/eslint-plugin prettier
```

### Setting Up SiYuan Plugin Development Environment

```bash
# Clone plugin template
git clone https://github.com/siyuan-note/plugin-sample-vite-svelte my-plugin
cd my-plugin

# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

### Project Structure

```
my-plugin/
├── src/
│   ├── index.ts          # Plugin entry point
│   ├── api/              # API wrapper functions
│   ├── components/       # UI components
│   └── utils/            # Utility functions
├── public/
│   └── i18n/            # Translation files
├── plugin.json          # Plugin manifest
├── preview.png          # Plugin preview image
└── README.md            # Plugin documentation
```

---

## Creating Plugin Projects

### Plugin Manifest (plugin.json)

```json
{
  "name": "my-awesome-plugin",
  "author": "Your Name",
  "url": "https://github.com/yourusername/my-awesome-plugin",
  "version": "1.0.0",
  "minAppVersion": "3.0.0",
  "backends": ["all"],
  "frontends": ["all"],
  "displayName": {
    "en_US": "My Awesome Plugin",
    "zh_CN": "我的超棒插件",
    "ja_JP": "私の素晴らしいプラグイン"
  },
  "description": {
    "en_US": "An awesome plugin that does amazing things",
    "zh_CN": "一个做奇妙事情的超棒插件",
    "ja_JP": "素晴らしいことをする素晴らしいプラグイン"
  },
  "readme": {
    "en_US": "README.md",
    "zh_CN": "README_zh_CN.md",
    "ja_JP": "README_ja_JP.md"
  },
  "keywords": ["productivity", "automation", "utility"],
  "funding": {
    "openCollective": "",
    "patreon": "",
    "github": "",
    "custom": ["https://your-donation-link.com"]
  }
}
```

### Plugin Entry Point (index.ts)

```typescript
import {
    Plugin,
    showMessage,
    confirm,
    Dialog,
    Menu,
    openTab,
    adaptHotkey
} from "siyuan";
import "./index.scss";

export default class MyAwesomePlugin extends Plugin {
    private isMobile: boolean;

    onload() {
        this.data[STORAGE_NAME] = { readonlyText: "Readonly" };

        const frontEnd = getFrontend();
        this.isMobile = frontEnd === "mobile" || frontEnd === "browser-mobile";

        // Add plugin icon to toolbar
        this.addIcons(`<symbol id="iconMyPlugin" viewBox="0 0 32 32">
            <path d="your-svg-path-here"/>
        </symbol>`);

        // Create toolbar button
        this.addTopBar({
            icon: "iconMyPlugin",
            title: this.i18n.addTopBarIcon,
            position: "right",
            callback: () => {
                this.openPluginPanel();
            }
        });

        // Add menu items
        this.addMenu();

        // Register keyboard shortcuts
        this.addCommand({
            langKey: "showPluginPanel",
            hotkey: "⌘⇧P",
            callback: () => {
                this.openPluginPanel();
            }
        });

        // Listen to events
        this.eventBus.on("ws-main", this.onWebSocketMessage.bind(this));
    }

    onLayoutReady() {
        // Plugin initialization after layout is ready
        this.loadSettings();
        this.initializeUI();
    }

    onunload() {
        // Cleanup when plugin is unloaded
        this.saveSettings();
        this.cleanupResources();
    }

    private addMenu() {
        this.addMenu({
            label: "My Plugin",
            icon: "iconMyPlugin",
            callback: () => {
                this.showContextMenu();
            }
        });
    }

    private async openPluginPanel() {
        const dialog = new Dialog({
            title: this.i18n.pluginTitle,
            content: `<div id="myPluginPanel">${this.generatePanelHTML()}</div>`,
            width: this.isMobile ? "92vw" : "520px",
            height: this.isMobile ? "92vh" : "540px",
        });

        this.initializePanelEvents(dialog);
    }
}
```

---

## Basic Configuration and API Initialization

### API Client Setup

```typescript
class SiYuanAPI {
    private baseURL: string;
    private token: string;

    constructor() {
        this.baseURL = '/api';
        // Token is automatically handled by the plugin framework
    }

    async request(endpoint: string, data: any = {}): Promise<any> {
        try {
            const response = await fetch(`${this.baseURL}${endpoint}`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(data)
            });

            const result = await response.json();
            
            if (result.code !== 0) {
                throw new Error(`API Error: ${result.msg}`);
            }

            return result.data;
        } catch (error) {
            console.error('API request failed:', error);
            throw error;
        }
    }
}

// Create global API instance
const api = new SiYuanAPI();
```

### Configuration Management

```typescript
interface PluginConfig {
    enabled: boolean;
    autoSave: boolean;
    theme: 'light' | 'dark' | 'auto';
    customSettings: Record<string, any>;
}

class ConfigManager {
    private config: PluginConfig;
    private readonly STORAGE_KEY = 'myPlugin_config';

    constructor() {
        this.loadConfig();
    }

    private loadConfig(): void {
        const stored = localStorage.getItem(this.STORAGE_KEY);
        this.config = stored ? JSON.parse(stored) : this.getDefaultConfig();
    }

    private getDefaultConfig(): PluginConfig {
        return {
            enabled: true,
            autoSave: true,
            theme: 'auto',
            customSettings: {}
        };
    }

    get<K extends keyof PluginConfig>(key: K): PluginConfig[K] {
        return this.config[key];
    }

    set<K extends keyof PluginConfig>(key: K, value: PluginConfig[K]): void {
        this.config[key] = value;
        this.saveConfig();
    }

    private saveConfig(): void {
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(this.config));
    }
}
```

---

## Block Operations

### Creating and Inserting Blocks

```typescript
class BlockManager {
    async insertBlock(
        parentID: string, 
        content: string, 
        position: 'before' | 'after' | 'child' = 'child'
    ): Promise<string> {
        const data = {
            dataType: 'markdown',
            data: content,
            parentID: parentID
        };

        if (position !== 'child') {
            data['previousID'] = position === 'after' ? parentID : '';
        }

        const result = await api.request('/block/insertBlock', data);
        return result[0].doOperations[0].id;
    }

    async updateBlock(blockId: string, content: string): Promise<void> {
        await api.request('/block/updateBlock', {
            dataType: 'markdown',
            data: content,
            id: blockId
        });
    }

    async deleteBlock(blockId: string): Promise<void> {
        await api.request('/block/deleteBlock', {
            id: blockId
        });
    }

    async moveBlock(
        blockId: string, 
        targetParentId: string, 
        position?: string
    ): Promise<void> {
        const data: any = {
            id: blockId,
            parentID: targetParentId
        };

        if (position) {
            data.previousID = position;
        }

        await api.request('/block/moveBlock', data);
    }
}

// Usage example
const blockManager = new BlockManager();

// Create a new paragraph block
const newBlockId = await blockManager.insertBlock(
    parentBlockId,
    '# New Heading\n\nThis is a new paragraph.',
    'after'
);

// Update the block content
await blockManager.updateBlock(newBlockId, '# Updated Heading\n\nUpdated content.');
```

### Advanced Block Queries

```typescript
class AdvancedBlockQueries {
    async findBlocksByType(type: string, notebook?: string): Promise<Block[]> {
        let sql = `SELECT * FROM blocks WHERE type = '${type}'`;
        
        if (notebook) {
            sql += ` AND box = '${notebook}'`;
        }
        
        sql += ' ORDER BY created DESC';

        const result = await api.request('/query/sql', { stmt: sql });
        return result.map(this.mapRowToBlock);
    }

    async findBlocksWithAttributes(
        attribute: string, 
        value?: string
    ): Promise<Block[]> {
        let sql = `
            SELECT b.*, ba.value as attr_value
            FROM blocks b
            JOIN block_attrs ba ON b.id = ba.block_id
            WHERE ba.name = '${attribute}'
        `;

        if (value) {
            sql += ` AND ba.value = '${value}'`;
        }

        const result = await api.request('/query/sql', { stmt: sql });
        return result.map(this.mapRowToBlock);
    }

    async findReferencedBlocks(blockId: string): Promise<Block[]> {
        const sql = `
            SELECT b.* 
            FROM blocks b
            JOIN refs r ON b.id = r.def_block_id
            WHERE r.block_id = '${blockId}'
        `;

        const result = await api.request('/query/sql', { stmt: sql });
        return result.map(this.mapRowToBlock);
    }

    private mapRowToBlock(row: any): Block {
        return {
            id: row.id,
            content: row.content,
            type: row.type,
            subType: row.subtype,
            parentId: row.parent_id,
            rootId: row.root_id,
            created: new Date(row.created),
            updated: new Date(row.updated),
            attributes: row.attrs ? JSON.parse(row.attrs) : {}
        };
    }
}
```

---

## Document Operations

### Document Creation and Management

```typescript
class DocumentManager {
    async createDocument(
        notebookId: string,
        path: string,
        content: string
    ): Promise<string> {
        const result = await api.request('/filetree/createDocWithMd', {
            notebook: notebookId,
            path: path,
            markdown: content
        });
        
        return result.id;
    }

    async getDocumentContent(documentId: string): Promise<string> {
        const result = await api.request('/export/exportMdContent', {
            id: documentId
        });
        
        return result.content;
    }

    async renameDocument(
        documentId: string,
        newTitle: string
    ): Promise<void> {
        await api.request('/filetree/renameDoc', {
            id: documentId,
            title: newTitle
        });
    }

    async moveDocument(
        documentId: string,
        targetNotebook: string,
        targetPath: string
    ): Promise<void> {
        await api.request('/filetree/moveDoc', {
            id: documentId,
            toNotebook: targetNotebook,
            toPath: targetPath
        });
    }

    async deleteDocument(documentId: string): Promise<void> {
        await api.request('/filetree/removeDoc', {
            id: documentId
        });
    }
}

// Document template system
class DocumentTemplateManager {
    private templates = new Map<string, DocumentTemplate>();

    registerTemplate(name: string, template: DocumentTemplate): void {
        this.templates.set(name, template);
    }

    async createFromTemplate(
        templateName: string,
        notebookId: string,
        path: string,
        variables: Record<string, string> = {}
    ): Promise<string> {
        const template = this.templates.get(templateName);
        if (!template) {
            throw new Error(`Template '${templateName}' not found`);
        }

        const content = this.processTemplate(template.content, variables);
        
        return await new DocumentManager().createDocument(
            notebookId,
            path,
            content
        );
    }

    private processTemplate(
        content: string, 
        variables: Record<string, string>
    ): string {
        let processed = content;
        
        // Replace variables
        for (const [key, value] of Object.entries(variables)) {
            const regex = new RegExp(`\\{\\{${key}\\}\\}`, 'g');
            processed = processed.replace(regex, value);
        }

        // Replace built-in variables
        const now = new Date();
        processed = processed.replace(/\{\{date\}\}/g, now.toISOString().split('T')[0]);
        processed = processed.replace(/\{\{datetime\}\}/g, now.toISOString());
        processed = processed.replace(/\{\{time\}\}/g, now.toTimeString().split(' ')[0]);

        return processed;
    }
}

interface DocumentTemplate {
    name: string;
    content: string;
    variables: string[];
}

// Usage example
const templateManager = new DocumentTemplateManager();

templateManager.registerTemplate('meeting-notes', {
    name: 'Meeting Notes',
    content: `# {{meeting_title}}

**Date:** {{date}}  
**Attendees:** {{attendees}}  
**Duration:** {{duration}}

## Agenda

## Notes

## Action Items
- [ ] 

## Next Steps
`,
    variables: ['meeting_title', 'attendees', 'duration']
});

// Create a meeting note from template
const documentId = await templateManager.createFromTemplate(
    'meeting-notes',
    notebookId,
    '/Meetings/Weekly Standup 2024-01-15',
    {
        meeting_title: 'Weekly Standup',
        attendees: 'Alice, Bob, Charlie',
        duration: '30 minutes'
    }
);
```

---

## Notebook Operations

### Notebook Management

```typescript
class NotebookManager {
    async listNotebooks(): Promise<Notebook[]> {
        const result = await api.request('/notebook/lsNotebooks', {});
        return result.notebooks;
    }

    async createNotebook(name: string, icon?: string): Promise<string> {
        const data: any = { name };
        if (icon) data.icon = icon;
        
        const result = await api.request('/notebook/createNotebook', data);
        return result.id;
    }

    async openNotebook(notebookId: string): Promise<void> {
        await api.request('/notebook/openNotebook', {
            notebook: notebookId
        });
    }

    async closeNotebook(notebookId: string): Promise<void> {
        await api.request('/notebook/closeNotebook', {
            notebook: notebookId
        });
    }

    async renameNotebook(notebookId: string, newName: string): Promise<void> {
        await api.request('/notebook/renameNotebook', {
            notebook: notebookId,
            name: newName
        });
    }

    async getNotebookConfig(notebookId: string): Promise<NotebookConfig> {
        const result = await api.request('/notebook/getNotebookConf', {
            notebook: notebookId
        });
        return result.conf;
    }

    async setNotebookConfig(
        notebookId: string, 
        config: Partial<NotebookConfig>
    ): Promise<void> {
        await api.request('/notebook/setNotebookConf', {
            notebook: notebookId,
            conf: config
        });
    }
}

interface NotebookConfig {
    name: string;
    closed: boolean;
    refCreateSavePath: string;
    createDocNameTemplate: string;
    dailyNoteSavePath: string;
    dailyNoteTemplatePath: string;
}

// Advanced notebook utilities
class NotebookUtilities {
    async getNotebookStatistics(notebookId: string): Promise<NotebookStats> {
        const sql = `
            SELECT 
                COUNT(*) as total_blocks,
                COUNT(CASE WHEN type = 'd' THEN 1 END) as documents,
                COUNT(CASE WHEN type = 'p' THEN 1 END) as paragraphs,
                COUNT(CASE WHEN type = 'h' THEN 1 END) as headings,
                SUM(LENGTH(content)) as total_content_length,
                MIN(created) as oldest_content,
                MAX(updated) as newest_content
            FROM blocks 
            WHERE box = '${notebookId}'
        `;

        const result = await api.request('/query/sql', { stmt: sql });
        return result[0] as NotebookStats;
    }

    async exportNotebook(
        notebookId: string,
        format: 'markdown' | 'html' | 'pdf' = 'markdown'
    ): Promise<string> {
        // Implementation depends on desired export format
        const documents = await this.getNotebookDocuments(notebookId);
        
        let exportedContent = '';
        
        for (const doc of documents) {
            const content = await new DocumentManager().getDocumentContent(doc.id);
            exportedContent += `\n\n# ${doc.title}\n\n${content}`;
        }
        
        return exportedContent;
    }

    private async getNotebookDocuments(notebookId: string): Promise<DocumentInfo[]> {
        const sql = `
            SELECT id, content as title, path
            FROM blocks 
            WHERE box = '${notebookId}' AND type = 'd'
            ORDER BY path
        `;
        
        return await api.request('/query/sql', { stmt: sql });
    }
}
```

---

## Database Operations (SQL)

### Advanced SQL Queries for Plugin Development

```typescript
class AdvancedSQLQueries {
    async executeQuery(sql: string, params?: any[]): Promise<any[]> {
        const data: any = { stmt: sql };
        if (params && params.length > 0) {
            // Note: SiYuan API doesn't support parameterized queries directly
            // Manual sanitization is required
            data.stmt = this.sanitizeSQL(sql, params);
        }

        return await api.request('/query/sql', data);
    }

    // Content analysis queries
    async getContentStatistics(): Promise<ContentStats> {
        const sql = `
            SELECT 
                type,
                COUNT(*) as count,
                AVG(LENGTH(content)) as avg_length,
                SUM(LENGTH(content)) as total_length,
                MIN(created) as oldest,
                MAX(updated) as newest
            FROM blocks 
            WHERE type != 'd'  -- Exclude document blocks
            GROUP BY type
            ORDER BY count DESC
        `;

        const result = await this.executeQuery(sql);
        return this.processContentStats(result);
    }

    async findOrphanedBlocks(): Promise<Block[]> {
        const sql = `
            SELECT b.*
            FROM blocks b
            LEFT JOIN blocks parent ON b.parent_id = parent.id
            WHERE b.parent_id != '' 
            AND parent.id IS NULL
            AND b.type != 'd'
        `;

        return await this.executeQuery(sql);
    }

    async findDuplicateContent(threshold: number = 0.8): Promise<DuplicateGroup[]> {
        const sql = `
            SELECT 
                b1.id as id1,
                b2.id as id2,
                b1.content,
                LENGTH(b1.content) as content_length
            FROM blocks b1
            JOIN blocks b2 ON b1.id < b2.id
            WHERE b1.type = b2.type
            AND b1.type IN ('p', 'h')
            AND LENGTH(b1.content) > 10
            AND (
                b1.content = b2.content OR
                (LENGTH(b1.content) > 50 AND SIMILARITY(b1.content, b2.content) > ${threshold})
            )
            ORDER BY content_length DESC
        `;

        const result = await this.executeQuery(sql);
        return this.processDuplicateGroups(result);
    }

    async getBacklinkAnalysis(blockId: string): Promise<BacklinkAnalysis> {
        const sql = `
            WITH RECURSIVE link_chain AS (
                -- Direct backlinks
                SELECT 
                    r.block_id as source_id,
                    r.def_block_id as target_id,
                    b.content as source_content,
                    1 as depth,
                    r.block_id as original_source
                FROM refs r
                JOIN blocks b ON r.block_id = b.id
                WHERE r.def_block_id = '${blockId}'
                
                UNION ALL
                
                -- Indirect backlinks (up to 3 levels)
                SELECT 
                    r.block_id as source_id,
                    lc.target_id,
                    b.content as source_content,
                    lc.depth + 1,
                    lc.original_source
                FROM refs r
                JOIN link_chain lc ON r.def_block_id = lc.source_id
                JOIN blocks b ON r.block_id = b.id
                WHERE lc.depth < 3
            )
            SELECT DISTINCT 
                source_id,
                source_content,
                depth,
                COUNT(*) OVER (PARTITION BY source_id) as connection_strength
            FROM link_chain
            ORDER BY depth, connection_strength DESC
        `;

        const result = await this.executeQuery(sql);
        return this.processBacklinkAnalysis(result);
    }

    private sanitizeSQL(sql: string, params: any[]): string {
        let sanitized = sql;
        params.forEach((param, index) => {
            const placeholder = `$${index + 1}`;
            const sanitizedParam = this.sanitizeParam(param);
            sanitized = sanitized.replace(placeholder, sanitizedParam);
        });
        return sanitized;
    }

    private sanitizeParam(param: any): string {
        if (typeof param === 'string') {
            return `'${param.replace(/'/g, "''")}'`;
        }
        return String(param);
    }
}

// Advanced data analysis
class DataAnalyzer {
    async analyzeWritingPatterns(): Promise<WritingPattern[]> {
        const sql = `
            SELECT 
                DATE(datetime(created/1000, 'unixepoch')) as date,
                COUNT(*) as blocks_created,
                SUM(LENGTH(content)) as chars_written,
                AVG(LENGTH(content)) as avg_block_length
            FROM blocks 
            WHERE type IN ('p', 'h', 'l', 'b')
            AND created > (strftime('%s', 'now', '-30 days') * 1000)
            GROUP BY DATE(datetime(created/1000, 'unixepoch'))
            ORDER BY date DESC
        `;

        const result = await new AdvancedSQLQueries().executeQuery(sql);
        return result.map(row => ({
            date: row.date,
            blocksCreated: row.blocks_created,
            charsWritten: row.chars_written,
            avgBlockLength: row.avg_block_length
        }));
    }

    async findContentGaps(): Promise<ContentGap[]> {
        // Find topics mentioned but not expanded upon
        const sql = `
            SELECT 
                LOWER(TRIM(tag.value)) as topic,
                COUNT(*) as mentions,
                AVG(LENGTH(b.content)) as avg_content_length,
                COUNT(CASE WHEN LENGTH(b.content) > 200 THEN 1 END) as detailed_mentions
            FROM blocks b
            JOIN block_attrs tag ON b.id = tag.block_id
            WHERE tag.name = 'custom-tag'
            GROUP BY LOWER(TRIM(tag.value))
            HAVING mentions > 2 AND detailed_mentions < mentions * 0.3
            ORDER BY mentions DESC, avg_content_length ASC
        `;

        return await new AdvancedSQLQueries().executeQuery(sql);
    }
}
```

---

## UI Updates and Synchronization

### Real-time UI Updates

```typescript
class UIManager {
    private observers = new Map<string, MutationObserver>();
    private updateQueue: UIUpdate[] = [];
    private isProcessingQueue = false;

    async updateBlockInUI(blockId: string, newContent: string): Promise<void> {
        const blockElement = document.querySelector(`[data-node-id="${blockId}"]`);
        if (!blockElement) {
            console.warn(`Block element not found: ${blockId}`);
            return;
        }

        // Queue the update to avoid conflicts
        this.queueUpdate({
            type: 'block-content',
            elementId: blockId,
            content: newContent,
            element: blockElement as HTMLElement
        });
    }

    async refreshNotebookTree(): Promise<void> {
        // Trigger SiYuan's internal notebook tree refresh
        const event = new CustomEvent('siyuan-refresh-tree');
        document.dispatchEvent(event);
    }

    observeBlockChanges(
        blockId: string, 
        callback: (mutations: MutationRecord[]) => void
    ): void {
        const blockElement = document.querySelector(`[data-node-id="${blockId}"]`);
        if (!blockElement) return;

        const observer = new MutationObserver(callback);
        observer.observe(blockElement, {
            childList: true,
            subtree: true,
            characterData: true,
            attributes: true
        });

        this.observers.set(blockId, observer);
    }

    stopObservingBlock(blockId: string): void {
        const observer = this.observers.get(blockId);
        if (observer) {
            observer.disconnect();
            this.observers.delete(blockId);
        }
    }

    private queueUpdate(update: UIUpdate): void {
        this.updateQueue.push(update);
        
        if (!this.isProcessingQueue) {
            this.processUpdateQueue();
        }
    }

    private async processUpdateQueue(): Promise<void> {
        this.isProcessingQueue = true;

        while (this.updateQueue.length > 0) {
            const update = this.updateQueue.shift()!;
            
            try {
                await this.applyUpdate(update);
            } catch (error) {
                console.error('Failed to apply UI update:', error);
            }
            
            // Small delay to prevent UI blocking
            await this.sleep(10);
        }

        this.isProcessingQueue = false;
    }

    private async applyUpdate(update: UIUpdate): Promise<void> {
        switch (update.type) {
            case 'block-content':
                await this.updateBlockContent(update);
                break;
            case 'add-class':
                update.element.classList.add(update.className!);
                break;
            case 'remove-class':
                update.element.classList.remove(update.className!);
                break;
            case 'set-attribute':
                update.element.setAttribute(update.attributeName!, update.attributeValue!);
                break;
        }
    }

    private sleep(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

interface UIUpdate {
    type: 'block-content' | 'add-class' | 'remove-class' | 'set-attribute';
    elementId: string;
    element: HTMLElement;
    content?: string;
    className?: string;
    attributeName?: string;
    attributeValue?: string;
}

// Theme and styling management
class ThemeManager {
    async applyCustomCSS(css: string, id: string = 'plugin-custom-css'): Promise<void> {
        // Remove existing custom CSS
        const existing = document.getElementById(id);
        if (existing) {
            existing.remove();
        }

        // Add new custom CSS
        const style = document.createElement('style');
        style.id = id;
        style.textContent = css;
        document.head.appendChild(style);
    }

    async detectTheme(): Promise<'light' | 'dark'> {
        const body = document.body;
        const computedStyle = getComputedStyle(body);
        const bgColor = computedStyle.backgroundColor;
        
        // Simple heuristic to detect dark theme
        const rgb = bgColor.match(/\d+/g);
        if (rgb) {
            const brightness = (parseInt(rgb[0]) + parseInt(rgb[1]) + parseInt(rgb[2])) / 3;
            return brightness < 128 ? 'dark' : 'light';
        }
        
        return 'light';
    }

    onThemeChange(callback: (theme: 'light' | 'dark') => void): void {
        const observer = new MutationObserver(() => {
            const theme = this.detectTheme();
            callback(theme);
        });

        observer.observe(document.body, {
            attributes: true,
            attributeFilter: ['class', 'data-theme']
        });
    }
}
```

---

## Custom Dialogs and Panels

### Advanced Dialog System

```typescript
class AdvancedDialog extends Dialog {
    private formData = new Map<string, any>();
    private validators = new Map<string, (value: any) => ValidationResult>();
    private onSubmitCallbacks: Array<(data: any) => Promise<boolean>> = [];

    constructor(config: AdvancedDialogConfig) {
        super({
            title: config.title,
            content: config.content,
            width: config.width || "600px",
            height: config.height || "400px",
            destroyCallback: config.onClose
        });

        this.setupForm(config);
    }

    private setupForm(config: AdvancedDialogConfig): void {
        if (config.fields) {
            this.renderFormFields(config.fields);
        }

        if (config.buttons) {
            this.renderButtons(config.buttons);
        }
    }

    private renderFormFields(fields: FormField[]): void {
        const formContainer = this.element.querySelector('.form-container');
        if (!formContainer) return;

        fields.forEach(field => {
            const fieldElement = this.createFieldElement(field);
            formContainer.appendChild(fieldElement);

            if (field.validator) {
                this.validators.set(field.name, field.validator);
            }
        });
    }

    private createFieldElement(field: FormField): HTMLElement {
        const container = document.createElement('div');
        container.className = 'form-field';

        const label = document.createElement('label');
        label.textContent = field.label;
        label.setAttribute('for', field.name);

        let input: HTMLElement;

        switch (field.type) {
            case 'text':
                input = document.createElement('input');
                (input as HTMLInputElement).type = 'text';
                break;
            case 'textarea':
                input = document.createElement('textarea');
                (input as HTMLTextAreaElement).rows = field.rows || 4;
                break;
            case 'select':
                input = this.createSelectElement(field);
                break;
            case 'checkbox':
                input = document.createElement('input');
                (input as HTMLInputElement).type = 'checkbox';
                break;
            default:
                input = document.createElement('input');
        }

        input.id = field.name;
        input.addEventListener('input', (e) => {
            this.updateFormData(field.name, (e.target as any).value);
            this.validateField(field.name);
        });

        container.appendChild(label);
        container.appendChild(input);

        return container;
    }

    private createSelectElement(field: FormField): HTMLSelectElement {
        const select = document.createElement('select');
        
        if (field.options) {
            field.options.forEach(option => {
                const optElement = document.createElement('option');
                optElement.value = option.value;
                optElement.textContent = option.label;
                select.appendChild(optElement);
            });
        }

        return select;
    }

    private validateField(fieldName: string): boolean {
        const validator = this.validators.get(fieldName);
        if (!validator) return true;

        const value = this.formData.get(fieldName);
        const result = validator(value);

        const fieldElement = this.element.querySelector(`#${fieldName}`);
        const errorElement = this.element.querySelector(`.error-${fieldName}`);

        if (result.valid) {
            fieldElement?.classList.remove('error');
            errorElement?.remove();
        } else {
            fieldElement?.classList.add('error');
            
            if (!errorElement) {
                const error = document.createElement('div');
                error.className = `error-message error-${fieldName}`;
                error.textContent = result.message || 'Invalid value';
                fieldElement?.parentNode?.appendChild(error);
            }
        }

        return result.valid;
    }

    onSubmit(callback: (data: any) => Promise<boolean>): void {
        this.onSubmitCallbacks.push(callback);
    }

    private async handleSubmit(): Promise<void> {
        // Validate all fields
        let allValid = true;
        for (const [fieldName] of this.validators) {
            if (!this.validateField(fieldName)) {
                allValid = false;
            }
        }

        if (!allValid) {
            showMessage('Please fix validation errors', 6000, 'error');
            return;
        }

        // Execute submit callbacks
        for (const callback of this.onSubmitCallbacks) {
            try {
                const shouldClose = await callback(Object.fromEntries(this.formData));
                if (shouldClose) {
                    this.destroy();
                    break;
                }
            } catch (error) {
                console.error('Submit callback failed:', error);
                showMessage('An error occurred while processing', 6000, 'error');
            }
        }
    }

    private updateFormData(key: string, value: any): void {
        this.formData.set(key, value);
    }
}

interface AdvancedDialogConfig {
    title: string;
    content?: string;
    width?: string;
    height?: string;
    fields?: FormField[];
    buttons?: DialogButton[];
    onClose?: () => void;
}

interface FormField {
    name: string;
    label: string;
    type: 'text' | 'textarea' | 'select' | 'checkbox' | 'number';
    required?: boolean;
    placeholder?: string;
    rows?: number;
    options?: Array<{ value: string; label: string }>;
    validator?: (value: any) => ValidationResult;
}

interface ValidationResult {
    valid: boolean;
    message?: string;
}

// Usage example
const dialog = new AdvancedDialog({
    title: 'Create New Template',
    fields: [
        {
            name: 'name',
            label: 'Template Name',
            type: 'text',
            required: true,
            validator: (value) => ({
                valid: value && value.length > 2,
                message: 'Name must be at least 3 characters'
            })
        },
        {
            name: 'content',
            label: 'Template Content',
            type: 'textarea',
            rows: 10,
            required: true
        }
    ]
});

dialog.onSubmit(async (data) => {
    await createTemplate(data.name, data.content);
    showMessage('Template created successfully!');
    return true; // Close dialog
});
```

---

*This is a comprehensive guide that covers the essential aspects of SiYuan plugin development. For more advanced topics and specific use cases, refer to the related documentation and community resources.*
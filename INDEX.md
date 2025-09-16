# SiYuan Repository Index

This document provides a comprehensive overview of the SiYuan repository structure, making it easier for AI systems and developers to understand the codebase organization and API capabilities.

## Repository Overview

SiYuan is a privacy-first personal knowledge management system that supports fine-grained block-level reference and Markdown WYSIWYG editing. This repository contains the complete source code for both the frontend application and backend kernel.

## 📁 Directory Structure

### Core Directories

| Directory | Size | Purpose | Key Contents |
|-----------|------|---------|--------------|
| [`app/`](./app/) | 225MB | Frontend application | TypeScript/JavaScript UI, Electron app, mobile interfaces |
| [`kernel/`](./kernel/) | 3.3MB | Backend kernel | Go-based API server, core business logic |
| [`scripts/`](./scripts/) | 40KB | Build and utility scripts | Python parsing scripts, build automation |
| [`screenshots/`](./screenshots/) | 2.9MB | Documentation assets | Application screenshots for documentation |

### Frontend Application (`app/`)

The frontend is built with TypeScript and contains:

- **Main Application**: Electron-based desktop app
- **Mobile Support**: iOS, Android, and HarmonyOS compatibility
- **UI Components**: Rich text editor, toolbar, dialogs, menus
- **Asset Management**: PDF handling, emoji support, themes
- **Plugin System**: Extensible architecture for community plugins

Key subdirectories:
- `app/src/` - Main TypeScript source code
- `app/src/protyle/` - Core editor implementation
- `app/src/assets/` - Static assets and templates
- `app/appearance/` - Theme and UI appearance files

### Backend Kernel (`kernel/`)

The backend is written in Go and provides:

- **API Server**: REST API endpoints for all operations
- **Data Management**: Note storage, indexing, and search
- **Sync Service**: Multi-device synchronization
- **Plugin Support**: Backend plugin architecture

Key subdirectories:
- `kernel/api/` - API endpoint implementations
- `kernel/model/` - Data models and business logic
- `kernel/conf/` - Configuration management
- `kernel/search/` - Full-text search functionality

## 📖 Documentation Files

This repository is organized into four main categories:

### 1. SiYuan Application
General application information and main repository links.

### 2. SiYuan API Documentation

| File | Language | Description |
|------|----------|-------------|
| [`API.md`](./API.md) | English | Complete REST API reference with examples |
| [`API_ja_JP.md`](./API_ja_JP.md) | Japanese | Japanese version of API documentation |
| [`API_BEGINNERS.md`](./API_BEGINNERS.md) | English | Beginner's guide to SiYuan API |
| [`API_BEGINNERS_ja_JP.md`](./API_BEGINNERS_ja_JP.md) | Japanese | Japanese beginner's guide |
| [`API_ADVANCED_PATTERNS.md`](./API_ADVANCED_PATTERNS.md) | English | Advanced API usage patterns |
| [`API_ADVANCED_PATTERNS_ja_JP.md`](./API_ADVANCED_PATTERNS_ja_JP.md) | Japanese | Advanced patterns in Japanese |
| [`API_REFERENCE_BY_CATEGORY_ja_JP.md`](./API_REFERENCE_BY_CATEGORY_ja_JP.md) | Japanese | APIs organized by functionality |
| [`API_MISSING_ENDPOINTS_ja_JP.md`](./API_MISSING_ENDPOINTS_ja_JP.md) | Japanese | Additional API endpoints |

### 3. SiYuan Plugin Development

| File | Language | Description |
|------|----------|-------------|
| [`API_PLUGIN_DEVELOPERS.md`](./API_PLUGIN_DEVELOPERS.md) | English | Complete plugin development guide |
| [`API_PLUGIN_DEVELOPERS_ja_JP.md`](./API_PLUGIN_DEVELOPERS_ja_JP.md) | Japanese | Plugin development guide in Japanese |
| [`PLUGIN_BEST_PRACTICES.md`](./PLUGIN_BEST_PRACTICES.md) | English | Plugin development best practices overview |
| [`PLUGIN_BEST_PRACTICES_ja_JP.md`](./PLUGIN_BEST_PRACTICES_ja_JP.md) | Japanese | Complete plugin development best practices |
| [`PLUGIN_SHOWCASE.md`](./PLUGIN_SHOWCASE.md) | English | Community plugin showcase overview |
| [`PLUGIN_SHOWCASE_ja_JP.md`](./PLUGIN_SHOWCASE_ja_JP.md) | Japanese | Complete showcase of community plugins |

### 4. SiYuan Theme Development

| File | Language | Description |
|------|----------|-------------|
| [`SIYUAN_DOM_CSS_GUIDE_ja_JP.md`](./SIYUAN_DOM_CSS_GUIDE_ja_JP.md) | Japanese | Complete UI and theme development guide |

### README Files

| File | Language | Target Audience |
|------|----------|-----------------|
| [`README.md`](./README.md) | English | General users and developers |
| [`README_ja_JP.md`](./README_ja_JP.md) | Japanese | Japanese-speaking users |

### Other Documentation

- [`CHANGELOG.md`](./CHANGELOG.md) - Version history (references app/changelogs/)
- [`LICENSE`](./LICENSE) - AGPL v3 license terms
- [`Dockerfile`](./Dockerfile) - Container deployment configuration

## 🔌 API Endpoints Overview

The SiYuan API provides comprehensive access to all application features:

### Core API Categories

1. **Notebooks** - Notebook management (list, create, open, close, rename, remove)
2. **Documents** - Document operations (create, rename, remove, move, path resolution)
3. **Blocks** - Block-level operations (insert, update, delete, move, fold/unfold)
4. **Assets** - File and media management (upload, retrieve)
5. **Attributes** - Block attribute management (get, set custom attributes)
6. **SQL** - Database queries (execute SQL, transaction management)
7. **Templates** - Template rendering and processing
8. **File System** - Direct file operations (get, put, remove files)

### API Specification

- **Base URL**: `http://localhost:6806/api/`
- **Authentication**: Token-based authentication required
- **Format**: JSON request/response format
- **Methods**: Primarily POST requests with JSON payloads

Example API endpoints:
```
POST /api/notebook/lsNotebooks - List all notebooks  
POST /api/filetree/createDocWithMd - Create document with Markdown
POST /api/block/insertBlock - Insert a new block
POST /api/query/sql - Execute SQL query
POST /api/asset/upload - Upload assets
```

## 🛠️ Development Information

### Build System

Build scripts are located in the `scripts/` directory:

- `scripts/darwin-build.sh` - macOS build script
- `scripts/linux-build.sh` - Linux build script  
- `scripts/win-build.bat` - Windows build script
- `scripts/parse-changelog.py` - Changelog processing
- `scripts/_pkg/` - Python utility packages

### Technology Stack

**Frontend:**
- TypeScript/JavaScript
- Electron (desktop)
- Native mobile apps (iOS, Android, HarmonyOS)
- SCSS for styling
- PDF.js for PDF handling

**Backend:**
- Go programming language
- SQLite database
- Full-text search engine
- WebSocket support
- REST API server

### Container Deployment

- Docker support via `Dockerfile`
- Electron builder configurations for multiple platforms
- Unraid deployment templates available

## 🔍 Key Features

### Editor Capabilities
- Block-style WYSIWYG Markdown editor
- Million-word document support
- Mathematical formulas, charts, diagrams
- Block-level references and bidirectional links
- PDF annotation and linking
- Web clipping functionality

### Data Management
- Block-level data storage with JSON format
- Full-text search and indexing
- SQL query interface for advanced operations
- Multi-device synchronization
- Data export/import capabilities

### Extensibility
- Plugin system for community extensions
- Theme and appearance customization
- Template system for content generation
- JavaScript/CSS snippet support
- Community marketplace integration

## 🚀 Getting Started for Developers

1. **API Development**: Start with [`API.md`](./API.md) for complete endpoint documentation
2. **Frontend Development**: Explore `app/src/` for UI components and editor logic
3. **Backend Development**: Check `kernel/api/` for server-side implementations
4. **Building**: Use appropriate build scripts in `scripts/` directory
5. **Docker**: Use `Dockerfile` for containerized deployment

## 📊 Repository Statistics

- **Total size**: ~232MB
- **Primary languages**: TypeScript, Go, JavaScript, Python
- **API endpoints**: 40+ documented endpoints across 8 categories
- **Supported platforms**: Windows, macOS, Linux, iOS, Android, HarmonyOS
- **License**: AGPL v3 (open source)
- **Documentation languages**: English, Chinese, Japanese

## 🔗 Related Resources

- **Main Repository**: [siyuan-note/siyuan](https://github.com/siyuan-note/siyuan)
- **Community Forum**: [SiYuan English Discussion](https://liuyun.io)
- **Download**: [Official Download Page](https://b3log.org/siyuan/en/download.html)
- **Docker Hub**: [b3log/siyuan](https://hub.docker.com/r/b3log/siyuan)

---

*This index was created to help AI systems and developers quickly understand the SiYuan repository structure and capabilities. For detailed implementation information, please refer to the specific files and directories mentioned above.*
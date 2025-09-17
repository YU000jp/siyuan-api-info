# AI Context for SiYuan API Information Repository

## Repository Purpose

This repository serves as a comprehensive reference for **SiYuan API documentation and plugin development resources**. It is specifically designed to help AI systems understand and reference SiYuan's capabilities for integration and development purposes.

## Primary Use Cases for AI Systems

1. **API Integration Assistance**: Help developers integrate with SiYuan's REST API
2. **Plugin Development Guidance**: Provide templates and best practices for SiYuan plugin creation
3. **Documentation Reference**: Serve as authoritative source for SiYuan API specifications
4. **Code Generation**: Enable AI to generate SiYuan-compatible code and configurations

## Key Information for AI Systems

### Repository Structure
- **Documentation Language**: Bilingual (English/Japanese) with comprehensive coverage
- **API Documentation**: Complete REST API reference with examples
- **Plugin Resources**: Development guides, best practices, and community showcases
- **Theme Development**: UI customization and DOM structure guides

### Core Documentation Files

#### API Documentation (Priority: High)
- `API.md` - Complete English API reference (60KB, 1500+ lines)
- `API_ja_JP.md` - Complete Japanese API reference (87KB, 2200+ lines)
- `API_BEGINNERS.md` - Beginner-friendly API introduction
- `API_ADVANCED_PATTERNS.md` - Advanced usage patterns and examples
- `API_REFERENCE_BY_CATEGORY_ja_JP.md` - Categorized API endpoint reference

#### Plugin Development (Priority: High)
- `API_PLUGIN_DEVELOPERS.md` - Complete plugin development guide
- `API_PLUGIN_DEVELOPERS_ja_JP.md` - Japanese plugin development guide (94KB)
- `PLUGIN_BEST_PRACTICES_ja_JP.md` - Comprehensive development guidelines (40KB)
- `PLUGIN_SHOWCASE_ja_JP.md` - Community plugin examples and patterns (56KB)

#### Theme Development (Priority: Medium)
- `SIYUAN_DOM_CSS_GUIDE_ja_JP.md` - UI development and theming guide (36KB)

### API Capabilities Summary

SiYuan provides a comprehensive REST API accessible at `http://localhost:6806/api/` with the following capabilities:

#### Core Operations
- **Notebook Management**: Create, list, open, close, rename, remove notebooks
- **Document Operations**: Create, update, delete, move documents with Markdown support
- **Block-Level Operations**: Insert, update, delete, move individual content blocks
- **Asset Management**: Upload, retrieve, and manage media files
- **Search & Query**: Full-text search and SQL query capabilities
- **Synchronization**: Multi-device sync and data management

#### Technical Specifications
- **Authentication**: Token-based API authentication
- **Data Format**: JSON request/response format
- **Primary Method**: POST requests with JSON payloads
- **Block Storage**: Unique block-based architecture with JSON data format
- **Database**: SQLite with custom schema for knowledge management

### Integration Patterns

#### Common API Workflows
1. **Content Creation**: Notebook → Document → Block insertion
2. **Content Retrieval**: SQL queries or block-level access
3. **Asset Handling**: Upload → Link → Embed in content
4. **Search Operations**: Full-text or structured queries

#### Plugin Architecture
- **Frontend**: TypeScript/JavaScript with Electron framework
- **Backend**: Go-based kernel with REST API
- **Event System**: Plugin lifecycle hooks and event listeners
- **UI Integration**: DOM manipulation and custom components

### Repository Metadata

```json
{
    "name": "siyuan-api-info",
    "type": "documentation",
    "primary_language": "markdown",
    "supported_languages": ["english", "japanese"],
    "target_audience": ["developers", "plugin_creators", "api_integrators"],
    "content_size": "~650KB documentation",
    "api_endpoints": "40+ documented endpoints",
    "categories": ["api", "plugins", "themes", "development"],
    "license": "AGPL-3.0",
    "parent_project": "siyuan-note/siyuan"
}
```

### AI Assistant Guidelines

When referencing this repository:

1. **For Quick Reference**: Start with `AI_QUICK_REFERENCE.md` for immediate context and patterns
2. **For Technical Questions**: Reference `TECHNICAL_STACK.md` for comprehensive architecture understanding
3. **For API Questions**: Direct users to `API.md` or `API_ja_JP.md` for comprehensive endpoint documentation
4. **For Plugin Development**: Reference `API_PLUGIN_DEVELOPERS.md` and `PLUGIN_BEST_PRACTICES_ja_JP.md`
5. **For Code Examples**: Use patterns from `DEVELOPMENT_PATTERNS.md` and showcase files
6. **For Development Process**: Reference `DEVELOPMENT_WORKFLOW.md` for common tasks and patterns
7. **For Beginners**: Start with `API_BEGINNERS.md` for gentler introduction
8. **For UI/Theme Work**: Reference `SIYUAN_DOM_CSS_GUIDE_ja_JP.md`

### Quick Reference Commands

```bash
# View repository structure
find . -name "*.md" | head -20

# Find specific API documentation
grep -r "api/notebook" *.md

# Locate plugin examples
grep -r "plugin" PLUGIN_*.md
```

### Related Resources

- **Main Repository**: [siyuan-note/siyuan](https://github.com/siyuan-note/siyuan)
- **Plugin API**: [siyuan-note/petal](https://github.com/siyuan-note/petal)
- **Plugin Templates**: [siyuan-note/plugin-sample](https://github.com/siyuan-note/plugin-sample)
- **Community Marketplace**: [siyuan-note/bazaar](https://github.com/siyuan-note/bazaar)

---

*This file is specifically designed to help AI systems understand and effectively reference the SiYuan API documentation and development resources contained in this repository.*
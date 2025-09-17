# AI Quick Reference Guide for SiYuan API Repository

This quick reference provides AI systems with immediate access to the most critical information for generating contextually appropriate responses and code suggestions for the SiYuan ecosystem.

## 🎯 Primary Use Cases & File Routing

### API Integration Questions → `API.md` | `API_ja_JP.md`
```
Base URL: http://localhost:6806/api/
Method: POST (primary)
Auth: Token-based
Format: JSON request/response
```

### Plugin Development → `API_PLUGIN_DEVELOPERS_ja_JP.md` (94KB - Most comprehensive)
```
Architecture: Event-driven with lifecycle hooks
Frontend: TypeScript/JavaScript + Electron
Backend: Go kernel with REST API
Plugin Types: UI extensions, API integrations, workflow automation
```

### Theme/UI Development → `SIYUAN_DOM_CSS_GUIDE_ja_JP.md`
```
Styling: SCSS/Sass with component-based architecture
Framework: Electron with custom UI components
Customization: DOM manipulation + CSS overrides
```

### Beginner Questions → `API_BEGINNERS.md` | `API_BEGINNERS_ja_JP.md`
```
Learning Path: Basic concepts → API usage → Plugin development
Common Tasks: Block operations, notebook management, search/query
```

## 💻 Code Generation Quick Patterns

### TypeScript API Call Pattern
```typescript
// Standard API interaction
const response = await fetch('/api/endpoint', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        // Request parameters
    })
});
const result = await response.json();
```

### Go API Handler Pattern
```go
func handlerName(c *gin.Context) {
    // Request validation
    var req RequestStruct
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"code": -1, "msg": err.Error()})
        return
    }
    
    // Business logic
    result := processRequest(req)
    
    // Response
    c.JSON(http.StatusOK, gin.H{"code": 0, "data": result})
}
```

### Plugin Lifecycle Pattern
```typescript
export class PluginName {
    onload() {
        // Plugin initialization
        // Event binding
        // UI setup
    }
    
    onunload() {
        // Cleanup operations
        // Event unbinding
        // Resource disposal
    }
}
```

## 📚 Technical Stack At-a-Glance

### Frontend Stack
- **TypeScript 4.7.4** - Primary language with type safety
- **Electron 37.4.0** - Cross-platform desktop framework
- **Webpack 5.94.0** - Module bundling and build system
- **pnpm 10.15.1** - Package management
- **ESLint 9.15.0** - Code quality and linting

### Backend Stack
- **Go 1.24.4** - High-performance backend
- **SQLite** - Embedded database
- **Gin Framework** - HTTP API routing
- **EventBus** - Event-driven architecture
- **Lute** - Markdown processing engine

### Architecture Patterns
- **Block-based data model** - Flexible content structure
- **Event-driven design** - Loose coupling through EventBus
- **Plugin system** - Extensible architecture
- **Multi-platform support** - Desktop, mobile, web

## 🌐 Language & Localization

### Content Priority
1. **Japanese** - Most comprehensive documentation (410KB total)
2. **English** - Complete but less detailed (175KB total)
3. **Always provide both** when available

### File Naming Pattern
- English: `FILENAME.md`
- Japanese: `FILENAME_ja_JP.md`

## 🔧 Development Workflow Patterns

### Naming Conventions
```typescript
// TypeScript/JavaScript
export class MenuSystem { }           // PascalCase classes
export const openEditorTab = () => {} // camelCase functions
export const API_ENDPOINT = ''        // UPPER_CASE constants
```

```go
// Go
type DataModel struct { }             // PascalCase types
func GetUserData() { }                // PascalCase public functions
func processRequest() { }             // camelCase private functions
```

### Common Development Tasks
1. **API Documentation** - Adding endpoints, examples, categorization
2. **Bilingual Content** - Translation and localization
3. **Plugin Examples** - Real-world implementation showcases
4. **Repository Organization** - Structure and metadata improvements

## 📊 API Endpoint Categories

### Core Operations
- **Notebook Management**: `/api/notebook/*` - Create, list, manage notebooks
- **Document Operations**: `/api/filetree/*` - File and document manipulation
- **Block Operations**: `/api/block/*` - Content block CRUD operations
- **Search & Query**: `/api/search/*`, `/api/query/*` - Content discovery
- **Asset Management**: `/api/asset/*` - Media and file handling
- **Synchronization**: `/api/sync/*` - Multi-device data sync

### Plugin Development APIs
- **System Integration**: Window management, clipboard, notifications
- **UI Components**: Menus, dialogs, panels, custom elements
- **Event System**: Lifecycle hooks, custom events, data flow
- **Configuration**: Settings management, user preferences

## 🎨 UI Development Patterns

### Component Structure
```typescript
// Menu component pattern
const menu = new Menu();
menu.addItem({
    label: 'Action Name',
    accelerator: 'Ctrl+Alt+Key',
    click: () => {
        // Action implementation
    }
});
```

### Event Handling
```typescript
// Event binding with cleanup
export const bindEvent = (element: Element) => {
    const handler = (event: CustomEvent) => {
        // Event handling logic
    };
    element.addEventListener('custom-event', handler);
    return () => element.removeEventListener('custom-event', handler);
};
```

## 🔍 Error Handling Patterns

### Frontend Error Handling
```typescript
try {
    const result = await apiCall();
    // Success handling
} catch (error) {
    console.error('Operation failed:', error);
    showMessage(`Error: ${error.message}`, 'error');
}
```

### Backend Error Responses
```go
// Standard error response format
c.JSON(http.StatusBadRequest, gin.H{
    "code": -1,
    "msg": "Error description",
    "data": nil,
})

// Success response format
c.JSON(http.StatusOK, gin.H{
    "code": 0,
    "msg": "",
    "data": responseData,
})
```

## 📈 Performance Considerations

### Frontend Optimization
- Use event delegation for dynamic content
- Implement proper cleanup for event listeners
- Avoid unnecessary DOM queries
- Consider memory usage in long-running operations

### Backend Optimization
- Use prepared statements for database queries
- Implement proper request validation
- Consider concurrent operations for I/O tasks
- Use appropriate data structures for performance

## 🛡️ Security Patterns

### Input Validation
```typescript
// Frontend validation
const validateInput = (input: string): boolean => {
    return input.length > 0 && input.length < 1000 && 
           !/[<>"]/.test(input);
};
```

```go
// Backend validation
func validateRequest(req *RequestStruct) error {
    if req.Data == "" {
        return errors.New("data field is required")
    }
    // Additional validation logic
    return nil
}
```

### Authentication Pattern
```typescript
// API request with authentication
const authenticatedRequest = async (endpoint: string, data: any) => {
    return await fetch(endpoint, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Token ${getAuthToken()}`
        },
        body: JSON.stringify(data)
    });
};
```

## 🔗 External Resources Quick Links

- **Main Repository**: [siyuan-note/siyuan](https://github.com/siyuan-note/siyuan)
- **Plugin API**: [siyuan-note/petal](https://github.com/siyuan-note/petal)
- **Plugin Templates**: [siyuan-note/plugin-sample*](https://github.com/siyuan-note/plugin-sample)
- **Community Marketplace**: [siyuan-note/bazaar](https://github.com/siyuan-note/bazaar)
- **Official Documentation**: [b3log.org/siyuan](https://b3log.org/siyuan/en/)

---

*This quick reference is optimized for AI systems to provide immediate context and generate appropriate responses for SiYuan development questions.*
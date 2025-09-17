# SiYuan Development Patterns and Conventions

This document outlines the coding patterns, naming conventions, and architectural principles used in the SiYuan project to help AI systems generate contextually appropriate code suggestions.

## 📝 Naming Conventions

### TypeScript/JavaScript Frontend
- **Classes**: PascalCase with descriptive names
  ```typescript
  export class Menu { }
  export class MenuItem { }
  export class Menus { }
  ```

- **Functions**: camelCase with verb-noun pattern
  ```typescript
  export const bindMenuKeydown = (event: KeyboardEvent) => { }
  export const openEditorTab = (app: App, ids: string[]) => { }
  export const exportAsset = (src: string) => { }
  ```

- **Constants**: camelCase for functions, UPPER_CASE for constants
- **Files**: kebab-case for component files, camelCase for utility files

### Go Backend
- **Types/Structs**: PascalCase following Go conventions
  ```go
  type DisplayName struct { }
  type Description struct { }
  type Package struct { }
  ```

- **Functions**: 
  - Public functions: PascalCase (`GetPreferredName`)
  - Private functions: camelCase (`getPreferredReadme`)
  - API handlers: camelCase with descriptive action (`checkWorkspaceDir`)

- **Variables**: camelCase for local variables, PascalCase for exported variables

## 🏗️ Architectural Patterns

### Frontend Architecture

#### Module Organization
```
app/src/
├── boot/          # Application initialization
├── config/        # Configuration management
├── editor/        # Editor-related components
├── menus/         # Menu system components
├── plugin/        # Plugin management
├── types/         # TypeScript type definitions
├── window/        # Window management
└── ...
```

#### Event-Driven Pattern
- Extensive use of event listeners and handlers
- Global event management in `boot/globalEvent/`
- Command pattern implementation in `boot/globalEvent/command/`

#### Component Pattern
```typescript
// Class-based components for complex functionality
export class Menu {
    // Component initialization
    // Event binding
    // State management
}

// Function-based utilities for simple operations
export const openEditorTab = (app: App, ids: string[]) => {
    // Direct operation execution
}
```

### Backend Architecture

#### API Handler Pattern
```go
// Consistent API handler signature
func handlerName(c *gin.Context) {
    // Request validation
    // Business logic execution  
    // Response formatting
}
```

#### Data Model Pattern
```go
// Structured data types with JSON tags
type ModelName struct {
    Field1 string `json:"field1"`
    Field2 int    `json:"field2"`
}
```

#### Service Layer Pattern
- Clear separation between API handlers and business logic
- Utility functions for common operations
- Error handling with consistent patterns

## 🔄 Common Development Patterns

### API Integration Pattern
```typescript
// Frontend API call pattern
const response = await fetch('/api/endpoint', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(requestData)
});
const result = await response.json();
```

### Event Handling Pattern
```typescript
// Event binding with proper cleanup
export const bindEventName = (element: Element) => {
    const handler = (event: Event) => {
        // Event handling logic
    };
    element.addEventListener('eventType', handler);
    // Return cleanup function if needed
    return () => element.removeEventListener('eventType', handler);
};
```

### Menu System Pattern
```typescript
// Consistent menu creation pattern
const menu = new Menu();
menu.addItem({
    label: 'Action Name',
    click: () => {
        // Action implementation
    }
});
```

### Plugin Integration Pattern
```typescript
// Plugin lifecycle management
export class PluginManager {
    load(plugin: IPlugin) {
        // Plugin registration
        // Hook binding
        // Event setup
    }
    
    unload(pluginId: string) {
        // Cleanup operations
        // Event unbinding
        // Resource disposal
    }
}
```

## 📊 Data Flow Patterns

### Request-Response Cycle
1. **Frontend Event** → User interaction or system trigger
2. **API Call** → POST request to `/api/endpoint`
3. **Backend Processing** → Gin handler → Business logic
4. **Response** → JSON formatted data
5. **Frontend Update** → DOM manipulation or state update

### State Management Pattern
```typescript
// Configuration state pattern
interface IAppState {
    config: IConfig;
    workspace: IWorkspace;
    plugins: IPlugin[];
}

// State update pattern
const updateState = (newState: Partial<IAppState>) => {
    // Merge with existing state
    // Trigger UI updates
    // Persist if necessary
};
```

### Error Handling Pattern
```typescript
// Frontend error handling
try {
    const result = await apiCall();
    // Success handling
} catch (error) {
    console.error('Operation failed:', error);
    showMessage(error.message, 'error');
}
```

```go
// Backend error handling
func handler(c *gin.Context) {
    if err := validateRequest(); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{
            "code": -1,
            "msg": err.Error(),
        })
        return
    }
    
    // Success response
    c.JSON(http.StatusOK, gin.H{
        "code": 0,
        "data": result,
    })
}
```

## 🔧 Code Generation Guidelines

### TypeScript Interface Patterns
```typescript
// Configuration interfaces
interface IConfig {
    readonly property: string;
    optional?: boolean;
    nested: {
        subProperty: number;
    };
}

// Event handler interfaces
interface IEventHandler {
    (event: CustomEvent): void | Promise<void>;
}

// Plugin interfaces
interface IPlugin {
    name: string;
    version: string;
    onload(): void;
    onunload(): void;
}
```

### Go Struct Patterns
```go
// API request/response structures
type APIRequest struct {
    Action string      `json:"action"`
    Data   interface{} `json:"data"`
}

type APIResponse struct {
    Code int         `json:"code"`
    Msg  string      `json:"msg"`
    Data interface{} `json:"data,omitempty"`
}

// Business model structures
type ModelName struct {
    ID        string    `json:"id"`
    CreatedAt time.Time `json:"created"`
    UpdatedAt time.Time `json:"updated"`
}
```

## 🎯 Best Practices for AI Code Generation

### Function Naming
- Use descriptive verbs for actions: `create`, `update`, `delete`, `get`, `set`
- Include context in function names: `openEditorTab`, `bindMenuKeydown`
- Follow existing patterns in the same module

### Error Messages
- Provide clear, actionable error messages
- Include context about what operation failed
- Use consistent error response format

### Documentation
- Include JSDoc comments for complex functions
- Document parameter types and return values
- Explain business logic for domain-specific operations

### Testing Patterns
```typescript
// Mock pattern for testing
const mockAPI = {
    fetch: jest.fn().mockResolvedValue({
        json: () => Promise.resolve(mockData)
    })
};

// Test structure pattern
describe('Component Name', () => {
    beforeEach(() => {
        // Setup
    });
    
    it('should perform expected action', async () => {
        // Test implementation
    });
    
    afterEach(() => {
        // Cleanup
    });
});
```

## 🔍 Code Review Guidelines

### Consistency Checks
- Follow established naming conventions
- Use existing utility functions where appropriate
- Maintain consistent indentation and formatting
- Include proper error handling

### Performance Considerations
- Avoid unnecessary DOM queries
- Use event delegation for dynamic content
- Implement proper cleanup for event listeners
- Consider memory usage in long-running operations

### Security Patterns
- Validate all user inputs
- Sanitize data before DOM insertion
- Use prepared statements for database queries
- Implement proper authentication checks

---

*This development patterns guide is designed to help AI systems understand and follow SiYuan's established conventions for consistent, maintainable code generation.*
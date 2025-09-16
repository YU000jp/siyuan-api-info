# SiYuan API Beginner's Guide

An introductory guide for developers new to the SiYuan API. Learn from basic concepts to practical usage, step by step.

**Target Audience**: Developers new to SiYuan API

**Prerequisites**: 
- Basic programming knowledge (JavaScript/TypeScript recommended)
- Understanding of HTTP API concepts
- Knowledge of JSON format

**Related Documentation**:
- [Plugin Developer API Guide](API_PLUGIN_DEVELOPERS.md) - Plugin development focused
- [Advanced API Patterns](API_ADVANCED_PATTERNS.md) - Complex operations
- [Complete API Reference](API.md) - All endpoints

---

## Table of Contents

- [What is SiYuan API](#what-is-siyuan-api)
- [Environment Setup](#environment-setup)
- [Basic Concepts](#basic-concepts)
- [Your First API Call](#your-first-api-call)
- [Learning Basic Operations](#learning-basic-operations)
- [Practical Usage Examples](#practical-usage-examples)
- [Common Questions and Solutions](#common-questions-and-solutions)
- [Next Steps](#next-steps)

---

## What is SiYuan API

The SiYuan API is a RESTful API that allows you to programmatically control the functionality of the SiYuan note application.

### What You Can Do

- **Note Management**: Create, edit, and delete documents and blocks
- **Data Search**: Search note content, execute SQL queries
- **File Operations**: Upload and manage images and attachments
- **UI Operations**: Update application screens and change states
- **Extensions**: Develop plugins and custom features

### Main Use Cases

- **Automation**: Periodic backups and data organization
- **Integration**: Connect with other applications
- **Customization**: Add unique features
- **Analysis**: Get usage statistics and analytics for notes

### API Features

**Block-Based Architecture**
- Everything in SiYuan is a block (paragraphs, headings, lists, etc.)
- Each block has a unique ID for precise operations
- Support for hierarchical structures and references

**Real-Time Synchronization**
- Changes are reflected immediately in the UI
- Multiple clients can be synchronized
- Event-driven updates

**Flexible Querying**
- SQL-based advanced queries
- Full-text search capabilities
- Attribute-based filtering

---

## Environment Setup

### 1. SiYuan Installation

First, install SiYuan if you haven't already:

1. Download from [SiYuan Official Site](https://b3log.org/siyuan/)
2. Install and launch the application
3. Create or open a workspace

### 2. API Access Setup

**Enable API Access**
1. Open SiYuan Settings
2. Go to "About" section
3. Enable "Serve" mode
4. Note the API token displayed

**Default Settings**
- API URL: `http://localhost:6806`
- Authentication: Token-based
- Format: JSON

### 3. Development Environment

**Recommended Tools**
- Code editor (VS Code, etc.)
- HTTP client (Postman, curl, etc.)
- Browser developer tools

**Programming Languages**
- JavaScript/TypeScript (recommended)
- Python
- Any language with HTTP client support

---

## Basic Concepts

### Authentication

All API requests require token authentication:

```javascript
const headers = {
  'Authorization': `Token your_api_token_here`,
  'Content-Type': 'application/json'
};
```

### Request Format

Most API endpoints use POST method with JSON payload:

```javascript
const response = await fetch('http://localhost:6806/api/notebook/lsNotebooks', {
  method: 'POST',
  headers: headers,
  body: JSON.stringify({})
});
```

### Response Format

Standard response structure:

```json
{
  "code": 0,           // 0 = success, non-zero = error
  "msg": "success",    // Status message
  "data": {            // Response data (varies by endpoint)
    // ... endpoint-specific data
  }
}
```

### Key Concepts

**Notebooks**
- Top-level containers for documents
- Each notebook has a unique ID
- Can be opened/closed independently

**Documents**
- Individual note files
- Stored as Markdown with metadata
- Belong to a specific notebook

**Blocks**
- Smallest unit of content
- Have unique IDs and types
- Can contain text, images, tables, etc.

---

## Your First API Call

Let's start with a simple API call to list all notebooks.

### Step 1: Get API Token

1. Open SiYuan Settings → About
2. Enable "Serve" option if not already enabled
3. Copy the API token shown

### Step 2: Make Your First Request

**Using JavaScript (Browser/Node.js):**

```javascript
async function listNotebooks() {
  const token = 'your_api_token_here'; // Replace with actual token
  
  try {
    const response = await fetch('http://localhost:6806/api/notebook/lsNotebooks', {
      method: 'POST',
      headers: {
        'Authorization': `Token ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({})
    });
    
    const result = await response.json();
    console.log('Notebooks:', result.data.notebooks);
    return result.data.notebooks;
  } catch (error) {
    console.error('Error:', error);
  }
}

// Call the function
listNotebooks();
```

**Using curl:**

```bash
curl -X POST \
  -H "Authorization: Token your_api_token_here" \
  -H "Content-Type: application/json" \
  -d '{}' \
  http://localhost:6806/api/notebook/lsNotebooks
```

**Using Python:**

```python
import requests
import json

def list_notebooks():
    token = 'your_api_token_here'  # Replace with actual token
    url = 'http://localhost:6806/api/notebook/lsNotebooks'
    headers = {
        'Authorization': f'Token {token}',
        'Content-Type': 'application/json'
    }
    
    response = requests.post(url, headers=headers, json={})
    result = response.json()
    
    if result['code'] == 0:
        print('Notebooks:', result['data']['notebooks'])
        return result['data']['notebooks']
    else:
        print('Error:', result['msg'])

# Call the function
list_notebooks()
```

### Step 3: Understanding the Response

A successful response will look like:

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "notebooks": [
      {
        "id": "20210808180117-czj9bvb",
        "name": "Welcome",
        "icon": "1f4dd",
        "sort": 0,
        "closed": false
      }
    ]
  }
}
```

---

## Learning Basic Operations

### 1. Notebook Operations

**List all notebooks:**
```javascript
await callAPI('/notebook/lsNotebooks', {});
```

**Create a new notebook:**
```javascript
await callAPI('/notebook/createNotebook', {
  name: 'My New Notebook'
});
```

**Open a notebook:**
```javascript
await callAPI('/notebook/openNotebook', {
  notebook: 'notebook_id_here'
});
```

### 2. Document Operations

**Create a document with Markdown:**
```javascript
await callAPI('/filetree/createDocWithMd', {
  notebook: 'notebook_id',
  path: '/New Document',
  markdown: '# Hello World\n\nThis is my first document!'
});
```

**Get document content:**
```javascript
await callAPI('/export/exportMdContent', {
  id: 'document_id_here'
});
```

### 3. Block Operations

**Insert a new block:**
```javascript
await callAPI('/block/insertBlock', {
  dataType: 'markdown',
  data: 'This is a new paragraph',
  parentID: 'parent_block_id'
});
```

**Update block content:**
```javascript
await callAPI('/block/updateBlock', {
  dataType: 'markdown',
  data: 'Updated content',
  id: 'block_id_here'
});
```

### 4. Search Operations

**Simple text search:**
```javascript
await callAPI('/search/searchBlock', {
  query: 'search term'
});
```

**SQL query:**
```javascript
await callAPI('/query/sql', {
  stmt: 'SELECT * FROM blocks WHERE content LIKE "%search%" LIMIT 10'
});
```

### Helper Function

Create a reusable helper function:

```javascript
async function callAPI(endpoint, data) {
  const token = 'your_api_token_here';
  
  try {
    const response = await fetch(`http://localhost:6806/api${endpoint}`, {
      method: 'POST',
      headers: {
        'Authorization': `Token ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    });
    
    const result = await response.json();
    
    if (result.code === 0) {
      return result.data;
    } else {
      throw new Error(`API Error: ${result.msg}`);
    }
  } catch (error) {
    console.error('API call failed:', error);
    throw error;
  }
}
```

---

## Practical Usage Examples

### Example 1: Daily Note Creator

Create a daily note with current date:

```javascript
async function createDailyNote() {
  const today = new Date().toISOString().split('T')[0];
  const notebooks = await callAPI('/notebook/lsNotebooks', {});
  const notebook = notebooks.notebooks[0]; // Use first notebook
  
  const markdown = `# Daily Note - ${today}

## Tasks for Today
- [ ] Task 1
- [ ] Task 2

## Notes

## Reflections
`;

  await callAPI('/filetree/createDocWithMd', {
    notebook: notebook.id,
    path: `/Daily Notes/${today}`,
    markdown: markdown
  });
  
  console.log(`Created daily note for ${today}`);
}
```

### Example 2: Backup All Notes

Export all documents in a notebook:

```javascript
async function backupNotebook(notebookId) {
  // Get all documents in the notebook
  const query = `
    SELECT id, content, path 
    FROM blocks 
    WHERE type = 'd' AND box = '${notebookId}'
  `;
  
  const documents = await callAPI('/query/sql', { stmt: query });
  
  for (const doc of documents) {
    const content = await callAPI('/export/exportMdContent', {
      id: doc.id
    });
    
    // Save content to file or process as needed
    console.log(`Document: ${doc.path}`);
    console.log(`Content length: ${content.content.length} characters`);
  }
}
```

### Example 3: Tag Organizer

Find and organize blocks with specific tags:

```javascript
async function organizeByTags() {
  const taggedBlocks = await callAPI('/query/sql', {
    stmt: `
      SELECT b.id, b.content, b.path, ba.value as tags
      FROM blocks b
      JOIN block_attrs ba ON b.id = ba.block_id
      WHERE ba.name = 'custom-tag'
    `
  });
  
  const tagGroups = {};
  
  taggedBlocks.forEach(block => {
    const tags = block.tags.split(',');
    tags.forEach(tag => {
      if (!tagGroups[tag.trim()]) {
        tagGroups[tag.trim()] = [];
      }
      tagGroups[tag.trim()].push(block);
    });
  });
  
  console.log('Blocks organized by tags:', tagGroups);
}
```

---

## Common Questions and Solutions

### Q: How do I get the API token?

**A:** Go to SiYuan Settings → About, enable "Serve" mode, and copy the displayed token.

### Q: API calls are failing with authentication error

**A:** 
1. Verify the token is correct
2. Check the `Authorization` header format: `Token your_token_here`
3. Ensure SiYuan is running and API is enabled

### Q: How do I find block IDs?

**A:** Use SQL queries or search endpoints:
```javascript
await callAPI('/query/sql', {
  stmt: 'SELECT id, content FROM blocks WHERE content LIKE "%search_term%" LIMIT 5'
});
```

### Q: Can I create nested documents?

**A:** Yes, use the `path` parameter with forward slashes:
```javascript
await callAPI('/filetree/createDocWithMd', {
  notebook: 'notebook_id',
  path: '/Folder/Subfolder/Document',
  markdown: '# Content'
});
```

### Q: How do I handle API errors?

**A:** Always check the response `code` field:
```javascript
const result = await response.json();
if (result.code !== 0) {
  throw new Error(`API Error ${result.code}: ${result.msg}`);
}
```

### Q: Are there rate limits?

**A:** The local API has no strict rate limits, but avoid excessive concurrent requests for best performance.

---

## Next Steps

### Advanced Learning

1. **[Plugin Developer API Guide](API_PLUGIN_DEVELOPERS.md)** - Learn plugin-specific APIs and patterns
2. **[Advanced API Patterns](API_ADVANCED_PATTERNS.md)** - Complex operations and optimizations
3. **[Complete API Reference](API.md)** - Full documentation of all endpoints

### Practical Projects

**Beginner Projects:**
- Daily note creator
- Simple backup script
- Tag organizer

**Intermediate Projects:**
- Note template system
- Custom search interface
- Statistics dashboard

**Advanced Projects:**
- SiYuan plugin development
- Third-party integrations
- Custom sync solutions

### Community Resources

- **[SiYuan Community Forum](https://liuyun.io/)** - Get help and share ideas
- **[Plugin Marketplace](https://github.com/siyuan-note/bazaar)** - Explore existing plugins
- **[Official Documentation](https://github.com/siyuan-note/siyuan)** - Latest updates and releases

### Development Best Practices

1. **Always handle errors gracefully**
2. **Use descriptive variable names**
3. **Test with small datasets first**
4. **Keep API tokens secure**
5. **Document your code**

---

*This guide provides a foundation for working with the SiYuan API. As you become more comfortable with the basics, explore the advanced documentation and start building your own customizations!*
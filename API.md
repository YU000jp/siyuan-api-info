# SiYuan API Documentation

A comprehensive guide to the SiYuan Note-Taking Application API. This API allows you to programmatically interact with SiYuan to manage notebooks, documents, blocks, and perform various operations.

**Language Options:**
- [日本語 (Japanese)](API_ja_JP.md)

---

## Overview

The SiYuan API provides a RESTful interface for interacting with your knowledge base. You can use it to:

- **Manage Notebooks**: Create, open, close, rename, and delete notebooks
- **Handle Documents**: Create documents from Markdown, organize file structure
- **Manipulate Blocks**: Insert, update, move, and delete content blocks
- **Query Data**: Execute SQL queries against your knowledge base
- **Upload Assets**: Manage images and other media files
- **Export Content**: Convert and export your notes in various formats

## Table of Contents

* [Specification](#Specification)
    * [Parameters and return values](#Parameters-and-return-values)
    * [Authentication](#Authentication)
* [Notebooks](#Notebooks)
    * [List notebooks](#List-notebooks)
    * [Open a notebook](#Open-a-notebook)
    * [Close a notebook](#Close-a-notebook)
    * [Rename a notebook](#Rename-a-notebook)
    * [Create a notebook](#Create-a-notebook)
    * [Remove a notebook](#Remove-a-notebook)
    * [Get notebook configuration](#Get-notebook-configuration)
    * [Save notebook configuration](#Save-notebook-configuration)
* [Documents](#Documents)
    * [Create a document with Markdown](#Create-a-document-with-Markdown)
    * [Duplicate a document](#Duplicate-a-document)
    * [List document tree](#List-document-tree)
    * [Rename a document](#Rename-a-document)
    * [Remove a document](#Remove-a-document)
    * [Move documents](#Move-documents)
    * [Get human-readable path based on path](#Get-human-readable-path-based-on-path)
    * [Get human-readable path based on ID](#Get-human-readable-path-based-on-ID)
    * [Get storage path based on ID](#Get-storage-path-based-on-ID)
    * [Get IDs based on human-readable path](#Get-IDs-based-on-human-readable-path)
    * [Upsert indexes](#Upsert-indexes)
    * [Remove indexes](#Remove-indexes)
* [Assets](#Assets)
    * [Upload assets](#Upload-assets)
* [Blocks](#Blocks)
    * [Insert blocks](#Insert-blocks)
    * [Prepend blocks](#Prepend-blocks)
    * [Append blocks](#Append-blocks)
    * [Update a block](#Update-a-block)
    * [Batch update blocks](#Batch-update-blocks)
    * [Delete a block](#Delete-a-block)
    * [Move a block](#Move-a-block)
    * [Fold a block](#Fold-a-block)
    * [Unfold a block](#Unfold-a-block)
    * [Get a block kramdown](#Get-a-block-kramdown)
    * [Get child blocks](#get-child-blocks)
    * [Get tail child blocks](#Get-tail-child-blocks)
    * [Get block DOMs](#Get-block-DOMs)
    * [Get block tree infos](#Get-block-tree-infos)
    * [Get blocks indexes](#Get-blocks-indexes)
    * [Get docs info](#Get-docs-info)
    * [Append daily note block](#Append-daily-note-block)
    * [Prepend daily note block](#Prepend-daily-note-block)
    * [Transfer block ref](#transfer-block-ref)
* [Attributes](#Attributes)
    * [Set block attributes](#Set-block-attributes)
    * [Batch set block attributes](#Batch-set-block-attributes)
    * [Get block attributes](#Get-block-attributes)
    * [Batch get block attributes](#Batch-get-block-attributes)
* [SQL](#SQL)
    * [Execute SQL query](#Execute-SQL-query)
    * [Flush transaction](#Flush-transaction)
* [Templates](#Templates)
    * [Render a template](#Render-a-template)
    * [Render Sprig](#Render-Sprig)
* [File](#File)
    * [Get file](#Get-file)
    * [Put file](#Put-file)
    * [Remove file](#Remove-file)
    * [Rename file](#Rename-file)
    * [List files](#List-files)
* [Export](#Export)
    * [Export Markdown](#Export-Markdown)
    * [Export Files and Folders](#Export-files-and-folders)
* [Conversion](#Conversion)
    * [Pandoc](#Pandoc)
* [Notification](#Notification)
    * [Push message](#Push-message)
    * [Push error message](#Push-error-message)
* [Network](#Network)
    * [Forward proxy](#Forward-proxy)
* [System](#System)
    * [Get boot progress](#Get-boot-progress)
    * [Get system version](#Get-system-version)
    * [Get the current time of the system](#Get-the-current-time-of-the-system)
* [Archive](#Archive)
    * [Create ZIP archive](#Create-ZIP-archive)
    * [Extract ZIP archive](#Extract-ZIP-archive)
* [Search](#Search)
    * [Get embed block](#Get-embed-block)
    * [Update embed block](#Update-embed-block)
* [References](#References)
    * [Get backlink document](#Get-backlink-document)
* [UI Operations](#UI-Operations)
    * [Reload attribute view](#Reload-attribute-view)
    * [Reload file tree](#Reload-file-tree)
    * [Reload protyle](#Reload-protyle)
    * [Reload tag](#Reload-tag)
    * [Reload UI](#Reload-UI)
* [Settings](#Settings)
    * [Set editor read-only](#Set-editor-read-only)
    * [Add virtual block reference include](#Add-virtual-block-reference-include)
    * [Refresh virtual block reference](#Refresh-virtual-block-reference)
* [Attribute View](#Attribute-View)
    * [Append attribute view detached blocks with values](#Append-attribute-view-detached-blocks-with-values)
    * [Batch replace attribute view blocks](#Batch-replace-attribute-view-blocks)
    * [Batch set attribute view block attributes](#Batch-set-attribute-view-block-attributes)
    * [Get attribute view bound block IDs](#Get-attribute-view-bound-block-IDs)
* [Riff (Spaced Repetition)](#Riff-Spaced-Repetition)
    * [Get riff cards](#Get-riff-cards)
    * [Get riff cards by block IDs](#Get-riff-cards-by-block-IDs)
    * [Batch set riff cards due time](#Batch-set-riff-cards-due-time)
* [Repository](#Repository)
    * [Get repository file](#Get-repository-file)
* [File Operations](#File-Operations)
    * [Global copy files](#Global-copy-files)
* [Icon](#Icon)
    * [Get dynamic icon](#Get-dynamic-icon)
* [Lute](#Lute)
    * [HTML to block DOM](#HTML-to-block-DOM)

---

## API Specification

### Endpoint Configuration

The SiYuan API operates on a local HTTP server:

- **Base URL**: `http://127.0.0.1:6806`
- **Protocol**: HTTP POST only
- **Content-Type**: `application/json`
- **Port**: 6806 (default, configurable in settings)

### Request Format

All API requests must be sent as HTTP POST with JSON payload in the request body:

```bash
curl -X POST \
  http://127.0.0.1:6806/api/notebook/lsNotebooks \
  -H "Content-Type: application/json" \
  -H "Authorization: Token your_api_token_here" \
  -d '{}'
```

### Response Format

All API responses follow a consistent structure:

```json
{
  "code": 0,
  "msg": "",
  "data": {}
}
```

**Response Fields:**
- **`code`** (integer): Status code
  - `0`: Success
  - Non-zero: Error occurred
- **`msg`** (string): Message
  - Empty string on success
  - Error description on failure
- **`data`** (object/array/null): Response payload
  - Varies by endpoint: `{}`, `[]`, or `null`

### Authentication

**How to get your API token:**
1. Open SiYuan application
2. Go to <kbd>Settings</kbd> → <kbd>About</kbd>
3. Copy the API token

**Using the token:**
Add the authorization header to all requests:
```
Authorization: Token your_api_token_here
```

**Example with authentication:**
```javascript
const response = await fetch('http://127.0.0.1:6806/api/notebook/lsNotebooks', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Token your_api_token_here'
  },
  body: JSON.stringify({})
});
```

### Error Handling

When an error occurs, the response will include:
- `code`: Non-zero error code
- `msg`: Descriptive error message
- `data`: Usually `null` or empty

Example error response:
```json
{
  "code": -1,
  "msg": "Notebook not found",
  "data": null
}
```

## Notebooks

Notebooks are the top-level containers for organizing your documents in SiYuan. Each notebook can contain multiple documents and subdirectories, providing a hierarchical structure for your knowledge base.

### List notebooks

Retrieves all available notebooks in your workspace, including their current status (open/closed).

**Endpoint:** `/api/notebook/lsNotebooks`

**Parameters:** None

**Example Request:**
```bash
curl -X POST http://127.0.0.1:6806/api/notebook/lsNotebooks \
  -H "Content-Type: application/json" \
  -H "Authorization: Token your_token" \
  -d '{}'
```

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "notebooks": [
      {
        "id": "20210817205410-2kvfpfn", 
        "name": "Test Notebook",
        "icon": "1f41b",
        "sort": 0,
        "closed": false
      },
      {
        "id": "20210808180117-czj9bvb",
        "name": "SiYuan User Guide",
        "icon": "1f4d4",
        "sort": 1,
        "closed": false
      }
    ]
  }
}
```

**Response Fields:**
- `id`: Unique notebook identifier (timestamp-based)
- `name`: User-defined notebook name
- `icon`: Unicode emoji code for the notebook icon
- `sort`: Display order index
- `closed`: Whether the notebook is currently closed

**Use Cases:**
- Dashboard applications showing available notebooks
- Backup scripts that need to process all notebooks
- Workspace overview tools

### Open a notebook

Opens a closed notebook, making its contents accessible for reading and editing.

**Endpoint:** `/api/notebook/openNotebook`

**Parameters:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (string): The unique ID of the notebook to open

**Example:**
```javascript
const openNotebook = async (notebookId) => {
  const response = await fetch('http://127.0.0.1:6806/api/notebook/openNotebook', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Token your_token'
    },
    body: JSON.stringify({
      notebook: notebookId
    })
  });
  return response.json();
};
```

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**Use Cases:**
- Automatic workspace restoration
- Context-sensitive notebook activation
- Batch operations requiring specific notebooks to be open

### Close a notebook

Closes an open notebook, freeing up resources while preserving the notebook's data.

**Endpoint:** `/api/notebook/closeNotebook`

**Parameters:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (string): The unique ID of the notebook to close

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**Use Cases:**
- Memory optimization for large workspaces
- Project-based workflow management
- Automated cleanup processes

### Rename a notebook

Changes the display name of an existing notebook.

**Endpoint:** `/api/notebook/renameNotebook`

**Parameters:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "name": "New name for notebook"
}
```

- `notebook` (string): Notebook ID
- `name` (string): New name for the notebook

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**Use Cases:**
- Workspace reorganization
- Project rebranding
- Batch renaming operations

### Create a notebook

Creates a new empty notebook in your workspace.

**Endpoint:** `/api/notebook/createNotebook`

**Parameters:**
```json
{
  "name": "Notebook name"
}
```

- `name` (string): Name for the new notebook

**Example:**
```python
import requests

def create_notebook(name, token):
    url = "http://127.0.0.1:6806/api/notebook/createNotebook"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Token {token}"
    }
    payload = {"name": name}
    
    response = requests.post(url, json=payload, headers=headers)
    return response.json()

# Usage
result = create_notebook("My New Project", "your_token_here")
print(f"Created notebook with ID: {result['data']['notebook']['id']}")
```

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "notebook": {
      "id": "20220126215949-r1wvoch",
      "name": "Notebook name",
      "icon": "",
      "sort": 0,
      "closed": false
    }
  }
}
```

**Use Cases:**
- Project initialization scripts
- Template-based notebook creation
- Automated workspace setup

### Remove a notebook

Permanently deletes a notebook and all its contents. ⚠️ **This operation cannot be undone.**

**Endpoint:** `/api/notebook/removeNotebook`

**Parameters:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (string): ID of the notebook to delete

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**⚠️ Warning:** This operation permanently deletes all data in the notebook. Always backup important data before deletion.

**Use Cases:**
- Cleanup scripts for temporary notebooks
- Project archival workflows
- Workspace maintenance

### Get notebook configuration

Retrieves the configuration settings for a specific notebook.

**Endpoint:** `/api/notebook/getNotebookConf`

**Parameters:**
```json
{
  "notebook": "20210817205410-2kvfpfn"
}
```

- `notebook` (string): Notebook ID

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "box": "20210817205410-2kvfpfn",
    "conf": {
      "name": "Test Notebook",
      "closed": false,
      "refCreateSavePath": "",
      "createDocNameTemplate": "",
      "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
      "dailyNoteTemplatePath": ""
    },
    "name": "Test Notebook"
  }
}
```

**Configuration Fields:**
- `name`: Notebook display name
- `closed`: Whether the notebook is closed
- `refCreateSavePath`: Default path for creating reference documents
- `createDocNameTemplate`: Template for new document names
- `dailyNoteSavePath`: Path template for daily notes (supports date formatting)
- `dailyNoteTemplatePath`: Path to template used for daily notes

**Use Cases:**
- Backup and restore notebook settings
- Synchronizing configuration across instances
- Building configuration management tools

### Save notebook configuration

Updates the configuration settings for a specific notebook.

**Endpoint:** `/api/notebook/setNotebookConf`

**Parameters:**
```json
{
  "notebook": "20210817205410-2kvfpfn",
  "conf": {
    "name": "Test Notebook",
    "closed": false,
    "refCreateSavePath": "",
    "createDocNameTemplate": "",
    "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
    "dailyNoteTemplatePath": ""
  }
}
```

- `notebook` (string): Notebook ID
- `conf` (object): Configuration object with settings to update

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "name": "Test Notebook",
    "closed": false,
    "refCreateSavePath": "",
    "createDocNameTemplate": "",
    "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
    "dailyNoteTemplatePath": ""
  }
}
```

**Example - Setting up daily notes:**
```javascript
const setupDailyNotes = async (notebookId, token) => {
  const conf = {
    name: "My Daily Journal",
    closed: false,
    refCreateSavePath: "/references",
    createDocNameTemplate: "{{now | date \"2006-01-02\"}} - {{.title}}",
    dailyNoteSavePath: "/daily/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
    dailyNoteTemplatePath: "/templates/daily-template"
  };
  
  const response = await fetch('http://127.0.0.1:6806/api/notebook/setNotebookConf', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Token ${token}`
    },
    body: JSON.stringify({
      notebook: notebookId,
      conf: conf
    })
  });
  
  return response.json();
};
```

**Use Cases:**
- Automated workspace setup
- Template-based notebook configuration
- Bulk configuration updates

---

## Documents

Documents are individual notes or files within notebooks. They can contain various types of content including text, images, tables, and complex block structures. The Documents API allows you to create, manage, and organize your knowledge base content.

### Create a document with Markdown

Creates a new document from Markdown content. This is useful for importing existing notes or creating documents programmatically.

**Endpoint:** `/api/filetree/createDocWithMd`

**Parameters:**
```json
{
  "notebook": "20210817205410-2kvfpfn",
  "path": "/foo/bar",
  "markdown": "# Hello World\n\nThis is my first document."
}
```

- `notebook` (string): Target notebook ID
- `path` (string): Document path within the notebook, must start with `/` and separate levels with `/` (corresponds to database `hpath` field)
- `markdown` (string): GFM (GitHub Flavored Markdown) content for the document

**Example:**
```python
def create_markdown_document(notebook_id, path, content, token):
    """Create a new document from markdown content"""
    import requests
    
    url = "http://127.0.0.1:6806/api/filetree/createDocWithMd"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Token {token}"
    }
    
    payload = {
        "notebook": notebook_id,
        "path": path,
        "markdown": content
    }
    
    response = requests.post(url, json=payload, headers=headers)
    return response.json()

# Usage example
markdown_content = """# Project Overview

## Goals
- Complete the API documentation
- Add Japanese translation
- Improve user experience

## Timeline
- Week 1: Documentation enhancement
- Week 2: Translation work
- Week 3: Review and testing
"""

result = create_markdown_document(
    "20210817205410-2kvfpfn", 
    "/projects/api-documentation", 
    markdown_content,
    "your_token_here"
)
```

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": "20210914223645-oj2vnx2"
}
```

- `data`: ID of the created document

**Important Notes:**
- If you call this endpoint repeatedly with the same `path`, existing documents will not be overwritten
- The path should not include the `.sy` file extension
- Markdown content is converted to SiYuan's internal block structure

**Use Cases:**
- Importing content from other note-taking applications
- Automated content generation
- Template-based document creation
- Batch document creation from external sources

### Rename a document

Changes the title of an existing document. You can rename by document path or by document ID.

**Method 1: Rename by path**

**Endpoint:** `/api/filetree/renameDoc`

**Parameters:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210902210113-0avi12f.sy",
  "title": "New document title"
}
```

- `notebook` (string): Notebook ID containing the document
- `path` (string): Full document path including `.sy` extension
- `title` (string): New title for the document

**Method 2: Rename by ID**

**Endpoint:** `/api/filetree/renameDocByID`

**Parameters:**
```json
{
  "id": "20210902210113-0avi12f",
  "title": "New document title"
}
```

- `id` (string): Document ID
- `title` (string): New title for the document

**Response (both methods):**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**Example:**
```javascript
const renameDocument = async (documentId, newTitle, token) => {
  const response = await fetch('http://127.0.0.1:6806/api/filetree/renameDocByID', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Token ${token}`
    },
    body: JSON.stringify({
      id: documentId,
      title: newTitle
    })
  });
  
  return response.json();
};

// Usage
await renameDocument("20210902210113-0avi12f", "My Updated Document Title", "your_token");
```

**Use Cases:**
- Document organization and restructuring
- Batch renaming operations
- Content management workflows
- Template-based naming conventions

### Remove a document

Permanently deletes a document and all its content. ⚠️ **This operation cannot be undone.**

**Method 1: Remove by path**

**Endpoint:** `/api/filetree/removeDoc`

**Parameters:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210902210113-0avi12f.sy"
}
```

- `notebook` (string): Notebook ID containing the document
- `path` (string): Full document path including `.sy` extension

**Method 2: Remove by ID**

**Endpoint:** `/api/filetree/removeDocByID`

**Parameters:**
```json
{
  "id": "20210902210113-0avi12f"
}
```

- `id` (string): Document ID to remove

**Response (both methods):**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**⚠️ Warning:** This operation permanently deletes the document and all its content. Always backup important data before deletion.

**Example:**
```bash
# Remove document by ID
curl -X POST http://127.0.0.1:6806/api/filetree/removeDocByID \
  -H "Content-Type: application/json" \
  -H "Authorization: Token your_token" \
  -d '{"id": "20210902210113-0avi12f"}'
```

**Use Cases:**
- Cleanup scripts for temporary documents
- Content management workflows
- Batch deletion operations
- Document lifecycle management

### Move documents

Moves documents between different locations within the same notebook or to different notebooks.

**Method 1: Move by paths**

**Endpoint:** `/api/filetree/moveDocs`

**Parameters:**
```json
{
  "fromPaths": ["/20210917220056-yxtyl7i.sy"],
  "toNotebook": "20210817205410-2kvfpfn",
  "toPath": "/"
}
```

- `fromPaths` (array): Array of source document paths
- `toNotebook` (string): Target notebook ID
- `toPath` (string): Target path within the notebook

**Method 2: Move by IDs**

**Endpoint:** `/api/filetree/moveDocsByID`

**Parameters:**
```json
{
  "fromIDs": ["20210917220056-yxtyl7i"],
  "toID": "20210817205410-2kvfpfn"
}
```

- `fromIDs` (array): Array of source document IDs
- `toID` (string): Target parent document ID or notebook ID

**Response (both methods):**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**Example - Organizing documents into folders:**
```javascript
const organizeDocuments = async (token) => {
  // Move multiple documents to a project folder
  const response = await fetch('http://127.0.0.1:6806/api/filetree/moveDocsByID', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Token ${token}`
    },
    body: JSON.stringify({
      fromIDs: [
        "20210917220056-yxtyl7i",
        "20210918110023-abc123f",
        "20210919140055-def456g"
      ],
      toID: "20210920000000-project1"  // Parent folder ID
    })
  });
  
  return response.json();
};
```

**Use Cases:**
- Document organization and restructuring
- Project-based content management
- Cross-notebook document migration
- Automated content categorization

### Get human-readable path based on path

* `/api/filetree/getHPathByPath`
* Parameters

  ```json
  {
    "notebook": "20210831090520-7dvbdv0",
    "path": "/20210917220500-sz588nq/20210917220056-yxtyl7i.sy"
  }
  ```

    * `notebook`: Notebook ID
    * `path`: Document path
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": "/foo/bar"
  }
  ```

### Get human-readable path based on ID

* `/api/filetree/getHPathByID`
* Parameters

  ```json
  {
    "id": "20210917220056-yxtyl7i"
  }
  ```

    * `id`: Block ID
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": "/foo/bar"
  }
  ```

### Get storage path based on ID

* `/api/filetree/getPathByID`
* Parameters

  ```json
  {
    "id": "20210808180320-fqgskfj"
  }
  ```

    * `id`: Block ID
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
    "notebook": "20210808180117-czj9bvb",
    "path": "/20200812220555-lj3enxa/20210808180320-fqgskfj.sy"
    }
  }
  ```

### Get IDs based on human-readable path

* `/api/filetree/getIDsByHPath`
* Parameters

  ```json
  {
    "path": "/foo/bar",
    "notebook": "20210808180117-czj9bvb"
  }
  ```

    * `path`: Human-readable path
    * `notebook`: Notebook ID
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
        "20200813004931-q4cu8na"
    ]
  }
  ```

## Assets

### Upload assets

* `/api/asset/upload`
* The parameter is an HTTP Multipart form

    * `assetsDirPath`: The folder path where assets are stored, with the data folder as the root path, for example:
        * `"/assets/"`: workspace/data/assets/ folder
        * `"/assets/sub/"`: workspace/data/assets/sub/ folder

      Under normal circumstances, it is recommended to use the first method, which is stored in the assets folder of the
      workspace, putting in a subdirectory has some side effects, please refer to the assets chapter of the user guide.
    * `file[]`: Uploaded file list
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "errFiles": [""],
      "succMap": {
        "foo.png": "assets/foo-20210719092549-9j5y79r.png"
      }
    }
  }
  ```

    * `errFiles`: List of filenames with errors in upload processing
    * `succMap`: For successfully processed files, the key is the file name when uploading, and the value is
      assets/foo-id.png, which is used to replace the asset link address in the existing Markdown content with the
      uploaded address

## Blocks

### Insert blocks

* `/api/block/insertBlock`
* Parameters

  ```json
  {
    "dataType": "markdown",
    "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
    "nextID": "",
    "previousID": "20211229114650-vrek5x6",
    "parentID": ""
  }
  ```

    * `dataType`: The data type to be inserted, the value can be `markdown` or `dom`
    * `data`: Data to be inserted
    * `nextID`: The ID of the next block, used to anchor the insertion position
    * `previousID`: The ID of the previous block, used to anchor the insertion position
    * `parentID`: The ID of the parent block, used to anchor the insertion position

  `nextID`, `previousID`, and `parentID` must have at least one value, using priority: `nextID` > `previousID` >
  `parentID`
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
      {
        "doOperations": [
          {
            "action": "insert",
            "data": "<div data-node-id=\"20211230115020-g02dfx0\" data-node-index=\"1\" data-type=\"NodeParagraph\" class=\"p\"><div contenteditable=\"true\" spellcheck=\"false\">foo<strong style=\"color: var(--b3-font-color8);\">bar</strong>baz</div><div class=\"protyle-attr\" contenteditable=\"false\"></div></div>",
            "id": "20211230115020-g02dfx0",
            "parentID": "",
            "previousID": "20211229114650-vrek5x6",
            "retData": null
          }
        ],
        "undoOperations": null
      }
    ]
  }
  ```

    * `action.data`: DOM generated by the newly inserted block
    * `action.id`: ID of the newly inserted block

### Prepend blocks

* `/api/block/prependBlock`
* Parameters

  ```json
  {
    "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
    "dataType": "markdown",
    "parentID": "20220107173950-7f9m1nb"
  }
  ```

    * `dataType`: The data type to be inserted, the value can be `markdown` or `dom`
    * `data`: Data to be inserted
    * `parentID`: The ID of the parent block, used to anchor the insertion position
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
      {
        "doOperations": [
          {
            "action": "insert",
            "data": "<div data-node-id=\"20220108003710-hm0x9sc\" data-node-index=\"1\" data-type=\"NodeParagraph\" class=\"p\"><div contenteditable=\"true\" spellcheck=\"false\">foo<strong style=\"color: var(--b3-font-color8);\">bar</strong>baz</div><div class=\"protyle-attr\" contenteditable=\"false\"></div></div>",
            "id": "20220108003710-hm0x9sc",
            "parentID": "20220107173950-7f9m1nb",
            "previousID": "",
            "retData": null
          }
        ],
        "undoOperations": null
      }
    ]
  }
  ```

    * `action.data`: DOM generated by the newly inserted block
    * `action.id`: ID of the newly inserted block

### Append blocks

* `/api/block/appendBlock`
* Parameters

  ```json
  {
    "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
    "dataType": "markdown",
    "parentID": "20220107173950-7f9m1nb"
  }
  ```

    * `dataType`: The data type to be inserted, the value can be `markdown` or `dom`
    * `data`: Data to be inserted
    * `parentID`: The ID of the parent block, used to anchor the insertion position
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
      {
        "doOperations": [
          {
            "action": "insert",
            "data": "<div data-node-id=\"20220108003642-y2wmpcv\" data-node-index=\"1\" data-type=\"NodeParagraph\" class=\"p\"><div contenteditable=\"true\" spellcheck=\"false\">foo<strong style=\"color: var(--b3-font-color8);\">bar</strong>baz</div><div class=\"protyle-attr\" contenteditable=\"false\"></div></div>",
            "id": "20220108003642-y2wmpcv",
            "parentID": "20220107173950-7f9m1nb",
            "previousID": "20220108003615-7rk41t1",
            "retData": null
          }
        ],
        "undoOperations": null
      }
    ]
  }
  ```

    * `action.data`: DOM generated by the newly inserted block
    * `action.id`: ID of the newly inserted block

### Update a block

* `/api/block/updateBlock`
* Parameters

  ```json
  {
    "dataType": "markdown",
    "data": "foobarbaz",
    "id": "20211230161520-querkps"
  }
  ```

    * `dataType`: The data type to be updated, the value can be `markdown` or `dom`
    * `data`: Data to be updated
    * `id`: ID of the block to be updated
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
      {
        "doOperations": [
          {
            "action": "update",
            "data": "<div data-node-id=\"20211230161520-querkps\" data-node-index=\"1\" data-type=\"NodeParagraph\" class=\"p\"><div contenteditable=\"true\" spellcheck=\"false\">foo<strong>bar</strong>baz</div><div class=\"protyle-attr\" contenteditable=\"false\"></div></div>",
            "id": "20211230161520-querkps",
            "parentID": "",
            "previousID": "",
            "retData": null
            }
          ],
        "undoOperations": null
      }
    ]
  }
  ```

    * `action.data`: DOM generated by the updated block

### Delete a block

* `/api/block/deleteBlock`
* Parameters

  ```json
  {
    "id": "20211230161520-querkps"
  }
  ```

    * `id`: ID of the block to be deleted
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
      {
        "doOperations": [
          {
            "action": "delete",
            "data": null,
            "id": "20211230162439-vtm09qo",
            "parentID": "",
            "previousID": "",
            "retData": null
          }
        ],
       "undoOperations": null
      }
    ]
  }
  ```

### Move a block

* `/api/block/moveBlock`
* Parameters

  ```json
  {
    "id": "20230406180530-3o1rqkc",
    "previousID": "20230406152734-if5kyx6",
    "parentID": "20230404183855-woe52ko"
  }
  ```

    * `id`: Block ID to move
    * `previousID`: The ID of the previous block, used to anchor the insertion position
    * `parentID`: The ID of the parent block, used to anchor the insertion position, `previousID` and `parentID` cannot
      be empty at the same time, if they exist at the same time, `previousID` will be used first
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
        {
            "doOperations": [
                {
                    "action": "move",
                    "data": null,
                    "id": "20230406180530-3o1rqkc",
                    "parentID": "20230404183855-woe52ko",
                    "previousID": "20230406152734-if5kyx6",
                    "nextID": "",
                    "retData": null,
                    "srcIDs": null,
                    "name": "",
                    "type": ""
                }
            ],
            "undoOperations": null
        }
    ]
  }
  ```

### Fold a block

* `/api/block/foldBlock`
* Parameters

  ```json
  {
    "id": "20231224160424-2f5680o"
  }
  ```

    * `id`: Block ID to fold
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### Unfold a block

* `/api/block/unfoldBlock`
* Parameters

  ```json
  {
    "id": "20231224160424-2f5680o"
  }
  ```

    * `id`: Block ID to unfold
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### Get a block kramdown

* `/api/block/getBlockKramdown`
* Parameters

  ```json
  {
    "id": "20201225220954-dlgzk1o"
  }
  ```

    * `id`: ID of the block to be got
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "id": "20201225220954-dlgzk1o",
      "kramdown": "* {: id=\"20201225220954-e913snx\"}Create a new notebook, create a new document under the notebook\n  {: id=\"20210131161940-kfs31q6\"}\n* {: id=\"20201225220954-ygz217h\"}Enter <kbd>/</kbd> in the editor to trigger the function menu\n  {: id=\"20210131161940-eo0riwq\"}\n* {: id=\"20201225220954-875yybt\"}((20200924101200-gss5vee \"Navigate in the content block\")) and ((20200924100906-0u4zfq3 \"Window and tab\"))\n  {: id=\"20210131161940-b5uow2h\"}"
    }
  }
  ```

### Get child blocks

* `/api/block/getChildBlocks`
* Parameters

  ```json
  {
    "id": "20230506212712-vt9ajwj"
  }
  ```

    * `id`: Parent block ID
    * The blocks below a heading are also counted as child blocks
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
      {
        "id": "20230512083858-mjdwkbn",
        "type": "h",
        "subType": "h1"
      },
      {
        "id": "20230513213727-thswvfd",
        "type": "s"
      },
      {
        "id": "20230513213633-9lsj4ew",
        "type": "l",
        "subType": "u"
      }
    ]
  }
  ```

### Transfer block ref

* `/api/block/transferBlockRef`
* Parameters

  ```json
  {
    "fromID": "20230612160235-mv6rrh1",
    "toID": "20230613093045-uwcomng",
    "refIDs": ["20230613092230-cpyimmd"]
  }
  ```

    * `fromID`: Def block ID
    * `toID`: Target block ID
    * `refIDs`: Ref block IDs point to def block ID, optional, if not specified, all ref block IDs will be transferred
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## Attributes

### Set block attributes

* `/api/attr/setBlockAttrs`
* Parameters

  ```json
  {
    "id": "20210912214605-uhi5gco",
    "attrs": {
      "custom-attr1": "line1\nline2"
    }
  }
  ```

    * `id`: Block ID
    * `attrs`: Block attributes, custom attributes must be prefixed with `custom-`
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### Get block attributes

* `/api/attr/getBlockAttrs`
* Parameters

  ```json
  {
    "id": "20210912214605-uhi5gco"
  }
  ```

    * `id`: Block ID
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "custom-attr1": "line1\nline2",
      "id": "20210912214605-uhi5gco",
      "title": "PDF Annotation Demo",
      "type": "doc",
      "updated": "20210916120715"
    }
  }
  ```

## SQL

### Execute SQL query

* `/api/query/sql`
* Parameters

  ```json
  {
    "stmt": "SELECT * FROM blocks WHERE content LIKE'%content%' LIMIT 7"
  }
  ```

    * `stmt`: SQL statement
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
      { "col": "val" }
    ]
  }
  ```

### Flush transaction

* `/api/sqlite/flushTransaction`
* No parameters
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## Templates

### Render a template

* `/api/template/render`
* Parameters

  ```json
  {
    "id": "20220724223548-j6g0o87",
    "path": "F:\\SiYuan\\data\\templates\\foo.md"
  }
  ```

    * `id`: The ID of the document where the rendering is called
    * `path`: Template file absolute path
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "content": "<div data-node-id=\"20220729234848-dlgsah7\" data-node-index=\"1\" data-type=\"NodeParagraph\" class=\"p\" updated=\"20220729234840\"><div contenteditable=\"true\" spellcheck=\"false\">foo</div><div class=\"protyle-attr\" contenteditable=\"false\">​</div></div>",
      "path": "F:\\SiYuan\\data\\templates\\foo.md"
    }
  }
  ```

### Render Sprig

* `/api/template/renderSprig`
* Parameters

  ```json
  {
    "template": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}"
  }
  ```
    * `template`: template content
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": "/daily note/2023/03/2023-03-24"
  }
  ```

## File

### Get file

* `/api/file/getFile`
* Parameters

  ``json {
  "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p.sy"
  }
  ``
    * `path`: the file path under the workspace path
* Return value

    * Response status code `200`: File content
    * Response status code `202`: Exception information

      ```json
      {
        "code": 404,
        "msg": "",
        "data": null
      }
      ```

        * `code`: non-zero for exceptions

            * `-1`: Parameter parsing error
            * `403`: Permission denied (file is not in the workspace)
            * `404`: Not Found (file doesn't exist)
            * `405`: Method Not Allowed (it's a directory)
            * `500`: Server Error (stat file failed / read file failed)
        * `msg`: a piece of text describing the error

### Put file

* `/api/file/putFile`
* The parameter is an HTTP Multipart form

    * `path`: the file path under the workspace path
    * `isDir`: whether to create a folder, when `true` only create a folder, ignore `file`
    * `modTime`: last access and modification time, Unix time
    * `file`: the uploaded file
* Return value

   ```json
   {
     "code": 0,
     "msg": "",
     "data": null
   }
   ```

### Remove file

* `/api/file/removeFile`
* Parameters

  ```json
  {
    "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p.sy"
  }
  ```
    * `path`: the file path under the workspace path
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### Rename file

* `/api/file/renameFile`
* Parameters

  ```json
  {
    "path": "/data/assets/image-20230523085812-k3o9t32.png",
    "newPath": "/data/assets/test-20230523085812-k3o9t32.png"
  }
  ```
    * `path`: the file path under the workspace path
    * `newPath`: the new file path under the workspace path
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### List files

* `/api/file/readDir`
* Parameters

  ```json
  {
    "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p"
  }
  ```
    * `path`: the dir path under the workspace path
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
      {
        "isDir": true,
        "isSymlink": false,
        "name": "20210808180303-6yi0dv5",
        "updated": 1691467624
      },
      {
        "isDir": false,
        "isSymlink": false,
        "name": "20210808180303-6yi0dv5.sy",
        "updated": 1663298365
      }
    ]
  }
  ```

## Export

### Export Markdown

* `/api/export/exportMdContent`
* Parameters

  ```json
  {
    "id": ""
  }
  ```

    * `id`: ID of the doc block to export
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "hPath": "/Please Start Here",
      "content": "## 🍫 Content Block\n\nIn SiYuan, the only important core concept is..."
    }
  }
  ```

    * `hPath`: human-readable path
    * `content`: Markdown content

### Export files and folders

* `/api/export/exportResources`
* Parameters

  ```json
  {
    "paths": [
      "/conf/appearance/boot",
      "/conf/appearance/langs",
      "/conf/appearance/emojis/conf.json",
      "/conf/appearance/icons/index.html"
    ],
    "name": "zip-file-name"
  }
  ```

    * `paths`: A list of file or folder paths to be exported, the same filename/folder name will be overwritten
    * `name`: (Optional) The exported file name, which defaults to `export-YYYY-MM-DD_hh-mm-ss.zip` when not set
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "path": "temp/export/zip-file-name.zip"
    }
  }
  ```

    * `path`: The path of `*.zip` file created
        * The directory structure in `zip-file-name.zip` is as follows:
            * `zip-file-name`
                * `boot`
                * `langs`
                * `conf.json`
                * `index.html`

## Conversion

### Pandoc

* `/api/convert/pandoc`
* Working directory
    * Executing the pandoc command will set the working directory to `workspace/temp/convert/pandoc/${dir}`
    * API [`Put file`](#put-file) can be used to write the file to be converted to this directory first
    * Then call the API for conversion, and the converted file will also be written to this directory
    * Finally, call the API [`Get file`](#get-file) to get the converted file
        * Or call the API [Create a document with Markdown](#Create-a-document-with-Markdown)
        * Or call the internal API `importStdMd` to import the converted folder directly
* Parameters

  ```json
  {
    "dir": "test",
    "args": [
      "--to", "markdown_strict-raw_html",
      "foo.epub",
      "-o", "foo.md"
   ]
  }
  ```

    * `args`: Pandoc command line parameters
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
       "path": "/temp/convert/pandoc/test"
    }
  }
  ```
    * `path`: the path under the workspace

## Notification

### Push message

* `/api/notification/pushMsg`
* Parameters

  ```json
  {
    "msg": "test",
    "timeout": 7000
  }
  ```
    * `timeout`: The duration of the message display in milliseconds. This field can be omitted, the default is 7000
      milliseconds
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
        "id": "62jtmqi"
    }
  }
  ```
    * `id`: Message ID

### Push error message

* `/api/notification/pushErrMsg`
* Parameters

  ```json
  {
    "msg": "test",
    "timeout": 7000
  }
  ```
    * `timeout`: The duration of the message display in milliseconds. This field can be omitted, the default is 7000
      milliseconds
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
        "id": "qc9znut"
    }
  }
  ```
    * `id`: Message ID

## Network

### Forward proxy

* `/api/network/forwardProxy`
* Parameters

  ```json
  {
    "url": "https://b3log.org/siyuan/",
    "method": "GET",
    "timeout": 7000,
    "contentType": "text/html",
    "headers": [
        {
            "Cookie": ""
        }
    ],
    "payload": {},
    "payloadEncoding": "text",
    "responseEncoding": "text"
  }
  ```

    * `url`: URL to forward
    * `method`: HTTP method, default is `POST`
    * `timeout`: timeout in milliseconds, default is `7000`
    * `contentType`: Content-Type, default is `application/json`
    * `headers`: HTTP headers
    * `payload`: HTTP payload, object or string
    * `payloadEncoding`: The encoding scheme used by `pyaload`, default is `text`, optional values are as follows

        * `text`
        * `base64` | `base64-std`
        * `base64-url`
        * `base32` | `base32-std`
        * `base32-hex`
        * `hex`
    * `responseEncoding`: The encoding scheme used by `body` in response data, default is `text`, optional values are as
      follows

        * `text`
        * `base64` | `base64-std`
        * `base64-url`
        * `base32` | `base32-std`
        * `base32-hex`
        * `hex`
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "body": "",
      "bodyEncoding": "text",
      "contentType": "text/html",
      "elapsed": 1976,
      "headers": {
      },
      "status": 200,
      "url": "https://b3log.org/siyuan"
    }
  }
  ```

    * `bodyEncoding`: The encoding scheme used by `body`, is consistent with field `responseEncoding` in request,
      default is `text`, optional values are as follows

        * `text`
        * `base64` | `base64-std`
        * `base64-url`
        * `base32` | `base32-std`
        * `base32-hex`
        * `hex`

## System

### Get boot progress

* `/api/system/bootProgress`
* No parameters
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "details": "Finishing boot...",
      "progress": 100
    }
  }
  ```

### Get system version

* `/api/system/version`
* No parameters
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": "1.3.5"
  }
  ```

### Get the current time of the system

* `/api/system/currentTime`
* No parameters
* Return value

  ```json
  {
    "code": 0,
    "msg": "",
    "data": 1631850968131
  }
  ```

    * `data`: Precision in milliseconds

---

## Additional API Endpoints

The following sections document additional SiYuan API endpoints that have been added in recent versions and provide extended functionality for advanced use cases.

## Archive

### Create ZIP archive

Creates a ZIP archive from specified files and folders.

**Endpoint:** `/api/archive/zip`

**Parameters:**
```json
{
  "paths": ["/path/to/file1", "/path/to/folder"],
  "name": "archive-name"
}
```

- `paths` (array): Array of file/folder paths to include in the archive
- `name` (string): Name for the ZIP file (optional)

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "path": "/path/to/archive.zip"
  }
}
```

### Extract ZIP archive

Extracts a ZIP archive to a specified location.

**Endpoint:** `/api/archive/unzip`

**Parameters:**
```json
{
  "path": "/path/to/archive.zip",
  "dest": "/extraction/destination"
}
```

- `path` (string): Path to the ZIP file to extract
- `dest` (string): Destination folder for extraction

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

## Search

### Get embed block

Retrieves information about an embedded block.

**Endpoint:** `/api/search/getEmbedBlock`

**Parameters:**
```json
{
  "id": "20210808180320-fqgskfj",
  "includeIDs": ["20210808180320-abc123"]
}
```

- `id` (string): Block ID to search for
- `includeIDs` (array): Array of block IDs to include in results

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "block": {
      "id": "20210808180320-fqgskfj",
      "content": "Block content here"
    }
  }
}
```

### Update embed block

Updates an embedded block's content.

**Endpoint:** `/api/search/updateEmbedBlock`

**Parameters:**
```json
{
  "id": "20210808180320-fqgskfj",
  "content": "Updated block content"
}
```

- `id` (string): Block ID to update
- `content` (string): New content for the block

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

## References

### Get backlink document

Retrieves the document containing backlinks to a specific block.

**Endpoint:** `/api/ref/getBacklinkDoc`

**Parameters:**
```json
{
  "id": "20210808180320-fqgskfj"
}
```

- `id` (string): Block ID to find backlinks for

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "backlinks": [
      {
        "id": "20210809123456-abc123",
        "title": "Document Title",
        "path": "/path/to/document"
      }
    ]
  }
}
```

## UI Operations

### Reload attribute view

Reloads the attribute view in the UI.

**Endpoint:** `/api/ui/reloadAttributeView`

**Parameters:**
```json
{
  "id": "20210808180320-fqgskfj"
}
```

- `id` (string): Attribute view ID to reload

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Reload file tree

Reloads the file tree in the UI.

**Endpoint:** `/api/ui/reloadFiletree`

**Parameters:** None

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Reload protyle

Reloads the protyle editor.

**Endpoint:** `/api/ui/reloadProtyle`

**Parameters:**
```json
{
  "id": "20210808180320-fqgskfj"
}
```

- `id` (string): Block ID to reload in protyle

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Reload tag

Reloads the tag panel in the UI.

**Endpoint:** `/api/ui/reloadTag`

**Parameters:** None

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Reload UI

Reloads the entire UI.

**Endpoint:** `/api/ui/reloadUI`

**Parameters:** None

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

## Settings

### Set editor read-only

Sets the editor to read-only mode.

**Endpoint:** `/api/setting/setEditorReadOnly`

**Parameters:**
```json
{
  "readOnly": true
}
```

- `readOnly` (boolean): Whether to enable read-only mode

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Add virtual block reference include

Adds a virtual block reference include pattern.

**Endpoint:** `/api/setting/addVirtualBlockRefInclude`

**Parameters:**
```json
{
  "pattern": "*.md"
}
```

- `pattern` (string): File pattern to include for virtual block references

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Refresh virtual block reference

Refreshes virtual block references.

**Endpoint:** `/api/setting/refreshVirtualBlockRef`

**Parameters:** None

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

## Attribute View

### Append attribute view detached blocks with values

Appends detached blocks to an attribute view with specific values.

**Endpoint:** `/api/av/appendAttributeViewDetachedBlocksWithValues`

**Parameters:**
```json
{
  "avID": "20210808180320-fqgskfj",
  "blockIDs": ["20210808180320-abc123"],
  "values": {
    "column1": "value1",
    "column2": "value2"
  }
}
```

- `avID` (string): Attribute view ID
- `blockIDs` (array): Array of block IDs to append
- `values` (object): Key-value pairs for attribute values

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Batch replace attribute view blocks

Replaces multiple blocks in an attribute view.

**Endpoint:** `/api/av/batchReplaceAttributeViewBlocks`

**Parameters:**
```json
{
  "avID": "20210808180320-fqgskfj",
  "blockIDs": ["20210808180320-abc123"],
  "replacements": ["20210808180320-def456"]
}
```

- `avID` (string): Attribute view ID
- `blockIDs` (array): Array of block IDs to replace
- `replacements` (array): Array of replacement block IDs

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Batch set attribute view block attributes

Sets attributes for multiple blocks in an attribute view.

**Endpoint:** `/api/av/batchSetAttributeViewBlockAttrs`

**Parameters:**
```json
{
  "avID": "20210808180320-fqgskfj",
  "blockIDs": ["20210808180320-abc123"],
  "attrs": {
    "custom-attr1": "value1"
  }
}
```

- `avID` (string): Attribute view ID
- `blockIDs` (array): Array of block IDs to update
- `attrs` (object): Attributes to set

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Get attribute view bound block IDs

Gets the block IDs bound to an attribute view.

**Endpoint:** `/api/av/getAttributeViewBoundBlockIDs`

**Parameters:**
```json
{
  "avID": "20210808180320-fqgskfj"
}
```

- `avID` (string): Attribute view ID

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "blockIDs": ["20210808180320-abc123", "20210808180320-def456"]
  }
}
```

## Riff (Spaced Repetition)

### Get riff cards

Retrieves spaced repetition cards.

**Endpoint:** `/api/riff/getRiffCards`

**Parameters:**
```json
{
  "deckID": "20210808180320-fqgskfj"
}
```

- `deckID` (string): Deck ID to get cards from

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "cards": [
      {
        "id": "20210808180320-abc123",
        "blockID": "20210808180320-def456",
        "due": 1631850968131
      }
    ]
  }
}
```

### Get riff cards by block IDs

Retrieves spaced repetition cards for specific block IDs.

**Endpoint:** `/api/riff/getRiffCardsByBlockIDs`

**Parameters:**
```json
{
  "blockIDs": ["20210808180320-abc123", "20210808180320-def456"]
}
```

- `blockIDs` (array): Array of block IDs to get cards for

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "cards": [
      {
        "id": "20210808180320-abc123",
        "blockID": "20210808180320-def456",
        "due": 1631850968131
      }
    ]
  }
}
```

### Batch set riff cards due time

Sets the due time for multiple riff cards.

**Endpoint:** `/api/riff/batchSetRiffCardsDueTime`

**Parameters:**
```json
{
  "cardIDs": ["20210808180320-abc123"],
  "dueTime": 1631850968131
}
```

- `cardIDs` (array): Array of card IDs to update
- `dueTime` (number): New due time (Unix timestamp in milliseconds)

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

## Repository

### Get repository file

Retrieves a file from the repository.

**Endpoint:** `/api/repo/getRepoFile`

**Parameters:**
```json
{
  "path": "/path/to/file.md"
}
```

- `path` (string): Path to the file in the repository

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "content": "File content here"
  }
}
```

## File Operations

### Global copy files

Copies files globally across the system.

**Endpoint:** `/api/file/globalCopyFiles`

**Parameters:**
```json
{
  "paths": ["/source/file1.md", "/source/file2.md"],
  "dest": "/destination/folder"
}
```

- `paths` (array): Array of source file paths
- `dest` (string): Destination folder path

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

## Icon

### Get dynamic icon

Retrieves a dynamic icon.

**Endpoint:** `/api/icon/getDynamicIcon`

**Parameters:**
```json
{
  "type": "emoji",
  "id": "1f41b"
}
```

- `type` (string): Type of icon (e.g., "emoji")
- `id` (string): Icon identifier

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "icon": "🐛"
  }
}
```

## Lute

### HTML to block DOM

Converts HTML content to SiYuan block DOM structure.

**Endpoint:** `/api/lute/html2BlockDOM`

**Parameters:**
```json
{
  "html": "<p>Hello <strong>world</strong></p>"
}
```

- `html` (string): HTML content to convert

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "dom": "<div data-node-id=\"20210808180320-abc123\" data-type=\"NodeParagraph\" class=\"p\"><div contenteditable=\"true\" spellcheck=\"false\">Hello <strong>world</strong></div></div>"
  }
}
```

---

## Missing Block Operations

Let me also add the missing block operations that were identified:

### Batch update blocks

Updates multiple blocks in a single operation.

**Endpoint:** `/api/block/batchUpdateBlock`

**Parameters:**
```json
{
  "blocks": [
    {
      "id": "20210808180320-abc123",
      "data": "Updated content 1",
      "dataType": "markdown"
    },
    {
      "id": "20210808180320-def456", 
      "data": "Updated content 2",
      "dataType": "markdown"
    }
  ]
}
```

- `blocks` (array): Array of block objects to update
  - `id` (string): Block ID
  - `data` (string): New content
  - `dataType` (string): Data type ("markdown" or "dom")

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Get tail child blocks

Gets the last child blocks of a parent block.

**Endpoint:** `/api/block/getTailChildBlocks`

**Parameters:**
```json
{
  "id": "20210808180320-fqgskfj"
}
```

- `id` (string): Parent block ID

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "id": "20210808180320-abc123",
      "type": "p"
    }
  ]
}
```

### Get block DOMs

Gets the DOM representation of blocks.

**Endpoint:** `/api/block/getBlockDOMs`

**Parameters:**
```json
{
  "ids": ["20210808180320-abc123", "20210808180320-def456"]
}
```

- `ids` (array): Array of block IDs

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "doms": [
      {
        "id": "20210808180320-abc123",
        "dom": "<div>Block DOM here</div>"
      }
    ]
  }
}
```

### Get block tree infos

Gets tree information for blocks.

**Endpoint:** `/api/block/getBlockTreeInfos`

**Parameters:**
```json
{
  "ids": ["20210808180320-abc123"]
}
```

- `ids` (array): Array of block IDs

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "id": "20210808180320-abc123",
      "type": "p",
      "children": []
    }
  ]
}
```

### Get blocks indexes

Gets index information for blocks.

**Endpoint:** `/api/block/getBlocksIndexes`

**Parameters:**
```json
{
  "ids": ["20210808180320-abc123"]
}
```

- `ids` (array): Array of block IDs

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "indexes": [
      {
        "id": "20210808180320-abc123",
        "index": 0
      }
    ]
  }
}
```

### Get docs info

Gets information about documents.

**Endpoint:** `/api/block/getDocsInfo`

**Parameters:**
```json
{
  "ids": ["20210808180320-abc123"]
}
```

- `ids` (array): Array of document IDs

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "id": "20210808180320-abc123",
      "title": "Document Title",
      "path": "/path/to/document"
    }
  ]
}
```

### Append daily note block

Appends a block to the daily note.

**Endpoint:** `/api/block/appendDailyNoteBlock`

**Parameters:**
```json
{
  "dataType": "markdown",
  "data": "Daily note content"
}
```

- `dataType` (string): Data type ("markdown" or "dom")
- `data` (string): Content to append

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "20210808180320-abc123"
  }
}
```

### Prepend daily note block

Prepends a block to the daily note.

**Endpoint:** `/api/block/prependDailyNoteBlock`

**Parameters:**
```json
{
  "dataType": "markdown", 
  "data": "Daily note content"
}
```

- `dataType` (string): Data type ("markdown" or "dom")
- `data` (string): Content to prepend

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "20210808180320-abc123"
  }
}
```

## Missing Attribute Operations

### Batch set block attributes

Sets attributes for multiple blocks in a single operation.

**Endpoint:** `/api/attr/batchSetBlockAttrs`

**Parameters:**
```json
{
  "attrs": [
    {
      "id": "20210808180320-abc123",
      "attrs": {
        "custom-attr1": "value1"
      }
    },
    {
      "id": "20210808180320-def456",
      "attrs": {
        "custom-attr2": "value2"
      }
    }
  ]
}
```

- `attrs` (array): Array of block attribute objects
  - `id` (string): Block ID
  - `attrs` (object): Attributes to set

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Batch get block attributes

Gets attributes for multiple blocks in a single operation.

**Endpoint:** `/api/attr/batchGetBlockAttrs`

**Parameters:**
```json
{
  "ids": ["20210808180320-abc123", "20210808180320-def456"]
}
```

- `ids` (array): Array of block IDs

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "id": "20210808180320-abc123",
      "attrs": {
        "custom-attr1": "value1"
      }
    }
  ]
}
```

## Missing Document Operations

### Duplicate a document

Creates a duplicate copy of an existing document.

**Endpoint:** `/api/filetree/duplicateDoc`

**Parameters:**
```json
{
  "id": "20210808180320-fqgskfj"
}
```

- `id` (string): Document ID to duplicate

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "20210808180320-abc123",
    "box": "20210817205410-2kvfpfn",
    "path": "/duplicated-document.sy",
    "hpath": "/Duplicated Document"
  }
}
```

### List document tree

Lists the document tree structure.

**Endpoint:** `/api/filetree/listDocTree`

**Parameters:**
```json
{
  "notebook": "20210817205410-2kvfpfn",
  "path": "/"
}
```

- `notebook` (string): Notebook ID
- `path` (string): Root path to list from

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "files": [
      {
        "name": "Document 1",
        "path": "/document1.sy",
        "id": "20210808180320-abc123"
      }
    ]
  }
}
```

### Upsert indexes

Creates or updates document indexes.

**Endpoint:** `/api/filetree/upsertIndexes`

**Parameters:**
```json
{
  "paths": ["/document1.sy", "/document2.sy"]
}
```

- `paths` (array): Array of document paths to index

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Remove indexes

Removes document indexes.

**Endpoint:** `/api/filetree/removeIndexes`

**Parameters:**
```json
{
  "paths": ["/document1.sy", "/document2.sy"]
}
```

- `paths` (array): Array of document paths to remove from index

**Response:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

---

This comprehensive documentation now includes all the missing SiYuan API endpoints that were identified from the changelogs and recent additions. Each endpoint includes detailed parameter descriptions, request/response examples, and usage information to help developers integrate with SiYuan effectively.

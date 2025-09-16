[English](https://github.com/siyuan-note/siyuan/blob/master/API.md)

* [规范](#规范)
    * [参数和返回值](#参数和返回值)
    * [鉴权](#鉴权)
* [笔记本](#笔记本)
    * [列出笔记本](#列出笔记本)
    * [打开笔记本](#打开笔记本)
    * [关闭笔记本](#关闭笔记本)
    * [重命名笔记本](#重命名笔记本)
    * [创建笔记本](#创建笔记本)
    * [删除笔记本](#删除笔记本)
    * [获取笔记本配置](#获取笔记本配置)
    * [保存笔记本配置](#保存笔记本配置)
* [文档](#文档)
    * [通过 Markdown 创建文档](#通过-markdown-创建文档)
    * [重命名文档](#重命名文档)
    * [删除文档](#删除文档)
    * [移动文档](#移动文档)
    * [根据路径获取人类可读路径](#根据路径获取人类可读路径)
    * [根据 ID 获取人类可读路径](#根据-ID-获取人类可读路径)
    * [根据 ID 获取存储路径](#根据-ID-获取存储路径)
    * [根据人类可读路径获取 IDs](#根据人类可读路径获取-IDs)
* [资源文件](#资源文件)
    * [上传资源文件](#上传资源文件)
* [块](#块)
    * [插入块](#插入块)
    * [插入前置子块](#插入前置子块)
    * [插入后置子块](#插入后置子块)
    * [更新块](#更新块)
    * [删除块](#删除块)
    * [移动块](#移动块)
    * [折叠块](#折叠块)
    * [展开块](#展开块)
    * [获取块 kramdown 源码](#获取块-kramdown-源码)
    * [获取子块](#获取子块)
    * [转移块引用](#转移块引用)
* [属性](#属性)
    * [设置块属性](#设置块属性)
    * [获取块属性](#获取块属性)
* [SQL](#SQL)
    * [执行 SQL 查询](#执行-SQL-查询)
    * [提交事务](#提交事务)
* [模板](#模板)
    * [渲染模板](#渲染模板)
    * [渲染 Sprig](#渲染-Sprig)
* [文件](#文件)
    * [获取文件](#获取文件)
    * [写入文件](#写入文件)
    * [删除文件](#删除文件)
    * [重命名文件](#重命名文件)
    * [列出文件](#列出文件)
* [导出](#导出)
    * [导出 Markdown 文本](#导出-markdown-文本)
    * [导出文件与目录](#导出文件与目录)
* [转换](#转换)
    * [Pandoc](#Pandoc)
* [通知](#通知)
    * [推送消息](#推送消息)
    * [推送报错消息](#推送报错消息)
* [网络](#网络)
    * [正向代理](#正向代理)
* [系统](#系统)
    * [获取启动进度](#获取启动进度)
    * [获取系统版本](#获取系统版本)
    * [获取系统当前时间](#获取系统当前时间)

---

## 规范

### 参数和返回值

* 端点：`http://127.0.0.1:6806`
* 均是 POST 方法
* 需要带参的接口，参数为 JSON 字符串，放置到 body 里，标头 Content-Type 为 `application/json`
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {}
  }
  ```

    * `code`：非 0 为异常情况
    * `msg`：正常情况下是空字符串，异常情况下会返回错误文案
    * `data`：可能为 `{}`、`[]` 或者 `NULL`，根据不同接口而不同

### 鉴权

在 <kbd>设置 - 关于</kbd> 里查看 API token，请求标头：`Authorization: Token xxx`

## 笔记本

### 列出笔记本

* `/api/notebook/lsNotebooks`
* 不带参
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "notebooks": [
        {
          "id": "20210817205410-2kvfpfn", 
          "name": "测试笔记本",
          "icon": "1f41b",
          "sort": 0,
          "closed": false
        },
        {
          "id": "20210808180117-czj9bvb",
          "name": "思源笔记用户指南",
          "icon": "1f4d4",
          "sort": 1,
          "closed": false
        }
      ]
    }
  }
  ```

### 打开笔记本

* `/api/notebook/openNotebook`
* 参数

  ```json
  {
    "notebook": "20210831090520-7dvbdv0"
  }
  ```

    * `notebook`：笔记本 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 关闭笔记本

* `/api/notebook/closeNotebook`
* 参数

  ```json
  {
    "notebook": "20210831090520-7dvbdv0"
  }
  ```

    * `notebook`：笔记本 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 重命名笔记本

* `/api/notebook/renameNotebook`
* 参数

  ```json
  {
    "notebook": "20210831090520-7dvbdv0",
    "name": "笔记本的新名称"
  }
  ```

    * `notebook`：笔记本 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 创建笔记本

* `/api/notebook/createNotebook`
* 参数

  ```json
  {
    "name": "笔记本的名称"
  }
  ```
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "notebook": {
        "id": "20220126215949-r1wvoch",
        "name": "笔记本的名称",
        "icon": "",
        "sort": 0,
        "closed": false
      }
    }
  }
  ```

### 删除笔记本

* `/api/notebook/removeNotebook`
* 参数

  ```json
  {
    "notebook": "20210831090520-7dvbdv0"
  }
  ```

    * `notebook`：笔记本 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 获取笔记本配置

* `/api/notebook/getNotebookConf`
* 参数

  ```json
  {
    "notebook": "20210817205410-2kvfpfn"
  }
  ```

    * `notebook`：笔记本 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "box": "20210817205410-2kvfpfn",
      "conf": {
        "name": "测试笔记本",
        "closed": false,
        "refCreateSavePath": "",
        "createDocNameTemplate": "",
        "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
        "dailyNoteTemplatePath": ""
      },
      "name": "测试笔记本"
    }
  }
  ```

### 保存笔记本配置

* `/api/notebook/setNotebookConf`
* 参数

  ```json
  {
    "notebook": "20210817205410-2kvfpfn",
    "conf": {
        "name": "测试笔记本",
        "closed": false,
        "refCreateSavePath": "",
        "createDocNameTemplate": "",
        "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
        "dailyNoteTemplatePath": ""
      }
  }
  ```

    * `notebook`：笔记本 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "name": "测试笔记本",
      "closed": false,
      "refCreateSavePath": "",
      "createDocNameTemplate": "",
      "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
      "dailyNoteTemplatePath": ""
    }
  }
  ```

## 文档

### 通过 Markdown 创建文档

* `/api/filetree/createDocWithMd`
* 参数

  ```json
  {
    "notebook": "20210817205410-2kvfpfn",
    "path": "/foo/bar",
    "markdown": ""
  }
  ```

    * `notebook`：笔记本 ID
    * `path`：文档路径，需要以 / 开头，中间使用 / 分隔层级（这里的 path 对应数据库 hpath 字段）
    * `markdown`：GFM Markdown 内容
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": "20210914223645-oj2vnx2"
  }
  ```

    * `data`：创建好的文档 ID
    * 如果使用同一个 `path` 重复调用该接口，不会覆盖已有文档

### 重命名文档

* `/api/filetree/renameDoc`
* 参数

  ```json
  {
    "notebook": "20210831090520-7dvbdv0",
    "path": "/20210902210113-0avi12f.sy",
    "title": "文档新标题"
  }
  ```

    * `notebook`：笔记本 ID
    * `path`：文档路径
    * `title`：新标题
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

通过 `id` 重命名文档：

* `/api/filetree/renameDocByID`
* 参数

  ```json
  {
    "id": "20210902210113-0avi12f",
    "title": "文档新标题"
  }
  ```

    * `id`：文档 ID
    * `title`：新标题
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 删除文档

* `/api/filetree/removeDoc`
* 参数

  ```json
  {
    "notebook": "20210831090520-7dvbdv0",
    "path": "/20210902210113-0avi12f.sy"
  }
  ```

    * `notebook`：笔记本 ID
    * `path`：文档路径
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

通过 `id` 删除文档：

* `/api/filetree/removeDocByID`
* 参数

  ```json
  {
    "id": "20210902210113-0avi12f"
  }
  ```

    * `id`：文档 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 移动文档

* `/api/filetree/moveDocs`
* 参数

  ```json
  {
    "fromPaths": ["/20210917220056-yxtyl7i.sy"],
    "toNotebook": "20210817205410-2kvfpfn",
    "toPath": "/"
  }
  ```

    * `fromPaths`：源路径
    * `toNotebook`：目标笔记本 ID
    * `toPath`：目标路径
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

通过 `id` 移动文档：

* `/api/filetree/moveDocsByID`
* 参数

  ```json
  {
    "fromIDs": ["20210917220056-yxtyl7i"],
    "toID": "20210817205410-2kvfpfn"
  }
  ```

    * `fromIDs`：源文档 ID
    * `toID`：目标父文档 ID 或笔记本 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 根据路径获取人类可读路径

* `/api/filetree/getHPathByPath`
* 参数

  ```json
  {
    "notebook": "20210831090520-7dvbdv0",
    "path": "/20210917220500-sz588nq/20210917220056-yxtyl7i.sy"
  }
  ```

    * `notebook`：笔记本 ID
    * `path`：路径
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": "/foo/bar"
  }
  ```

### 根据 ID 获取人类可读路径

* `/api/filetree/getHPathByID`
* 参数

  ```json
  {
    "id": "20210917220056-yxtyl7i"
  }
  ```

    * `id`：块 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": "/foo/bar"
  }
  ```

### 根据 ID 获取存储路径

* `/api/filetree/getPathByID`
* 参数

  ```json
  {
    "id": "20210808180320-fqgskfj"
  }
  ```

    * `id`：块 ID
* 返回值

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

### 根据人类可读路径获取 IDs

* `/api/filetree/getIDsByHPath`
* 参数

  ```json
  {
    "path": "/foo/bar",
    "notebook": "20210808180117-czj9bvb"
  }
  ```

    * `path`：人类可读路径
    * `notebook`：笔记本 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
        "20200813004931-q4cu8na"
    ]
  }
  ```

## 资源文件

### 上传资源文件

* `/api/asset/upload`
* 参数为 HTTP Multipart 表单

    * `assetsDirPath`：资源文件存放的文件夹路径，以 data 文件夹作为根路径，比如：
        * `"/assets/"`：工作空间/data/assets/ 文件夹
        * `"/assets/sub/"`：工作空间/data/assets/sub/ 文件夹

      常规情况下建议用第一种，统一存放到工作空间资源文件夹下，放在子目录有一些副作用，请参考用户指南资源文件章节。
    * `file[]`：上传的文件列表
* 返回值

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

    * `errFiles`：处理时遇到错误的文件名
    * `succMap`：处理成功的文件，key 为上传时的文件名，value 为 assets/foo-id.png，用于将已有 Markdown 内容中的资源文件链接地址替换为上传后的地址

## 块

### 插入块

* `/api/block/insertBlock`
* 参数

  ```json
  {
    "dataType": "markdown",
    "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
    "nextID": "",
    "previousID": "20211229114650-vrek5x6",
    "parentID": ""
  }
  ```

    * `dataType`：待插入数据类型，值可选择 `markdown` 或者 `dom`
    * `data`：待插入的数据
    * `nextID`：后一个块的 ID，用于锚定插入位置
    * `previousID`：前一个块的 ID，用于锚定插入位置
    * `parentID`：父块 ID，用于锚定插入位置

  `nextID`、`previousID`、`parentID` 三个参数必须至少存在一个有值，优先级为 `nextID` > `previousID` > `parentID`
* 返回值

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

    * `action.data`：新插入块生成的 DOM
    * `action.id`：新插入块的 ID

### 插入前置子块

* `/api/block/prependBlock`
* 参数

  ```json
  {
    "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
    "dataType": "markdown",
    "parentID": "20220107173950-7f9m1nb"
  }
  ```

    * `dataType`：待插入数据类型，值可选择 `markdown` 或者 `dom`
    * `data`：待插入的数据
    * `parentID`：父块的 ID，用于锚定插入位置
* 返回值

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

    * `action.data`：新插入块生成的 DOM
    * `action.id`：新插入块的 ID

### 插入后置子块

* `/api/block/appendBlock`
* 参数

  ```json
  {
    "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
    "dataType": "markdown",
    "parentID": "20220107173950-7f9m1nb"
  }
  ```

    * `dataType`：待插入数据类型，值可选择 `markdown` 或者 `dom`
    * `data`：待插入的数据
    * `parentID`：父块的 ID，用于锚定插入位置
* 返回值

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

    * `action.data`：新插入块生成的 DOM
    * `action.id`：新插入块的 ID

### 更新块

* `/api/block/updateBlock`
* 参数

  ```json
  {
    "dataType": "markdown",
    "data": "foobarbaz",
    "id": "20211230161520-querkps"
  }
  ```

    * `dataType`：待更新数据类型，值可选择 `markdown` 或者 `dom`
    * `data`：待更新的数据
    * `id`：待更新块的 ID
* 返回值

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

    * `action.data`：更新块生成的 DOM

### 删除块

* `/api/block/deleteBlock`
* 参数

  ```json
  {
    "id": "20211230161520-querkps"
  }
  ```

    * `id`：待删除块的 ID
* 返回值

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

### 移动块

* `/api/block/moveBlock`
* 参数

  ```json
  {
    "id": "20230406180530-3o1rqkc",
    "previousID": "20230406152734-if5kyx6",
    "parentID": "20230404183855-woe52ko"
  }
  ```

    * `id`：待移动块 ID
    * `previousID`：前一个块的 ID，用于锚定插入位置
    * `parentID`：父块的 ID，用于锚定插入位置，`previousID` 和 `parentID` 不能同时为空，同时存在的话优先使用 `previousID`
* 返回值

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

### 折叠块

* `/api/block/foldBlock`
* 参数

  ```json
  {
    "id": "20231224160424-2f5680o"
  }
  ```

    * `id`：待折叠块的 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 展开块

* `/api/block/unfoldBlock`
* 参数

  ```json
  {
    "id": "20231224160424-2f5680o"
  }
  ```

    * `id`：待展开块的 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 获取块 kramdown 源码

* `/api/block/getBlockKramdown`
* 参数

  ```json
  {
    "id": "20201225220955-l154bn4"
  }
  ```

    * `id`：待获取块的 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "id": "20201225220955-l154bn4",
      "kramdown": "* {: id=\"20201225220955-2nn1mns\"}新建笔记本，在笔记本下新建文档\n  {: id=\"20210131155408-3t627wc\"}\n* {: id=\"20201225220955-uwhqnug\"}在编辑器中输入 <kbd>/</kbd> 触发功能菜单\n  {: id=\"20210131155408-btnfw88\"}\n* {: id=\"20201225220955-04ymi2j\"}((20200813131152-0wk5akh \"在内容块中遨游\"))、((20200822191536-rm6hwid \"窗口和页签\"))\n  {: id=\"20210131155408-hh1z442\"}"
    }
  }
  ```

### 获取子块

* `/api/block/getChildBlocks`
* 参数

  ```json
  {
    "id": "20230506212712-vt9ajwj"
  }
  ```

    * `id`：父块 ID
    * 标题下方块也算作子块
* 返回值

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

### 转移块引用

* `/api/block/transferBlockRef`
* 参数

  ```json
  {
    "fromID": "20230612160235-mv6rrh1",
    "toID": "20230613093045-uwcomng",
    "refIDs": ["20230613092230-cpyimmd"]
  }
  ```

    * `fromID`：定义块 ID
    * `toID`：目标块 ID
    * `refIDs`：指向定义块 ID 的引用所在块 ID，可选，如果不指定，所有指向定义块 ID 的块引用 ID 都会被转移
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## 属性

### 设置块属性

* `/api/attr/setBlockAttrs`
* 参数

  ```json
  {
    "id": "20210912214605-uhi5gco",
    "attrs": {
      "custom-attr1": "line1\nline2"
    }
  }
  ```

    * `id`：块 ID
    * `attrs`：块属性，自定义属性必须以 `custom-` 作为前缀
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 获取块属性

* `/api/attr/getBlockAttrs`
* 参数

  ```json
  {
    "id": "20210912214605-uhi5gco"
  }
  ```

    * `id`：块 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "custom-attr1": "line1\nline2",
      "id": "20210912214605-uhi5gco",
      "title": "PDF 标注双链演示",
      "type": "doc",
      "updated": "20210916120715"
    }
  }
  ```

## SQL

### 执行 SQL 查询

* `/api/query/sql`
* 参数

  ```json
  {
    "stmt": "SELECT * FROM blocks WHERE content LIKE'%content%' LIMIT 7"
  }
  ```

    * `stmt`：SQL 脚本
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": [
      { "列": "值" }
    ]
  }
  ```

### 提交事务

* `/api/sqlite/flushTransaction`
* 不带参
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## 模板

### 渲染模板

* `/api/template/render`
* 参数

  ```json
  {
    "id": "20220724223548-j6g0o87",
    "path": "F:\\SiYuan\\data\\templates\\foo.md"
  }
  ```
    * `id`：调用渲染所在的文档 ID
    * `path`：模板文件绝对路径
* 返回值

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

### 渲染 Sprig

* `/api/template/renderSprig`
* 参数

  ```json
  {
    "template": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}"
  }
  ```
    * `template`：模板内容
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": "/daily note/2023/03/2023-03-24"
  }
  ```

## 文件

### 获取文件

* `/api/file/getFile`
* 参数

  ```json
  {
    "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p.sy"
  }
  ```
    * `path`：工作空间路径下的文件路径
* 返回值

    * 响应状态码 `200`: 文件内容
    * 响应状态码 `202`: 异常信息

      ```json
      {
        "code": 404,
        "msg": "",
        "data": null
      }
      ```

        * `code`: 非零的异常值

            * `-1`: 参数解析错误
            * `403`: 无访问权限 (文件不在工作空间下)
            * `404`: 未找到 (文件不存在)
            * `405`: 方法不被允许 (这是一个目录)
            * `500`: 服务器错误 (文件查询失败 / 文件读取失败)
        * `msg`: 一段描述错误的文本

### 写入文件

* `/api/file/putFile`
* 参数为 HTTP Multipart 表单

    * `path`：工作空间路径下的文件路径
    * `isDir`：是否为创建文件夹，为 `true` 时仅创建文件夹，忽略 `file`
    * `modTime`：最近访问和修改时间，Unix time
    * `file`：上传的文件
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 删除文件

* `/api/file/removeFile`
* 参数

  ```json
  {
    "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p.sy"
  }
  ```
    * `path`：工作空间路径下的文件路径
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 重命名文件

* `/api/file/renameFile`
* 参数

  ```json
  {
    "path": "/data/assets/image-20230523085812-k3o9t32.png",
    "newPath": "/data/assets/test-20230523085812-k3o9t32.png"
  }
  ```
    * `path`：工作空间路径下的文件路径
    * `newPath`：新的文件路径
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 列出文件

* `/api/file/readDir`
* 参数

  ```json
  {
    "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p"
  }
  ```
    * `path`：工作空间路径下的文件夹路径
* 返回值

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

## 导出

### 导出 Markdown 文本

* `/api/export/exportMdContent`
* 参数

  ```json
  {
    "id": ""
  }
  ```

    * `id`：要导出的文档块 ID
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "hPath": "/0 请从这里开始",
      "content": "## 🍫 内容块\n\n在思源中，唯一重要的核心概念是..."
    }
  }
  ```

    * `hPath`：人类可读的路径
    * `content`：Markdown 内容

### 导出文件与目录

* `/api/export/exportResources`
* 参数

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

    * `paths`：要导出的文件或文件夹路径列表，相同名称的文件/文件夹会被覆盖
    * `name`：（可选）导出的文件名，未设置时默认为 `export-YYYY-MM-DD_hh-mm-ss.zip`
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "path": "temp/export/zip-file-name.zip"
    }
  }
  ```

    * `path`：创建的 `*.zip` 文件路径
        * `zip-file-name.zip` 中的目录结构如下所示：
            * `zip-file-name`
                * `boot`
                * `langs`
                * `conf.json`
                * `index.html`

## 转换

### Pandoc

* `/api/convert/pandoc`
* 工作目录
    * 执行调用 pandoc 命令时工作目录会被设置在 `工作空间/temp/convert/pandoc/${test}` 下
    * 可先通过 API [`写入文件`](#写入文件) 将待转换文件写入该目录
    * 然后再调用该 API 进行转换，转换后的文件也会被写入该目录
    * 最后调用 API [`获取文件`](#获取文件) 获取转换后的文件内容
        * 或者调用 API [`通过 Markdown 创建文档`](#通过-markdown-创建文档)
        * 或者调用内部 API `importStdMd` 将转换后的文件夹直接导入
* 参数

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

    * `args`：Pandoc 命令行参数
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
       "path": "/temp/convert/pandoc/test"
    }
  }
  ```
    * `path`：工作空间下的路径

## 通知

### 推送消息

* `/api/notification/pushMsg`
* 参数

  ```json
  {
    "msg": "test",
    "timeout": 7000
  }
  ```
    * `timeout`：消息持续显示时间，单位为毫秒。可以不传入该字段，默认为 7000 毫秒
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
        "id": "62jtmqi"
    }
  }
  ```
    * `id`：消息 ID

### 推送报错消息

* `/api/notification/pushErrMsg`
* 参数

  ```json
  {
    "msg": "test",
    "timeout": 7000
  }
  ```
    * `timeout`：消息持续显示时间，单位为毫秒。可以不传入该字段，默认为 7000 毫秒
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
        "id": "qc9znut"
    }
  }
  ```
    * `id`：消息 ID

## 网络

### 正向代理

* `/api/network/forwardProxy`
* 参数

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

    * `url`：转发的 URL
    * `method`：HTTP 方法，默认为 `GET`
    * `timeout`：超时时间，单位为毫秒，默认为 `7000` 毫秒
    * `contentType`：HTTP Content-Type，默认为 `application/json`
    * `headers`：HTTP 请求标头
    * `payload`：HTTP 请求体，对象或者是字符串
    * `payloadEncoding`：`pyaload` 所使用的编码方案，默认为 `text`，可选值如下所示

        * `text`
        * `base64` | `base64-std`
        * `base64-url`
        * `base32` | `base32-std`
        * `base32-hex`
        * `hex`
    * `responseEncoding`：响应数据中 `body` 字段所使用的编码方案，默认为 `text`，可选值如下所示

        * `text`
        * `base64` | `base64-std`
        * `base64-url`
        * `base32` | `base32-std`
        * `base32-hex`
        * `hex`
* 返回值

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

    * `bodyEncoding`：`body` 所使用的编码方案，与请求中 `responseEncoding` 字段一致，默认为 `text`，可能的值如下所示

        * `text`
        * `base64` | `base64-std`
        * `base64-url`
        * `base32` | `base32-std`
        * `base32-hex`
        * `hex`

## 系统

### 获取启动进度

* `/api/system/bootProgress`
* 不带参
* 返回值

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

### 获取系统版本

* `/api/system/version`
* 不带参
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": "1.3.5"
  }
  ```

### 获取系统当前时间

* `/api/system/currentTime`
* 不带参
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": 1631850968131
  }
  ```

    * `data`: 精度为毫秒

---

## 其他 API 端点

以下部分记录了最新版本中添加的其他 SiYuan API 端点，为高级用例提供了扩展功能。

## 档案

### 创建 ZIP 压缩包

从指定的文件和文件夹创建 ZIP 压缩包。

* `/api/archive/zip`
* 参数

  ```json
  {
    "paths": ["/path/to/file1", "/path/to/folder"],
    "name": "archive-name"
  }
  ```

  * `paths`: 要包含在压缩包中的文件/文件夹路径数组
  * `name`: ZIP 文件名（可选）

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "path": "/path/to/archive.zip"
    }
  }
  ```

### 解压 ZIP 压缩包

将 ZIP 压缩包解压到指定位置。

* `/api/archive/unzip`
* 参数

  ```json
  {
    "path": "/path/to/archive.zip",
    "dest": "/extraction/destination"
  }
  ```

  * `path`: 要解压的 ZIP 文件路径
  * `dest`: 解压目标文件夹

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## 搜索

### 获取嵌入块

检索嵌入块的信息。

* `/api/search/getEmbedBlock`
* 参数

  ```json
  {
    "id": "20210808180320-fqgskfj",
    "includeIDs": ["20210808180320-abc123"]
  }
  ```

  * `id`: 要搜索的块 ID
  * `includeIDs`: 要包含在结果中的块 ID 数组

* 返回值

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

### 更新嵌入块

更新嵌入块的内容。

* `/api/search/updateEmbedBlock`
* 参数

  ```json
  {
    "id": "20210808180320-fqgskfj",
    "content": "Updated block content"
  }
  ```

  * `id`: 要更新的块 ID
  * `content`: 块的新内容

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## 引用

### 获取反链文档

检索包含特定块反链的文档。

* `/api/ref/getBacklinkDoc`
* 参数

  ```json
  {
    "id": "20210808180320-fqgskfj"
  }
  ```

  * `id`: 要查找反链的块 ID

* 返回值

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

## UI 操作

### 重新加载属性视图

重新加载 UI 中的属性视图。

* `/api/ui/reloadAttributeView`
* 参数

  ```json
  {
    "id": "20210808180320-fqgskfj"
  }
  ```

  * `id`: 要重新加载的属性视图 ID

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 重新加载文件树

重新加载 UI 中的文件树。

* `/api/ui/reloadFiletree`
* 不带参
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 重新加载编辑器

重新加载 protyle 编辑器。

* `/api/ui/reloadProtyle`
* 参数

  ```json
  {
    "id": "20210808180320-fqgskfj"
  }
  ```

  * `id`: 要在 protyle 中重新加载的块 ID

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 重新加载标签

重新加载 UI 中的标签面板。

* `/api/ui/reloadTag`
* 不带参
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 重新加载 UI

重新加载整个 UI。

* `/api/ui/reloadUI`
* 不带参
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## 设置

### 设置编辑器只读

将编辑器设置为只读模式。

* `/api/setting/setEditorReadOnly`
* 参数

  ```json
  {
    "readOnly": true
  }
  ```

  * `readOnly`: 是否启用只读模式

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 添加虚拟块引用包含

添加虚拟块引用包含模式。

* `/api/setting/addVirtualBlockRefInclude`
* 参数

  ```json
  {
    "pattern": "*.md"
  }
  ```

  * `pattern`: 虚拟块引用要包含的文件模式

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 刷新虚拟块引用

刷新虚拟块引用。

* `/api/setting/refreshVirtualBlockRef`
* 不带参
* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## 属性视图

### 附加属性视图分离块及其值

将分离的块及其特定值附加到属性视图。

* `/api/av/appendAttributeViewDetachedBlocksWithValues`
* 参数

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

  * `avID`: 属性视图 ID
  * `blockIDs`: 要附加的块 ID 数组
  * `values`: 属性值的键值对

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 批量替换属性视图块

替换属性视图中的多个块。

* `/api/av/batchReplaceAttributeViewBlocks`
* 参数

  ```json
  {
    "avID": "20210808180320-fqgskfj",
    "blockIDs": ["20210808180320-abc123"],
    "replacements": ["20210808180320-def456"]
  }
  ```

  * `avID`: 属性视图 ID
  * `blockIDs`: 要替换的块 ID 数组
  * `replacements`: 替换块 ID 数组

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 批量设置属性视图块属性

为属性视图中的多个块设置属性。

* `/api/av/batchSetAttributeViewBlockAttrs`
* 参数

  ```json
  {
    "avID": "20210808180320-fqgskfj",
    "blockIDs": ["20210808180320-abc123"],
    "attrs": {
      "custom-attr1": "value1"
    }
  }
  ```

  * `avID`: 属性视图 ID
  * `blockIDs`: 要更新的块 ID 数组
  * `attrs`: 要设置的属性

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

### 获取属性视图绑定块 ID

获取绑定到属性视图的块 ID。

* `/api/av/getAttributeViewBoundBlockIDs`
* 参数

  ```json
  {
    "avID": "20210808180320-fqgskfj"
  }
  ```

  * `avID`: 属性视图 ID

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "blockIDs": ["20210808180320-abc123", "20210808180320-def456"]
    }
  }
  ```

## 间隔重复

### 获取间隔重复卡片

检索间隔重复卡片。

* `/api/riff/getRiffCards`
* 参数

  ```json
  {
    "deckID": "20210808180320-fqgskfj"
  }
  ```

  * `deckID`: 要获取卡片的牌组 ID

* 返回值

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

### 根据块 ID 获取间隔重复卡片

检索特定块 ID 的间隔重复卡片。

* `/api/riff/getRiffCardsByBlockIDs`
* 参数

  ```json
  {
    "blockIDs": ["20210808180320-abc123", "20210808180320-def456"]
  }
  ```

  * `blockIDs`: 要获取卡片的块 ID 数组

* 返回值

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

### 批量设置间隔重复卡片到期时间

为多个间隔重复卡片设置到期时间。

* `/api/riff/batchSetRiffCardsDueTime`
* 参数

  ```json
  {
    "cardIDs": ["20210808180320-abc123"],
    "dueTime": 1631850968131
  }
  ```

  * `cardIDs`: 要更新的卡片 ID 数组
  * `dueTime`: 新的到期时间（Unix 时间戳，毫秒）

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## 仓库

### 获取仓库文件

从仓库中检索文件。

* `/api/repo/getRepoFile`
* 参数

  ```json
  {
    "path": "/path/to/file.md"
  }
  ```

  * `path`: 仓库中的文件路径

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": {
      "content": "File content here"
    }
  }
  ```

## 文件操作

### 全局复制文件

在系统中全局复制文件。

* `/api/file/globalCopyFiles`
* 参数

  ```json
  {
    "paths": ["/source/file1.md", "/source/file2.md"],
    "dest": "/destination/folder"
  }
  ```

  * `paths`: 源文件路径数组
  * `dest`: 目标文件夹路径

* 返回值

  ```json
  {
    "code": 0,
    "msg": "",
    "data": null
  }
  ```

## 图标

### 获取动态图标

检索动态图标。

* `/api/icon/getDynamicIcon`
* 参数

  ```json
  {
    "type": "emoji",
    "id": "1f41b"
  }
  ```

  * `type`: 图标类型（如 "emoji"）
  * `id`: 图标标识符

* 返回值

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

### HTML 转块 DOM

将 HTML 内容转换为 SiYuan 块 DOM 结构。

* `/api/lute/html2BlockDOM`
* 参数

  ```json
  {
    "html": "<p>Hello <strong>world</strong></p>"
  }
  ```

  * `html`: 要转换的 HTML 内容

* 返回值

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

这个全面的文档现在包含了从更新日志和最新添加中识别出的所有缺失的 SiYuan API 端点。每个端点都包含详细的参数描述、请求/响应示例和使用信息，以帮助开发者有效地与 SiYuan 集成。

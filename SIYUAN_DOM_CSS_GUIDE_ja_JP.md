# SiYuan DOM構造・CSS解説ガイド

プラグイン・テーマ制作者向けの完全リファレンス

**対象読者**: SiYuanプラグインやテーマを開発する開発者

**前提知識**: 
- HTML/CSS の基本的な知識
- JavaScript/TypeScript の基本的な知識
- Web開発の基本概念

---

## 📋 目次

### 🏗️ アーキテクチャ概要
- [SiYuanのDOM構造](#siyuanのdom構造)
- [CSSアーキテクチャ](#cssアーキテクチャ)
- [テーマシステム](#テーマシステム)

### 🎨 UI コンポーネント
- [トップバー（Toolbar）](#トップバーtoolbar)
- [サイドパネル](#サイドパネル)
- [エディター（Protyle）](#エディターprotyle)
- [ダイアログ・モーダル](#ダイアログモーダル)
- [メニューシステム](#メニューシステム)

### 🎯 レイアウトシステム
- [レイアウトマネージャー](#レイアウトマネージャー)
- [フレックスボックスシステム](#フレックスボックスシステム)
- [ドック・パネル](#ドックパネル)

### 🎪 カスタマイズ
- [テーマ開発](#テーマ開発)
- [CSS変数システム](#css変数システム)
- [プラグインUI開発](#プラグインui開発)
- [レスポンシブデザイン](#レスポンシブデザイン)

---

## 🏗️ SiYuanのDOM構造

### 基本的なHTML構造

```html
<!DOCTYPE html>
<html>
<head>
    <!-- SiYuan メタデータ -->
</head>
<body class="body--win32" data-theme-mode="light">
    <!-- ルートコンテナ -->
    <div id="app">
        <!-- トップバー -->
        <div id="toolbar" class="toolbar">
            <!-- ツールバー要素 -->
        </div>
        
        <!-- メインコンテンツエリア -->
        <div class="fn__flex-1 fn__flex" id="main">
            <!-- レイアウトマネージャー -->
            <div class="layout fn__flex">
                <!-- サイドパネル -->
                <div class="layout__side" data-type="left">
                    <!-- ドック -->
                </div>
                
                <!-- センターエリア -->
                <div class="layout__center fn__flex-1">
                    <!-- タブコンテナ -->
                    <div class="layout__tab-container">
                        <!-- エディター -->
                    </div>
                </div>
                
                <!-- 右サイドパネル -->
                <div class="layout__side" data-type="right">
                    <!-- 右ドック -->
                </div>
            </div>
        </div>
        
        <!-- ステータスバー -->
        <div id="status" class="status">
            <!-- ステータス要素 -->
        </div>
    </div>
    
    <!-- オーバーレイ要素 -->
    <div class="side-mask"></div>
    <div class="protyle-util"></div>
</body>
</html>
```

### DOM階層の詳細解説

#### 1. ルートレベル要素

```html
<body class="body--win32" data-theme-mode="light">
```

**重要なクラス・属性:**
- `body--win32` / `body--darwin` / `body--linux`: OS固有スタイル
- `data-theme-mode`: `light` / `dark` - テーマモード
- `data-lang`: 言語設定

#### 2. アプリケーションコンテナ

```html
<div id="app">
```

**役割**: SiYuanアプリケーション全体のルートコンテナ

---

## 🎨 CSSアーキテクチャ

### CSS変数システム

SiYuanは包括的なCSS変数システムを使用しています。

#### 主要なCSS変数グループ

```css
:root {
    /* === 主色（Primary Colors） === */
    --b3-theme-primary: #3575f0;           /* メイン色 */
    --b3-theme-primary-light: rgba(53, 117, 240, .54);
    --b3-theme-primary-lighter: rgba(53, 117, 240, .38);
    --b3-theme-primary-lightest: rgba(53, 117, 240, .12);
    --b3-theme-secondary: #ff9200;         /* セカンダリ色 */
    
    /* === 背景色（Background Colors） === */
    --b3-theme-background: #fff;           /* メイン背景 */
    --b3-theme-background-light: #dfe0e1;  /* ライト背景 */
    --b3-theme-surface: #f6f6f6;          /* サーフェス */
    --b3-theme-surface-light: rgba(243, 243, 243, .86);
    --b3-theme-surface-lighter: #e0e0e0;
    
    /* === 状態色（State Colors） === */
    --b3-theme-error: #d23f31;            /* エラー色 */
    --b3-theme-success: #65b84d;          /* 成功色 */
    
    /* === テキスト色（Text Colors） === */
    --b3-theme-on-primary: #fff;          /* プライマリ上のテキスト */
    --b3-theme-on-secondary: #fff;        /* セカンダリ上のテキスト */
    --b3-theme-on-background: #222;       /* 背景上のテキスト */
    --b3-theme-on-surface: #5f6368;      /* サーフェス上のテキスト */
    --b3-theme-on-surface-light: rgba(95, 99, 104, .68);
    
    /* === フォント（Typography） === */
    --b3-font-family: "Emojis Additional", "Emojis Reset", 
                       BlinkMacSystemFont, Helvetica, "Luxi Sans", 
                       "DejaVu Sans", arial, sans-serif, emojis;
    --b3-font-family-protyle: var(--b3-font-family);
    --b3-font-family-code: "JetBrainsMono-Regular", mononoki, 
                            Consolas, "Liberation Mono", var(--b3-font-family);
    --b3-font-size: 14px;
    
    /* === レイアウト（Layout） === */
    --b3-border-radius: 6px;              /* 基本の角丸 */
    --b3-border-radius-s: 3px;            /* 小さい角丸 */
    --b3-border-radius-b: 12px;           /* 大きい角丸 */
    --b3-border-color: var(--b3-theme-surface-lighter);
    
    /* === ツールバー（Toolbar） === */
    --b3-toolbar-background: var(--b3-theme-surface);
    --b3-toolbar-color: var(--b3-theme-on-surface);
    --b3-toolbar-hover: var(--b3-theme-background-light);
    --b3-toolbar-left-mac: 69px;          /* macOSの左マージン */
}
```

### ダークテーマの変数

```css
[data-theme-mode="dark"] {
    --b3-theme-primary: #4285f4;
    --b3-theme-background: #1e1e1e;
    --b3-theme-surface: #2d2d30;
    --b3-theme-on-background: #d4d4d4;
    --b3-theme-on-surface: #cccccc;
    /* ... その他のダークテーマ変数 */
}
```

---

## 🎪 トップバー（Toolbar）

### DOM構造

```html
<div id="toolbar" class="toolbar">
    <!-- ワークスペース選択 -->
    <div id="barWorkspace" class="toolbar__item toolbar__item--active">
        <span class="toolbar__text">ワークスペース名</span>
        <svg class="toolbar__svg">
            <use xlink:href="#iconDown"></use>
        </svg>
    </div>
    
    <!-- 同期ボタン -->
    <div id="barSync" class="toolbar__item">
        <svg><use xlink:href="#iconCloudSucc"></use></svg>
    </div>
    
    <!-- ナビゲーションボタン -->
    <button id="barBack" class="toolbar__item toolbar__item--disabled">
        <svg><use xlink:href="#iconBack"></use></svg>
    </button>
    <button id="barForward" class="toolbar__item toolbar__item--disabled">
        <svg><use xlink:href="#iconForward"></use></svg>
    </button>
    
    <!-- ドラッグエリア -->
    <div class="fn__flex-1 fn__ellipsis" id="drag">
        <span class="fn__none">タイトルテキスト</span>
    </div>
    
    <!-- 右側ボタン群 -->
    <div id="barPlugins" class="toolbar__item">
        <svg><use xlink:href="#iconPlugin"></use></svg>
    </div>
    <div id="barCommand" class="toolbar__item">
        <svg><use xlink:href="#iconTerminal"></use></svg>
    </div>
    <div id="barSearch" class="toolbar__item">
        <svg><use xlink:href="#iconSearch"></use></svg>
    </div>
</div>
```

### 重要なCSSクラス

```css
.toolbar {
    background-color: var(--b3-toolbar-background);
    height: 32px;
    line-height: 32px;
    padding: 0 5px 0 var(--b3-toolbar-left-mac);
    border-bottom: .5px solid var(--b3-border-color);
}

.toolbar__item {
    flex-shrink: 0;
    cursor: pointer;
    color: var(--b3-toolbar-color);
    padding: 5px;
    margin: 2px;
    border-radius: var(--b3-border-radius);
    
    &:hover {
        background-color: var(--b3-toolbar-hover);
    }
    
    &--active {
        background-color: var(--b3-theme-primary-lightest);
        color: var(--b3-theme-primary);
    }
    
    &--disabled {
        opacity: 0.3;
        cursor: not-allowed;
    }
}
```

### プラグインでのツールバーカスタマイズ

```typescript
// ツールバーボタンの追加
this.addTopBar({
    icon: "iconCustom",
    title: "カスタムボタン",
    position: "right",
    callback: () => {
        // ボタンクリック時の処理
    }
});
```

---

## 🎯 レイアウトマネージャー

### レイアウトクラスの構造

```typescript
class Layout {
    public element: HTMLElement;
    public children?: Array<Layout | Wnd>;
    public parent?: Layout;
    public direction: "tb" | "lr";  // 縦(top-bottom) | 横(left-right)
    public type?: "normal" | "center";
}
```

### DOM表現

```html
<div class="layout fn__flex-column">  <!-- 縦方向レイアウト -->
    <div class="layout fn__flex">      <!-- 横方向レイアウト -->
        <!-- 子要素 -->
    </div>
</div>
```

### レイアウト関連CSSクラス

```css
.layout {
    &__center {
        /* センターレイアウトの特別なスタイル */
    }
    
    &__side {
        /* サイドパネルのスタイル */
        flex-shrink: 0;
        
        &[data-type="left"] {
            /* 左サイドパネル */
        }
        
        &[data-type="right"] {
            /* 右サイドパネル */
        }
    }
    
    &__tab-container {
        /* タブコンテナ */
        position: relative;
        flex: 1;
    }
}

/* フレックスボックスユーティリティ */
.fn__flex {
    display: flex;
}

.fn__flex-column {
    display: flex;
    flex-direction: column;
}

.fn__flex-1 {
    flex: 1;
}
```

---

## 📝 エディター（Protyle）

### DOM構造

```html
<div class="protyle" data-doc-type="NodeDocument">
    <!-- ブレッドクラム -->
    <div class="protyle-breadcrumb">
        <div class="protyle-breadcrumb__bar">
            <!-- パス表示 -->
        </div>
    </div>
    
    <!-- エディターコンテンツ -->
    <div class="protyle-content">
        <div class="protyle-background">
            <!-- 背景要素 -->
        </div>
        
        <!-- 編集可能エリア -->
        <div class="protyle-wysiwyg" contenteditable="true">
            <!-- ブロック要素 -->
            <div data-node-id="20231201-123456" data-type="NodeParagraph">
                <div class="p">段落テキスト</div>
            </div>
            
            <div data-node-id="20231201-123457" data-type="NodeHeading">
                <div class="h1">見出し1</div>
            </div>
            
            <div data-node-id="20231201-123458" data-type="NodeList">
                <div class="list">
                    <div class="li">リストアイテム</div>
                </div>
            </div>
        </div>
    </div>
    
    <!-- ツールバー -->
    <div class="protyle-toolbar">
        <!-- ツールバーボタン -->
    </div>
</div>
```

### ブロック要素の種類とクラス

```css
/* 段落 */
.p {
    margin: 14px 0;
    line-height: 1.625;
}

/* 見出し */
.h1, .h2, .h3, .h4, .h5, .h6 {
    font-weight: bold;
    margin: 21px 0 14px 0;
}

.h1 { font-size: 1.75em; }
.h2 { font-size: 1.55em; }
.h3 { font-size: 1.38em; }

/* リスト */
.list {
    margin: 14px 0;
    
    .li {
        position: relative;
        padding-left: 32px;
        margin: 4px 0;
        
        &::before {
            content: "•";
            position: absolute;
            left: 16px;
        }
    }
}

/* コードブロック */
.code-block {
    background-color: var(--b3-theme-surface);
    border-radius: var(--b3-border-radius);
    padding: 16px;
    margin: 14px 0;
    font-family: var(--b3-font-family-code);
}

/* 引用 */
.bq {
    border-left: 4px solid var(--b3-theme-primary);
    padding-left: 16px;
    margin: 14px 0;
    color: var(--b3-theme-on-surface-light);
}
```

---

## 🗂️ サイドパネル・ドックシステム

### ドックの基本構造

```html
<div class="dock">
    <!-- ドックヘッダー -->
    <div class="dock__header">
        <span class="dock__title">ファイルツリー</span>
        <div class="dock__actions">
            <!-- アクションボタン -->
        </div>
    </div>
    
    <!-- ドックコンテンツ -->
    <div class="dock__content">
        <!-- 具体的なコンテンツ -->
    </div>
</div>
```

### ファイルツリーの構造

```html
<div class="file-tree">
    <ul class="b3-list b3-list--background">
        <li class="b3-list-item" data-type="navigation-file">
            <span class="b3-list-item__toggle">
                <svg><use xlink:href="#iconRight"></use></svg>
            </span>
            <span class="b3-list-item__icon">
                <svg><use xlink:href="#iconFile"></use></svg>
            </span>
            <span class="b3-list-item__text">ファイル名.md</span>
        </li>
    </ul>
</div>
```

---

## 🎭 メニューシステム

### コンテキストメニューの構造

```html
<div class="b3-menu" style="left: 100px; top: 200px;">
    <div class="b3-menu__item">
        <svg class="b3-menu__icon"><use xlink:href="#iconEdit"></use></svg>
        <span class="b3-menu__label">編集</span>
        <span class="b3-menu__accelerator">Ctrl+E</span>
    </div>
    
    <div class="b3-menu__separator"></div>
    
    <div class="b3-menu__item b3-menu__item--danger">
        <svg class="b3-menu__icon"><use xlink:href="#iconTrashcan"></use></svg>
        <span class="b3-menu__label">削除</span>
    </div>
</div>
```

### メニューのCSS

```css
.b3-menu {
    position: fixed;
    border-radius: var(--b3-border-radius-b);
    box-shadow: var(--b3-dialog-shadow);
    background-color: var(--b3-menu-background);
    padding: 8px 0;
    min-width: 200px;
    
    &__item {
        display: flex;
        align-items: center;
        padding: 8px 16px;
        cursor: pointer;
        
        &:hover {
            background-color: var(--b3-list-hover);
        }
        
        &--danger {
            color: var(--b3-theme-error);
        }
        
        &--disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }
    }
    
    &__icon {
        width: 16px;
        height: 16px;
        margin-right: 8px;
        flex-shrink: 0;
    }
    
    &__label {
        flex: 1;
    }
    
    &__accelerator {
        font-size: 12px;
        color: var(--b3-theme-on-surface-light);
        margin-left: 16px;
    }
    
    &__separator {
        height: 1px;
        background-color: var(--b3-border-color);
        margin: 4px 16px;
    }
}
```

---

## 🎨 ダイアログ・モーダル

### ダイアログの基本構造

```html
<div class="b3-dialog">
    <!-- オーバーレイ -->
    <div class="b3-dialog__scrim"></div>
    
    <!-- ダイアログ本体 -->
    <div class="b3-dialog__container">
        <div class="b3-dialog__header">
            <div class="b3-dialog__title">ダイアログタイトル</div>
            <button class="b3-dialog__close">
                <svg><use xlink:href="#iconClose"></use></svg>
            </button>
        </div>
        
        <div class="b3-dialog__body">
            <!-- ダイアログコンテンツ -->
        </div>
        
        <div class="b3-dialog__action">
            <button class="b3-button b3-button--cancel">キャンセル</button>
            <button class="b3-button b3-button--text">OK</button>
        </div>
    </div>
</div>
```

### ダイアログのCSS

```css
.b3-dialog {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 200;
    display: flex;
    align-items: center;
    justify-content: center;
    
    &__scrim {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: var(--b3-mask-background);
    }
    
    &__container {
        position: relative;
        background-color: var(--b3-theme-surface);
        border-radius: var(--b3-border-radius-b);
        box-shadow: var(--b3-dialog-shadow);
        max-width: 90vw;
        max-height: 90vh;
        display: flex;
        flex-direction: column;
    }
    
    &__header {
        display: flex;
        align-items: center;
        padding: 16px 24px;
        border-bottom: 1px solid var(--b3-border-color);
    }
    
    &__title {
        flex: 1;
        font-weight: 500;
        font-size: 16px;
    }
    
    &__body {
        flex: 1;
        overflow: auto;
        padding: 24px;
    }
    
    &__action {
        display: flex;
        gap: 8px;
        justify-content: flex-end;
        padding: 16px 24px;
        border-top: 1px solid var(--b3-border-color);
    }
}
```

---

## 🧩 フォームコンポーネント

### ボタンシステム

```css
.b3-button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 8px 16px;
    border-radius: var(--b3-border-radius);
    font-size: var(--b3-font-size);
    cursor: pointer;
    border: 1px solid transparent;
    transition: all 0.15s ease;
    
    /* プライマリボタン */
    &--text {
        background-color: var(--b3-theme-primary);
        color: var(--b3-theme-on-primary);
        
        &:hover {
            background-color: var(--b3-theme-primary-light);
        }
    }
    
    /* キャンセルボタン */
    &--cancel {
        background-color: transparent;
        color: var(--b3-theme-on-surface);
        border-color: var(--b3-border-color);
        
        &:hover {
            background-color: var(--b3-theme-surface-light);
        }
    }
    
    /* 危険なアクション */
    &--danger {
        background-color: var(--b3-theme-error);
        color: var(--b3-theme-on-error);
    }
    
    /* 無効状態 */
    &:disabled {
        opacity: 0.5;
        cursor: not-allowed;
    }
}
```

### テキストフィールド

```css
.b3-text-field {
    display: block;
    width: 100%;
    padding: 8px 12px;
    border: 1px solid var(--b3-border-color);
    border-radius: var(--b3-border-radius);
    background-color: var(--b3-theme-background);
    color: var(--b3-theme-on-background);
    font-size: var(--b3-font-size);
    transition: border-color 0.15s ease;
    
    &:focus {
        outline: none;
        border-color: var(--b3-theme-primary);
        box-shadow: 0 0 0 2px var(--b3-theme-primary-lightest);
    }
    
    &::placeholder {
        color: var(--b3-theme-on-surface-light);
    }
    
    &:disabled {
        opacity: 0.5;
        cursor: not-allowed;
        background-color: var(--b3-theme-surface-light);
    }
}
```

### セレクトボックス

```css
.b3-select {
    position: relative;
    display: inline-block;
    
    select {
        appearance: none;
        padding: 8px 32px 8px 12px;
        border: 1px solid var(--b3-border-color);
        border-radius: var(--b3-border-radius);
        background-color: var(--b3-theme-background);
        cursor: pointer;
        
        &:focus {
            outline: none;
            border-color: var(--b3-theme-primary);
        }
    }
    
    &::after {
        content: "";
        position: absolute;
        right: 12px;
        top: 50%;
        transform: translateY(-50%);
        width: 0;
        height: 0;
        border-left: 4px solid transparent;
        border-right: 4px solid transparent;
        border-top: 4px solid var(--b3-theme-on-surface);
        pointer-events: none;
    }
}
```

---

## 🎪 テーマ開発

### テーマファイルの構造

```
themes/
└── my-theme/
    ├── theme.json      # テーマメタデータ
    ├── theme.css       # メインスタイル
    ├── README.md       # ドキュメント
    └── preview.png     # プレビュー画像
```

### theme.json の例

```json
{
    "name": "my-awesome-theme",
    "author": "Your Name",
    "url": "https://github.com/username/my-theme",
    "version": "1.0.0",
    "minAppVersion": "2.10.0",
    "displayName": {
        "en_US": "My Awesome Theme",
        "ja_JP": "私の素晴らしいテーマ",
        "zh_CN": "我的超棒主题"
    },
    "description": {
        "en_US": "A beautiful theme for SiYuan",
        "ja_JP": "SiYuan用の美しいテーマ",
        "zh_CN": "SiYuan的美丽主题"
    },
    "readme": {
        "en_US": "README.md",
        "ja_JP": "README_ja_JP.md",
        "zh_CN": "README_zh_CN.md"
    },
    "modes": ["light", "dark"]
}
```

### 基本的なテーマCSS

```css
/* テーマの基本設定 */
:root {
    /* カスタムカラーパレット */
    --custom-primary: #6366f1;
    --custom-secondary: #f59e0b;
    --custom-accent: #10b981;
    
    /* SiYuan変数の上書き */
    --b3-theme-primary: var(--custom-primary);
    --b3-theme-secondary: var(--custom-secondary);
    
    /* カスタム背景 */
    --b3-theme-background: #fafbfc;
    --b3-theme-surface: #f1f3f4;
    
    /* カスタムフォント */
    --b3-font-family-protyle: "Noto Sans", var(--b3-font-family);
}

/* ダークモード */
[data-theme-mode="dark"] {
    --b3-theme-background: #0f172a;
    --b3-theme-surface: #1e293b;
    --b3-theme-on-background: #f1f5f9;
    --b3-theme-on-surface: #cbd5e1;
}

/* カスタムコンポーネントスタイル */
.protyle-wysiwyg .h1 {
    color: var(--custom-primary);
    border-bottom: 2px solid var(--custom-primary);
    padding-bottom: 8px;
}

.protyle-wysiwyg .h2 {
    color: var(--custom-secondary);
}

.protyle-wysiwyg .bq {
    background-color: var(--b3-theme-surface);
    border-left-color: var(--custom-accent);
    border-radius: 0 var(--b3-border-radius) var(--b3-border-radius) 0;
}
/* ツールバーのカスタマイズ */
.toolbar {
    background: linear-gradient(90deg, var(--b3-theme-surface), var(--b3-theme-background));
    border-bottom-color: var(--custom-primary);
}

/* ダイアログのカスタマイズ */
.b3-dialog__container {
    border: 1px solid var(--custom-primary);
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 
                0 10px 10px -5px rgba(0, 0, 0, 0.04);
}
```

### アニメーション・トランジション

```css
/* スムーズなトランジション */
* {
    transition-property: color, background-color, border-color, box-shadow;
    transition-duration: 0.15s;
    transition-timing-function: ease-in-out;
}

/* カスタムアニメーション */
@keyframes slideInFromRight {
    from {
        transform: translateX(100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}

.b3-dialog {
    animation: slideInFromRight 0.3s ease-out;
}

/* ホバーエフェクト */
.toolbar__item:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
```

---

## 🔧 プラグインUI開発

### プラグインダイアログの作成

```typescript
import { Dialog } from "siyuan";

class CustomDialog extends Dialog {
    constructor() {
        super({
            title: "カスタムダイアログ",
            content: `
                <div class="custom-dialog">
                    <div class="custom-dialog__section">
                        <h3>設定セクション</h3>
                        <div class="b3-form">
                            <div class="b3-form__item">
                                <label class="b3-form__label">オプション名</label>
                                <input type="text" class="b3-text-field" id="option1">
                            </div>
                            <div class="b3-form__item">
                                <label class="b3-form__label">選択項目</label>
                                <div class="b3-select">
                                    <select id="option2">
                                        <option value="1">オプション1</option>
                                        <option value="2">オプション2</option>
                                    </select>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            `,
            width: "600px",
            height: "400px"
        });
        
        this.attachEvents();
    }
    
    private attachEvents() {
        // イベントハンドラの追加
    }
}
```

### カスタムドックパネル

```typescript
import { Plugin, IPluginDockTab } from "siyuan";

export default class MyPlugin extends Plugin {
    private customDock: IPluginDockTab;

    async onload() {
        this.customDock = this.addDock({
            config: {
                position: "LeftTop",
                size: { width: 250, height: 0 },
                icon: "iconPlugin",
                title: "カスタムパネル"
            },
            data: {
                text: "Custom Panel Data"
            },
            type: "custom-panel",
            init() {
                this.element.innerHTML = `
                    <div class="custom-dock">
                        <div class="custom-dock__header">
                            <h3 class="custom-dock__title">マイパネル</h3>
                            <div class="custom-dock__actions">
                                <button class="b3-button b3-button--small" id="refresh">
                                    <svg><use xlink:href="#iconRefresh"></use></svg>
                                </button>
                            </div>
                        </div>
                        <div class="custom-dock__content">
                            <ul class="b3-list">
                                <li class="b3-list-item">アイテム1</li>
                                <li class="b3-list-item">アイテム2</li>
                                <li class="b3-list-item">アイテム3</li>
                            </ul>
                        </div>
                    </div>
                `;
                
                // イベントリスナーの追加
                this.element.querySelector('#refresh')?.addEventListener('click', () => {
                    this.refresh();
                });
            },
            destroy() {
                // クリーンアップ処理
            }
        });
    }
}
```

### プラグイン用CSS

```css
/* カスタムドックのスタイル */
.custom-dock {
    height: 100%;
    display: flex;
    flex-direction: column;
    
    &__header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 8px 12px;
        border-bottom: 1px solid var(--b3-border-color);
        background-color: var(--b3-theme-surface);
    }
    
    &__title {
        font-size: 14px;
        font-weight: 500;
        margin: 0;
    }
    
    &__actions {
        display: flex;
        gap: 4px;
    }
    
    &__content {
        flex: 1;
        overflow: auto;
        padding: 8px;
    }
}

/* カスタムダイアログのスタイル */
.custom-dialog {
    padding: 16px;
    
    &__section {
        margin-bottom: 24px;
        
        h3 {
            margin: 0 0 16px 0;
            color: var(--b3-theme-on-surface);
            font-size: 16px;
            font-weight: 500;
        }
    }
}

/* フォームスタイル */
.b3-form {
    &__item {
        margin-bottom: 16px;
    }
    
    &__label {
        display: block;
        margin-bottom: 4px;
        color: var(--b3-theme-on-surface);
        font-weight: 500;
        font-size: 14px;
    }
}

/* 小さいボタン */
.b3-button--small {
    padding: 4px 8px;
    font-size: 12px;
    min-height: 24px;
}
```

---

## 📱 レスポンシブデザイン

### ブレイクポイント

```css
/* モバイルファースト設計 */

/* スマートフォン（デフォルト） */
/* 〜768px */

/* タブレット */
@media (min-width: 768px) {
    .layout__side {
        width: 250px;
    }
    
    .toolbar {
        padding: 0 8px;
    }
}

/* デスクトップ */
@media (min-width: 1024px) {
    .layout__side {
        width: 300px;
    }
    
    .protyle-wysiwyg {
        max-width: 800px;
        margin: 0 auto;
    }
}

/* 大画面 */
@media (min-width: 1440px) {
    .layout__center {
        max-width: 1200px;
        margin: 0 auto;
    }
}
```

### モバイル対応

```css
/* モバイルでの調整 */
@media (max-width: 767px) {
    .toolbar {
        height: 44px;
        line-height: 44px;
        padding: 0 16px;
    }
    
    .toolbar__item {
        min-width: 44px;
        min-height: 44px;
    }
    
    .layout__side {
        position: fixed;
        top: 44px;
        left: -100%;
        width: 80%;
        height: calc(100vh - 44px);
        background-color: var(--b3-theme-surface);
        transition: left 0.3s ease;
        z-index: 100;
        
        &--show {
            left: 0;
        }
    }
    
    .protyle-wysiwyg {
        padding: 16px;
        font-size: 16px; /* iOS Safariでのズーム防止 */
    }
    
    .b3-dialog__container {
        margin: 16px;
        max-width: calc(100% - 32px);
        max-height: calc(100% - 32px);
    }
}
```

---

## 🎯 ベストプラクティス

### パフォーマンス最適化

```css
/* GPU加速の活用 */
.animated-element {
    transform: translateZ(0); /* ハードウェア加速を強制 */
    will-change: transform, opacity; /* 変更予定のプロパティを宣言 */
}

/* 効率的なセレクター */
/* ❌ 非効率 */
.layout div.protyle div.protyle-wysiwyg div.p {
    /* スタイル */
}

/* ✅ 効率的 */
.protyle-wysiwyg .p {
    /* スタイル */
}

/* CSSカスタムプロパティの効率的な使用 */
:root {
    --spacing-unit: 8px;
    --spacing-small: calc(var(--spacing-unit) * 1);
    --spacing-medium: calc(var(--spacing-unit) * 2);
    --spacing-large: calc(var(--spacing-unit) * 3);
}
```

### アクセシビリティ

```css
/* フォーカス表示 */
.b3-button:focus,
.b3-text-field:focus {
    outline: 2px solid var(--b3-theme-primary);
    outline-offset: 2px;
}

/* 高コントラストモード対応 */
@media (prefers-contrast: high) {
    :root {
        --b3-theme-primary: #0000ff;
        --b3-theme-on-background: #000000;
        --b3-border-color: #000000;
    }
}

/* 動きを減らす設定への対応 */
@media (prefers-reduced-motion: reduce) {
    * {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}
```

### デバッグ支援

```css
/* 開発時のデバッグ用スタイル */
[data-debug="true"] > * {
    outline: 1px solid red !important;
}

[data-debug="true"] .layout {
    background-color: rgba(255, 0, 0, 0.1) !important;
}

[data-debug="true"] .protyle-wysiwyg > div {
    border: 1px dashed blue !important;
}
/* CSS Grid/Flexboxのデバッグ */
.debug-grid {
    background-image: 
        linear-gradient(rgba(255, 0, 0, 0.1) 1px, transparent 1px),
        linear-gradient(90deg, rgba(255, 0, 0, 0.1) 1px, transparent 1px);
    background-size: 20px 20px;
}
```

---

## 🔍 トラブルシューティング

### よくある問題と解決方法

#### 1. CSS変数が効かない

```css
/* ❌ 問題のあるコード */
:root {
    --my-color: red;
}

.my-element {
    color: --my-color; /* var()が不足 */
}

/* ✅ 修正されたコード */
.my-element {
    color: var(--my-color);
}
```

#### 2. z-indexの競合

```css
/* SiYuanのz-index階層を理解する */
/*
.status: 2
.protyle-util: 4
.protyle-hint: 5
.side-mask: 6
Mobile menu & .side-panel: 7
.fullscreen: 8
#windowControls: 9
window.siyuan.zIndex: 10
.b3-tooltips: 1000000
*/

/* カスタム要素には適切なz-indexを設定 */
.my-plugin-dialog {
    z-index: 100; /* プラグイン用の安全な値 */
}
```

#### 3. レスポンシブデザインの問題

```css
/* ビューポートメタタグの確認が必要 */
/* HTML: <meta name="viewport" content="width=device-width, initial-scale=1"> */

/* モバイルでのタッチターゲットサイズ */
@media (max-width: 767px) {
    .clickable-element {
        min-height: 44px; /* iOS推奨サイズ */
        min-width: 44px;
    }
}
```

---

## 📚 リファレンス

### 重要なCSSクラス一覧

#### レイアウト関連
- `.fn__flex` - フレックスボックス
- `.fn__flex-column` - 縦方向フレックス
- `.fn__flex-1` - flex: 1
- `.layout` - レイアウトコンテナ
- `.layout__center` - センターレイアウト
- `.layout__side` - サイドパネル

#### コンポーネント関連
- `.b3-button` - ボタン
- `.b3-text-field` - テキスト入力
- `.b3-select` - セレクトボックス
- `.b3-menu` - メニュー
- `.b3-dialog` - ダイアログ
- `.b3-list` - リスト
- `.b3-tooltips` - ツールチップ

#### プロトタイプ関連
- `.protyle` - エディターコンテナ
- `.protyle-wysiwyg` - 編集可能エリア
- `.protyle-breadcrumb` - ブレッドクラム
- `.protyle-toolbar` - エディターツールバー

### CSS変数クイックリファレンス

```css
/* 主要な色 */
--b3-theme-primary: メイン色
--b3-theme-secondary: セカンダリ色
--b3-theme-background: 背景色
--b3-theme-surface: サーフェス色
--b3-theme-error: エラー色
--b3-theme-success: 成功色

/* テキスト色 */
--b3-theme-on-primary: プライマリ上のテキスト
--b3-theme-on-background: 背景上のテキスト
--b3-theme-on-surface: サーフェス上のテキスト

/* レイアウト */
--b3-border-radius: 基本の角丸
--b3-border-color: ボーダー色
--b3-font-size: 基本フォントサイズ
--b3-font-family: 基本フォント
--b3-font-family-code: コードフォント
```

---

## 🎉 まとめ

このガイドでは、SiYuanのDOM構造とCSSアーキテクチャについて包括的に解説しました。

### 🔑 **重要なポイント**

1. **CSS変数システム** - SiYuanは包括的なCSS変数システムを使用
2. **コンポーネント指向** - 再利用可能なUIコンポーネント
3. **レスポンシブ設計** - モバイルファーストのアプローチ
4. **テーマサポート** - ライト・ダークテーマの完全サポート
5. **プラグイン統合** - プラグインとの seamless な統合

### 🚀 **次のステップ**

1. **実際にテーマを作成** - 基本テンプレートから始める
2. **プラグインUI開発** - カスタムダイアログやパネルを作成
3. **コミュニティ参加** - 他の開発者と交流し学習
4. **継続的な学習** - SiYuanの新機能に合わせて更新

---

## 📖 関連ドキュメント

### 開発ガイド
- **[プラグイン開発者向けAPIガイド](API_PLUGIN_DEVELOPERS_ja_JP.md)** - 完全なプラグイン開発ガイド
- **[プラグイン開発ベストプラクティス](PLUGIN_BEST_PRACTICES_ja_JP.md)** - 開発手法とコード品質

### APIリファレンス
- **[完全なAPIリファレンス](API_ja_JP.md)** - 全エンドポイントの詳細
- **[初心者向けAPIガイド](API_BEGINNERS_ja_JP.md)** - API使用の基本
- **[カテゴリ別APIリファレンス](API_REFERENCE_BY_CATEGORY_ja_JP.md)** - 機能別API一覧

### 外部リソース
- **[SiYuan 公式サイト](https://b3log.org/siyuan/)** - 最新情報とダウンロード
- **[GitHub リポジトリ](https://github.com/siyuan-note/siyuan)** - ソースコードと課題管理
- **[コミュニティフォーラム](https://liuyun.io/)** - ユーザーコミュニティ

---

**Happy Theming & Plugin Development! 🎨**

皆さんの創造性豊かなテーマとプラグインが、SiYuanエコシステムをより豊かにすることを楽しみにしています。

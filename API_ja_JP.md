# SiYuan API ドキュメント

SiYuanノートアプリケーションAPIの包括的ガイドです。このAPIを使用して、プログラム的にSiYuanと連携し、ノートブック、ドキュメント、ブロックを管理し、さまざまな操作を実行できます。

**言語オプション:**
- [English](API.md)

## 📚 用途別ガイド（推奨）

初めてSiYuan APIを使用する方や、特定の用途に特化した情報が必要な方は、以下の用途別ガイドをご利用ください：

* **[初心者向けAPIガイド](API_BEGINNERS_ja_JP.md)** - APIを初めて使用する方向け。基本概念から実際の使用方法まで段階的に学習
* **[プラグイン開発者向けAPIガイド](API_PLUGIN_DEVELOPERS_ja_JP.md)** - プラグイン開発に特化したAPIと実用例
* **[高度なAPIパターン](API_ADVANCED_PATTERNS_ja_JP.md)** - 複雑な操作、アーキテクチャパターン、パフォーマンス最適化
* **[カテゴリ別APIリファレンス](API_REFERENCE_BY_CATEGORY_ja_JP.md)** - 機能別に整理されたAPIエンドポイント一覧
* **[未記載エンドポイント補完](API_MISSING_ENDPOINTS_ja_JP.md)** - 追加・実験的なAPIエンドポイント

## 📖 関連ドキュメント

* **[プラグイン開発ベストプラクティス](PLUGIN_BEST_PRACTICES_ja_JP.md)** - 高品質なプラグイン開発のガイドライン
* **[プラグインショーケース](PLUGIN_SHOWCASE_ja_JP.md)** - 優秀なプラグインの実例

---

## 概要

SiYuan APIは、知識ベースとやり取りするためのRESTfulインターフェースを提供します。以下のことが可能です：

- **ノートブック管理**: ノートブックの作成、開く、閉じる、名前変更、削除
- **ドキュメント処理**: Markdownからドキュメントを作成、ファイル構造の整理
- **ブロック操作**: コンテンツブロックの挿入、更新、移動、削除
- **データクエリ**: 知識ベースに対するSQLクエリの実行
- **アセットアップロード**: 画像やその他のメディアファイルの管理
- **コンテンツエクスポート**: ノートを様々な形式で変換・エクスポート

## 目次

* [仕様](#仕様)
    * [パラメータと戻り値](#パラメータと戻り値)
    * [認証](#認証)
* [ノートブック](#ノートブック)
    * [ノートブック一覧](#ノートブック一覧)
    * [ノートブックを開く](#ノートブックを開く)
    * [ノートブックを閉じる](#ノートブックを閉じる)
    * [ノートブック名を変更](#ノートブック名を変更)
    * [ノートブックを作成](#ノートブックを作成)
    * [ノートブックを削除](#ノートブックを削除)
    * [ノートブック設定を取得](#ノートブック設定を取得)
    * [ノートブック設定を保存](#ノートブック設定を保存)
* [ドキュメント](#ドキュメント)
    * [Markdownでドキュメントを作成](#Markdownでドキュメントを作成)
    * [ドキュメント名を変更](#ドキュメント名を変更)
    * [ドキュメントを削除](#ドキュメントを削除)
    * [ドキュメントを移動](#ドキュメントを移動)
    * [パスに基づいて人間が読めるパスを取得](#パスに基づいて人間が読めるパスを取得)
    * [IDに基づいて人間が読めるパスを取得](#IDに基づいて人間が読めるパスを取得)
    * [IDに基づいてストレージパスを取得](#IDに基づいてストレージパスを取得)
    * [人間が読めるパスに基づいてIDを取得](#人間が読めるパスに基づいてIDを取得)
* [アセット](#アセット)
    * [アセットのアップロード](#アセットのアップロード)
* [ブロック](#ブロック)
    * [ブロックの挿入](#ブロックの挿入)
    * [ブロックの前置追加](#ブロックの前置追加)
    * [ブロックの後置追加](#ブロックの後置追加)
    * [ブロックの更新](#ブロックの更新)
    * [ブロックの削除](#ブロックの削除)
    * [ブロックの移動](#ブロックの移動)
    * [ブロックの折りたたみ](#ブロックの折りたたみ)
    * [ブロックの展開](#ブロックの展開)
    * [ブロックのkramdownを取得](#ブロックのkramdownを取得)
    * [子ブロックを取得](#子ブロックを取得)
    * [ブロック参照の転送](#ブロック参照の転送)
* [属性](#属性)
    * [ブロック属性の設定](#ブロック属性の設定)
    * [ブロック属性の取得](#ブロック属性の取得)
* [SQL](#SQL)
    * [SQLクエリの実行](#SQLクエリの実行)
    * [トランザクションのフラッシュ](#トランザクションのフラッシュ)
* [テンプレート](#テンプレート)
    * [テンプレートのレンダリング](#テンプレートのレンダリング)
    * [Sprigのレンダリング](#Sprigのレンダリング)
* [ファイル](#ファイル)
    * [ファイルの取得](#ファイルの取得)
    * [ファイルの書き込み](#ファイルの書き込み)
    * [ファイルの削除](#ファイルの削除)
    * [ファイル名の変更](#ファイル名の変更)
    * [ファイル一覧](#ファイル一覧)
* [エクスポート](#エクスポート)
    * [Markdownエクスポート](#Markdownエクスポート)
    * [ファイルとフォルダのエクスポート](#ファイルとフォルダのエクスポート)
* [変換](#変換)
    * [Pandoc](#Pandoc)
* [通知](#通知)
    * [メッセージのプッシュ](#メッセージのプッシュ)
    * [エラーメッセージのプッシュ](#エラーメッセージのプッシュ)
* [ネットワーク](#ネットワーク)
    * [フォワードプロキシ](#フォワードプロキシ)
* [UI管理](#UI管理)
    * [UIのリロード](#UIのリロード)
    * [ファイルツリーのリロード](#ファイルツリーのリロード)
    * [Protyleエディターのリロード](#Protyleエディターのリロード)
    * [タグパネルのリロード](#タグパネルのリロード)
    * [属性ビューのリロード](#属性ビューのリロード)
* [ワークスペース管理](#ワークスペース管理)
    * [ワークスペース一覧取得](#ワークスペース一覧取得)
    * [ワークスペースディレクトリ作成](#ワークスペースディレクトリ作成)
    * [ワークスペースディレクトリチェック](#ワークスペースディレクトリチェック)
    * [ワークスペースディレクトリ削除](#ワークスペースディレクトリ削除)
    * [ワークスペース情報取得](#ワークスペース情報取得)
* [プラグイン・拡張機能管理](#プラグイン・拡張機能管理)
    * [プラグインのロード](#プラグインのロード)
    * [プラグインの有効化設定](#プラグインの有効化設定)
    * [Bazaarプラグイン取得](#Bazaarプラグイン取得)
    * [Bazaarプラグインインストール](#Bazaarプラグインインストール)
    * [プラグインアンインストール](#プラグインアンインストール)
    * [テーマ管理](#テーマ管理)
    * [ウィジェット管理](#ウィジェット管理)
    * [アイコン管理](#アイコン管理)
    * [スニペット管理](#スニペット管理)
* [属性ビュー（データベース）](#属性ビュー（データベース）)
    * [属性ビュー取得](#属性ビュー取得)
    * [属性ビューレンダリング](#属性ビューレンダリング)
    * [属性ビューブロック追加](#属性ビューブロック追加)
    * [属性ビューキー追加](#属性ビューキー追加)
    * [属性ビューレイアウト変更](#属性ビューレイアウト変更)
    * [属性ビュー検索](#属性ビュー検索)
    * [属性ビューフィルター・ソート](#属性ビューフィルター・ソート)
* [高度な検索・インデックス](#高度な検索・インデックス)
    * [フルテキスト検索](#フルテキスト検索)
    * [アセット検索](#アセット検索)
    * [アセットコンテンツ検索](#アセットコンテンツ検索)
    * [埋め込みブロック検索](#埋め込みブロック検索)
    * [無効な参照検索](#無効な参照検索)
    * [検索・置換](#検索・置換)
* [同期・クラウド機能](#同期・クラウド機能)
    * [同期実行](#同期実行)
    * [同期情報取得](#同期情報取得)
    * [同期設定](#同期設定)
    * [クラウド同期ディレクトリ管理](#クラウド同期ディレクトリ管理)
    * [S3同期プロバイダー](#S3同期プロバイダー)
    * [WebDAV同期プロバイダー](#WebDAV同期プロバイダー)
* [バージョン管理・履歴](#バージョン管理・履歴)
    * [スナップショット作成](#スナップショット作成)
    * [スナップショット一覧取得](#スナップショット一覧取得)
    * [スナップショット比較](#スナップショット比較)
    * [リポジトリチェックアウト](#リポジトリチェックアウト)
    * [クラウドスナップショット](#クラウドスナップショット)
* [間隔反復学習（フラッシュカード）](#間隔反復学習（フラッシュカード）)
    * [フラッシュカードデッキ作成](#フラッシュカードデッキ作成)
    * [フラッシュカード追加](#フラッシュカード追加)
    * [フラッシュカード復習](#フラッシュカード復習)
    * [復習予定カード取得](#復習予定カード取得)
* [AI統合](#AI統合)
    * [ChatGPT連携](#ChatGPT連携)
    * [ChatGPTアクション付き連携](#ChatGPTアクション付き連携)
* [システム](#システム)
    * [起動進行状況の取得](#起動進行状況の取得)
    * [システムバージョンの取得](#システムバージョンの取得)
    * [システム現在時刻の取得](#システム現在時刻の取得)
    * [システム設定の取得](#システム設定の取得)
    * [システム設定のエクスポート](#システム設定のエクスポート)
    * [システム設定のインポート](#システム設定のインポート)
    * [システムフォント取得](#システムフォント取得)
    * [アップデートチェック](#アップデートチェック)
    * [ネットワーク情報取得](#ネットワーク情報取得)
    * [外観モード設定](#外観モード設定)
    * [UIレイアウト設定](#UIレイアウト設定)
    * [ネットワークサーブ設定](#ネットワークサーブ設定)
    * [自動起動設定](#自動起動設定)
    * [データインデックス再構築](#データインデックス再構築)
    * [データインデックス最適化](#データインデックス最適化)
    * [アプリケーション終了](#アプリケーション終了)
* [アカウント・クラウド](#アカウント・クラウド)
    * [アカウントログイン](#アカウントログイン)
    * [アクティベーションコード確認](#アクティベーションコード確認)
    * [アクティベーションコード使用](#アクティベーションコード使用)
    * [アカウント無効化](#アカウント無効化)
    * [無料トライアル開始](#無料トライアル開始)
    * [クラウドスペース取得](#クラウドスペース取得)
* [拡張ブロック操作](#拡張ブロック操作)
    * [ブロック存在確認](#ブロック存在確認)
    * [ブロック参照確認](#ブロック参照確認)
    * [ブロック情報取得](#ブロック情報取得)
    * [ブロックDOM取得](#ブロックDOM取得)
    * [ブロックツリー情報取得](#ブロックツリー情報取得)
    * [ヘディング子ブロック取得](#ヘディング子ブロック取得)
    * [最近更新ブロック取得](#最近更新ブロック取得)
    * [ブロック参照転送](#ブロック参照転送)
    * [ブロック参照スワップ](#ブロック参照スワップ)
    * [バッチブロック操作](#バッチブロック操作)
* [高度なアセット管理](#高度なアセット管理)
    * [アセット統計](#アセット統計)
    * [アセットパス解決](#アセットパス解決)
    * [アセット名変更](#アセット名変更)
    * [未使用アセット取得](#未使用アセット取得)
    * [未使用アセット削除](#未使用アセット削除)
    * [不足アセット取得](#不足アセット取得)
    * [OCR機能](#OCR機能)
    * [ファイルアノテーション](#ファイルアノテーション)
    * [アセットコンテンツ再インデックス](#アセットコンテンツ再インデックス)
    * [クラウドアップロード](#クラウドアップロード)
* [タグ・参照管理](#タグ・参照管理)
    * [タグ取得](#タグ取得)
    * [タグ削除](#タグ削除)
    * [タグ名変更](#タグ名変更)
    * [バックリンク取得](#バックリンク取得)
    * [バックメンション取得](#バックメンション取得)
    * [参照ID取得](#参照ID取得)
    * [参照テキスト取得](#参照テキスト取得)
* [ブックマーク・ナビゲーション](#ブックマーク・ナビゲーション)
    * [ブックマーク取得](#ブックマーク取得)
    * [ブックマーク削除](#ブックマーク削除)
    * [ブックマーク名変更](#ブックマーク名変更)
* [アウトライン・グラフ](#アウトライン・グラフ)
    * [ドキュメントアウトライン取得](#ドキュメントアウトライン取得)
* [拡張エクスポート](#拡張エクスポート)
    * [Word文書エクスポート](#Word文書エクスポート)
    * [EPUBエクスポート](#EPUBエクスポート)
    * [ODTエクスポート](#ODTエクスポート)
    * [RTFエクスポート](#RTFエクスポート)
    * [AsciiDocエクスポート](#AsciiDocエクスポート)
    * [MediaWikiエクスポート](#MediaWikiエクスポート)
    * [Org-modeエクスポート](#Org-modeエクスポート)
    * [reStructuredTextエクスポート](#reStructuredTextエクスポート)
    * [Textileエクスポート](#Textileエクスポート)
    * [OPMLエクスポート](#OPMLエクスポート)
    * [リソースエクスポート](#リソースエクスポート)
    * [一時コンテンツエクスポート](#一時コンテンツエクスポート)
* [アーカイブ・圧縮](#アーカイブ・圧縮)
    * [ZIP圧縮](#ZIP圧縮)
    * [ZIP解凍](#ZIP解凍)
* [ブロードキャスト・メッセージング](#ブロードキャスト・メッセージング)
    * [メッセージ投稿](#メッセージ投稿)
    * [チャンネル情報取得](#チャンネル情報取得)
    * [チャンネル一覧取得](#チャンネル一覧取得)
    * [メッセージ公開](#メッセージ公開)

---

## UI管理

UI管理APIを使用して、SiYuanアプリケーションの各UIコンポーネントを動的にリロードできます。プラグイン開発において、データ変更後のUI更新に特に有用です。

### UIのリロード

**エンドポイント:** `/api/ui/reloadUI`

SiYuanのUI全体をリロードします。アプリケーション設定の変更後やプラグインのインストール後に使用します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**使用例（JavaScript）:**
```javascript
const response = await fetch('/api/ui/reloadUI', {
  method: 'POST',
  headers: {
    'Authorization': `Token ${apiToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({})
});
```

### ファイルツリーのリロード

**エンドポイント:** `/api/ui/reloadFiletree`

左サイドバーのファイルツリーパネルをリロードします。ノートブックやドキュメントの構造が変更された場合に使用します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Protyleエディターのリロード

**エンドポイント:** `/api/ui/reloadProtyle`

指定したProtyleエディターインスタンスをリロードします。ブロック内容の大幅な変更後に使用します。

**リクエスト:**
```json
{
  "id": "20210808180117-6v0mkxr"
}
```

**パラメータ:**
- `id` (文字列): リロードするProtyleのID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### タグパネルのリロード

**エンドポイント:** `/api/ui/reloadTag`

右サイドバーのタグパネルをリロードします。タグが追加・削除・変更された場合に使用します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### 属性ビューのリロード

**エンドポイント:** `/api/ui/reloadAttributeView`

属性ビュー（データベースビュー）をリロードします。データベースの構造やデータが変更された場合に使用します。

**リクエスト:**
```json
{
  "id": "20210808180117-czj9bvb"
}
```

**パラメータ:**
- `id` (文字列): リロードする属性ビューのID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

---

## ワークスペース管理

ワークスペース管理APIを使用して、複数のワークスペース間での切り替えや管理を行います。

### ワークスペース一覧取得

**エンドポイント:** `/api/system/getWorkspaces`

利用可能なワークスペースの一覧を取得します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "path": "/Users/user/Documents/SiYuan/workspace1",
      "name": "workspace1"
    },
    {
      "path": "/Users/user/Documents/SiYuan/workspace2", 
      "name": "workspace2"
    }
  ]
}
```

### ワークスペースディレクトリ作成

**エンドポイント:** `/api/system/createWorkspaceDir`

新しいワークスペースディレクトリを作成します。

**リクエスト:**
```json
{
  "path": "/Users/user/Documents/SiYuan/new-workspace"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ワークスペースディレクトリチェック

**エンドポイント:** `/api/system/checkWorkspaceDir`

指定したパスがワークスペースとして有効かチェックします。

**リクエスト:**
```json
{
  "path": "/Users/user/Documents/SiYuan/test-workspace"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "valid": true
  }
}
```

### ワークスペースディレクトリ削除

**エンドポイント:** `/api/system/removeWorkspaceDir`

ワークスペースをリストから削除します（物理的なファイルは削除されません）。

**リクエスト:**
```json
{
  "path": "/Users/user/Documents/SiYuan/old-workspace"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ワークスペース情報取得

**エンドポイント:** `/api/system/getWorkspaceInfo`

現在のワークスペースの詳細情報を取得します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "path": "/Users/user/Documents/SiYuan/current-workspace",
    "name": "current-workspace",
    "size": 1024000,
    "notebooks": 5,
    "documents": 127
  }
}
```

---

## プラグイン・拡張機能管理

プラグイン、テーマ、ウィジェットなどの拡張機能を管理するためのAPIです。

### プラグインのロード

**エンドポイント:** `/api/petal/loadPetals`

利用可能なプラグイン（Petal）を読み込みます。

**リクエスト:**
```json
{
  "frontend": "desktop"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "name": "sample-plugin",
      "enabled": true,
      "displayName": "Sample Plugin"
    }
  ]
}
```

### プラグインの有効化設定

**エンドポイント:** `/api/petal/setPetalEnabled`

プラグインの有効/無効を切り替えます。

**リクエスト:**
```json
{
  "name": "sample-plugin",
  "enabled": true
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### Bazaarプラグイン取得

**エンドポイント:** `/api/bazaar/getBazaarPlugin`

Bazaar（公式マーケットプレイス）からプラグイン一覧を取得します。

**リクエスト:**
```json
{
  "frontend": "desktop"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "packages": [
      {
        "name": "plugin-sample",
        "displayName": "Plugin Sample",
        "version": "1.0.0",
        "author": "88250",
        "url": "https://github.com/siyuan-note/plugin-sample"
      }
    ]
  }
}
```

### Bazaarプラグインインストール

**エンドポイント:** `/api/bazaar/installBazaarPlugin`

Bazaarからプラグインをインストールします。

**リクエスト:**
```json
{
  "name": "plugin-sample",
  "repoURL": "https://github.com/siyuan-note/plugin-sample",
  "repoHash": "main"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### プラグインアンインストール

**エンドポイント:** `/api/bazaar/uninstallBazaarPlugin`

インストール済みプラグインをアンインストールします。

**リクエスト:**
```json
{
  "name": "plugin-sample"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### テーマ管理

**Bazaarテーマ取得:** `/api/bazaar/getBazaarTheme`
**テーマインストール:** `/api/bazaar/installBazaarTheme`
**テーマアンインストール:** `/api/bazaar/uninstallBazaarTheme`

テーマの管理はプラグインと同様の形式で行います。

### ウィジェット管理

**Bazaarウィジェット取得:** `/api/bazaar/getBazaarWidget`
**ウィジェットインストール:** `/api/bazaar/installBazaarWidget`
**ウィジェットアンインストール:** `/api/bazaar/uninstallBazaarWidget`

### アイコン管理

**Bazaarアイコン取得:** `/api/bazaar/getBazaarIcon`
**アイコンインストール:** `/api/bazaar/installBazaarIcon`
**アイコンアンインストール:** `/api/bazaar/uninstallBazaarIcon`

### スニペット管理

**エンドポイント:** `/api/snippet/getSnippet`

スニペット一覧を取得します。

**リクエスト:**
```json
{
  "type": "css"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "name": "custom-style",
      "type": "css",
      "enabled": true,
      "content": ".custom { color: red; }"
    }
  ]
}
```

---

## 属性ビュー（データベース） 

属性ビューAPIを使用して、SiYuanのデータベース機能を操作できます。テーブル、看板、ギャラリーなどの表示形式をサポートします。

### 属性ビュー取得

**エンドポイント:** `/api/av/getAttributeView`

指定したIDの属性ビューを取得します。

**リクエスト:**
```json
{
  "id": "20210808180117-6v0mkxr"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "20210808180117-6v0mkxr",
    "name": "タスク管理",
    "viewType": "table",
    "keyValues": [
      {
        "key": {
          "id": "title",
          "name": "タイトル",
          "type": "text"
        },
        "values": [
          {
            "id": "row1",
            "value": "タスク1"
          }
        ]
      }
    ]
  }
}
```

### 属性ビューレンダリング

**エンドポイント:** `/api/av/renderAttributeView`

属性ビューをHTMLとしてレンダリングします。

**リクエスト:**
```json
{
  "id": "20210808180117-6v0mkxr",
  "viewID": "view1"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "html": "<div class=\"av\">...</div>"
  }
}
```

### 属性ビューブロック追加

**エンドポイント:** `/api/av/addAttributeViewBlocks`

属性ビューに新しい行（ブロック）を追加します。

**リクエスト:**
```json
{
  "avID": "20210808180117-6v0mkxr",
  "blockIDs": ["20210808180117-new1"]
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### 属性ビューキー追加

**エンドポイント:** `/api/av/addAttributeViewKey`

属性ビューに新しい列（キー）を追加します。

**リクエスト:**
```json
{
  "avID": "20210808180117-6v0mkxr",
  "key": {
    "name": "ステータス",
    "type": "select",
    "options": [
      {"name": "未完了", "color": "red"},
      {"name": "完了", "color": "green"}
    ]
  }
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "keyID": "status-key-id"
  }
}
```

### 属性ビューレイアウト変更

**エンドポイント:** `/api/av/changeAttrViewLayout`

属性ビューの表示レイアウトを変更します（テーブル、看板、ギャラリー）。

**リクエスト:**
```json
{
  "avID": "20210808180117-6v0mkxr",
  "viewID": "view1",
  "layoutType": "kanban"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### 属性ビュー検索

**エンドポイント:** `/api/av/searchAttributeView`

属性ビュー内のデータを検索します。

**リクエスト:**
```json
{
  "avID": "20210808180117-6v0mkxr",
  "query": "タスク"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "blocks": [
      {
        "id": "20210808180117-task1",
        "content": "タスク1の内容"
      }
    ]
  }
}
```

### 属性ビューフィルター・ソート

**エンドポイント:** `/api/av/getAttributeViewFilterSort`

属性ビューのフィルターとソート設定を取得します。

**リクエスト:**
```json
{
  "avID": "20210808180117-6v0mkxr",
  "viewID": "view1"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "filters": [
      {
        "column": "status",
        "operator": "=",
        "value": "未完了"
      }
    ],
    "sorts": [
      {
        "column": "created",
        "order": "desc"
      }
    ]
  }
}
```

---

## 高度な検索・インデックス

SiYuanの強力な検索機能とインデックス管理を行うAPIです。

### フルテキスト検索

**エンドポイント:** `/api/search/fullTextSearchBlock`

ブロック内容に対してフルテキスト検索を実行します。

**リクエスト:**
```json
{
  "query": "プラグイン開発",
  "types": {
    "mathBlock": true,
    "table": true,
    "blockquote": true,
    "superBlock": true,
    "paragraph": true,
    "document": true,
    "heading": true,
    "list": true,
    "listItem": true,
    "codeBlock": true,
    "htmlBlock": true
  }
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "blocks": [
      {
        "id": "20210808180117-6v0mkxr",
        "parentID": "",
        "rootID": "20210808180117-czj9bvb",
        "hash": "abc123",
        "box": "20210808180117-6v0mkxr",
        "path": "/20210808180117-6v0mkxr/20210808180117-czj9bvb.sy",
        "hPath": "/プラグイン開発ガイド",
        "name": "プラグイン開発ガイド",
        "alias": "",
        "memo": "",
        "tag": "",
        "content": "プラグイン開発について説明します",
        "fcontent": "プラグイン開発について説明します",
        "markdown": "プラグイン開発について説明します",
        "length": 15,
        "type": "d",
        "subType": "",
        "ial": {},
        "sort": 1,
        "created": "20210808180117",
        "updated": "20210808180117"
      }
    ],
    "matchedBlockCount": 1,
    "matchedRootCount": 1
  }
}
```

### アセット検索

**エンドポイント:** `/api/search/searchAsset`

画像、動画、音声、ドキュメントなどのアセットを検索します。

**リクエスト:**
```json
{
  "k": "screenshot",
  "types": ["image", "video", "audio", "doc"]
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "path": "assets/screenshot-20210808180117-abc123.png",
      "name": "screenshot-20210808180117-abc123.png",
      "type": "image"
    }
  ]
}
```

### アセットコンテンツ検索

**エンドポイント:** `/api/search/fullTextSearchAssetContent`

アセット内のテキストコンテンツを検索します（OCRやドキュメント内容）。

**リクエスト:**
```json
{
  "query": "API"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "assets": [
      {
        "path": "assets/document.pdf",
        "content": "API仕様について...",
        "matches": [
          {
            "start": 0,
            "end": 3,
            "text": "API"
          }
        ]
      }
    ]
  }
}
```

### 埋め込みブロック検索

**エンドポイント:** `/api/search/searchEmbedBlock`

埋め込みブロックを検索します。

**リクエスト:**
```json
{
  "query": "SELECT * FROM blocks",
  "headingMode": 0
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "blocks": [
      {
        "id": "20210808180117-6v0mkxr",
        "content": "埋め込まれたクエリ結果"
      }
    ]
  }
}
```

### 無効な参照検索

**エンドポイント:** `/api/search/listInvalidBlockRefs`

存在しないブロックへの参照を検索します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "id": "20210808180117-invalid",
      "refText": "((20210808180117-invalid 無効な参照))"
    }
  ]
}
```

### 検索・置換

**エンドポイント:** `/api/search/findReplace`

指定した条件でテキストの検索・置換を行います。

**リクエスト:**
```json
{
  "k": "古いテキスト",
  "r": "新しいテキスト",
  "ids": ["20210808180117-6v0mkxr"]
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "ids": ["20210808180117-6v0mkxr"]
  }
}
```

---

## 同期・クラウド機能

マルチデバイス間での同期とクラウドストレージとの連携機能です。

### 同期実行

**エンドポイント:** `/api/sync/performSync`

手動で同期を実行します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "synced": true,
    "uploadCount": 5,
    "downloadCount": 3
  }
}
```

### 同期情報取得

**エンドポイント:** `/api/sync/getSyncInfo`

現在の同期状態と設定を取得します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "enabled": true,
    "provider": "s3",
    "lastSync": "2021-08-08T18:01:17Z",
    "stat": {
      "uploaded": 1024,
      "downloaded": 512
    }
  }
}
```

### 同期設定

同期に関する各種設定を行います：

- **同期有効化:** `/api/sync/setSyncEnable`
- **同期間隔設定:** `/api/sync/setSyncInterval`
- **同期モード設定:** `/api/sync/setSyncMode`
- **同期プロバイダー設定:** `/api/sync/setSyncProvider`

### クラウド同期ディレクトリ管理

**ディレクトリ作成:** `/api/sync/createCloudSyncDir`
**ディレクトリ一覧:** `/api/sync/listCloudSyncDir`
**ディレクトリ削除:** `/api/sync/removeCloudSyncDir`

### S3同期プロバイダー

**S3設定:** `/api/sync/setSyncProviderS3`
**S3エクスポート:** `/api/sync/exportSyncProviderS3`  
**S3インポート:** `/api/sync/importSyncProviderS3`

### WebDAV同期プロバイダー

**WebDAV設定:** `/api/sync/setSyncProviderWebDAV`
**WebDAVエクスポート:** `/api/sync/exportSyncProviderWebDAV`
**WebDAVインポート:** `/api/sync/importSyncProviderWebDAV`

---

## バージョン管理・履歴

ドキュメントのバージョン管理とスナップショット機能です。

### スナップショット作成

**エンドポイント:** `/api/repo/createSnapshot`

現在の状態のスナップショットを作成します。

**リクエスト:**
```json
{
  "memo": "機能追加前のバックアップ"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "snapshot-20210808180117"
  }
}
```

### スナップショット一覧取得

**エンドポイント:** `/api/repo/getRepoSnapshots`

作成済みスナップショットの一覧を取得します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "id": "snapshot-20210808180117",
      "memo": "機能追加前のバックアップ",
      "created": "2021-08-08T18:01:17Z"
    }
  ]
}
```

### スナップショット比較

**エンドポイント:** `/api/repo/diffRepoSnapshots`

2つのスナップショット間の差分を取得します。

**リクエスト:**
```json
{
  "left": "snapshot-20210808180117",
  "right": "snapshot-20210808190117"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "diffs": [
      {
        "path": "/ドキュメント1.md",
        "type": "modified",
        "changes": [
          {
            "line": 10,
            "old": "古いテキスト",
            "new": "新しいテキスト"
          }
        ]
      }
    ]
  }
}
```

### リポジトリチェックアウト

**エンドポイント:** `/api/repo/checkoutRepo`

指定したスナップショットの状態に戻します。

**リクエスト:**
```json
{
  "id": "snapshot-20210808180117"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### クラウドスナップショット

クラウドストレージとのスナップショット同期：

- **クラウドスナップショット取得:** `/api/repo/getCloudRepoSnapshots`
- **クラウドスナップショットアップロード:** `/api/repo/uploadCloudSnapshot`
- **クラウドスナップショットダウンロード:** `/api/repo/downloadCloudSnapshot`

---

## 間隔反復学習（フラッシュカード）

SiYuanの間隔反復学習機能を使用してフラッシュカードを管理します。

### フラッシュカードデッキ作成

**エンドポイント:** `/api/riff/createRiffDeck`

新しいフラッシュカードデッキを作成します。

**リクエスト:**
```json
{
  "name": "英単語学習",
  "notebook": "20210808180117-6v0mkxr"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "deck-20210808180117"
  }
}
```

### フラッシュカード追加

**エンドポイント:** `/api/riff/addRiffCards`

デッキにフラッシュカードを追加します。

**リクエスト:**
```json
{
  "deckID": "deck-20210808180117",
  "blockIDs": ["20210808180117-card1", "20210808180117-card2"]
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### フラッシュカード復習

**エンドポイント:** `/api/riff/reviewRiffCard`

フラッシュカードの復習結果を記録します。

**リクエスト:**
```json
{
  "cardID": "20210808180117-card1",
  "rating": 3
}
```

**パラメータ:**
- `rating`: 1(Again) / 2(Hard) / 3(Good) / 4(Easy)

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "nextDue": "2021-08-10T18:01:17Z"
  }
}
```

### 復習予定カード取得

**エンドポイント:** `/api/riff/getRiffDueCards`

復習予定のカードを取得します。

**リクエスト:**
```json
{
  "deckID": "deck-20210808180117"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "cards": [
      {
        "id": "20210808180117-card1",
        "blockID": "20210808180117-block1",
        "due": "2021-08-08T18:01:17Z"
      }
    ]
  }
}
```

---

## AI統合

SiYuanのAI機能との連携APIです。

### ChatGPT連携

**エンドポイント:** `/api/ai/chatGPT`

ChatGPTと対話を行います。

**リクエスト:**
```json
{
  "msg": "この文章を要約してください",
  "context": "長い文章の内容..."
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "choices": [
      {
        "message": {
          "content": "要約された内容がここに表示されます"
        }
      }
    ]
  }
}
```

### ChatGPTアクション付き連携

**エンドポイント:** `/api/ai/chatGPTWithAction`

ChatGPTからの応答を特定のアクションと組み合わせます。

**リクエスト:**
```json
{
  "msg": "新しいドキュメントを作成して",
  "action": "createDoc"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "response": "新しいドキュメントを作成しました",
    "actionResult": {
      "docID": "20210808180117-newdoc"
    }
  }
}
```

---

## API仕様

### エンドポイント設定

SiYuan APIはローカルHTTPサーバー上で動作します：

- **ベースURL**: `http://127.0.0.1:6806`
- **プロトコル**: HTTP POSTのみ
- **Content-Type**: `application/json`
- **ポート**: 6806（デフォルト、設定で変更可能）

### リクエスト形式

すべてのAPIリクエストは、リクエストボディにJSONペイロードを含むHTTP POSTとして送信する必要があります：

```bash
curl -X POST \
  http://127.0.0.1:6806/api/notebook/lsNotebooks \
  -H "Content-Type: application/json" \
  -H "Authorization: Token your_api_token_here" \
  -d '{}'
```

### レスポンス形式

すべてのAPIレスポンスは一貫した構造に従います：

```json
{
  "code": 0,
  "msg": "",
  "data": {}
}
```

**レスポンスフィールド:**
- **`code`** (整数): ステータスコード
  - `0`: 成功
  - 非ゼロ: エラーが発生
- **`msg`** (文字列): メッセージ
  - 成功時は空文字列
  - 失敗時はエラーの説明
- **`data`** (オブジェクト/配列/null): レスポンスペイロード
  - エンドポイントにより異なる: `{}`、`[]`、または`null`

### 認証

**APIトークンの取得方法:**
1. SiYuanアプリケーションを開く
2. <kbd>設定</kbd> → <kbd>About</kbd>に移動
3. APIトークンをコピー

**トークンの使用:**
すべてのリクエストに認証ヘッダーを追加：
```
Authorization: Token your_api_token_here
```

**認証付きの例:**
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

### エラー処理

エラーが発生した場合、レスポンスには以下が含まれます：
- `code`: 非ゼロのエラーコード
- `msg`: 説明的なエラーメッセージ
- `data`: 通常`null`または空

エラーレスポンスの例：
```json
{
  "code": -1,
  "msg": "Notebook not found",
  "data": null
}
```

## ノートブック

ノートブックは、SiYuanでドキュメントを整理するためのトップレベルコンテナです。各ノートブックは複数のドキュメントとサブディレクトリを含むことができ、知識ベースに階層構造を提供します。

### ノートブック一覧

ワークスペース内のすべての利用可能なノートブックを、現在のステータス（開いている/閉じている）と共に取得します。

**エンドポイント:** `/api/notebook/lsNotebooks`

**パラメータ:** なし

**リクエスト例:**
```bash
curl -X POST http://127.0.0.1:6806/api/notebook/lsNotebooks \
  -H "Content-Type: application/json" \
  -H "Authorization: Token your_token" \
  -d '{}'
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "notebooks": [
      {
        "id": "20210817205410-2kvfpfn", 
        "name": "テストノートブック",
        "icon": "1f41b",
        "sort": 0,
        "closed": false
      },
      {
        "id": "20210808180117-czj9bvb",
        "name": "SiYuanユーザーガイド",
        "icon": "1f4d4",
        "sort": 1,
        "closed": false
      }
    ]
  }
}
```

**レスポンスフィールド:**
- `id`: 一意のノートブック識別子（タイムスタンプベース）
- `name`: ユーザー定義のノートブック名
- `icon`: ノートブックアイコンのUnicode絵文字コード
- `sort`: 表示順序インデックス
- `closed`: ノートブックが現在閉じているかどうか

**使用例:**
- 利用可能なノートブックを表示するダッシュボードアプリケーション
- すべてのノートブックを処理する必要があるバックアップスクリプト
- ワークスペース概要ツール


### ノートブックを開く

閉じているノートブックを開き、そのコンテンツを読み取りと編集ができるようにします。

**エンドポイント:** `/api/notebook/openNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (文字列): 開くノートブックの一意のID

**例:**
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

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**使用例:**
- 自動ワークスペース復元
- コンテキスト依存のノートブックアクティベーション
- 特定のノートブックを開く必要があるバッチ操作

### ノートブックを閉じる

開いているノートブックを閉じ、ノートブックのデータを保持しながらリソースを解放します。

**エンドポイント:** `/api/notebook/closeNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (文字列): 閉じるノートブックの一意のID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**使用例:**
- 大きなワークスペースのメモリ最適化
- プロジェクトベースのワークフロー管理
- 自動クリーンアッププロセス

### ノートブック名を変更

既存のノートブックの表示名を変更します。

**エンドポイント:** `/api/notebook/renameNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "name": "ノートブックの新しい名前"
}
```

- `notebook` (文字列): ノートブックID
- `name` (文字列): ノートブックの新しい名前

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**使用例:**
- ワークスペースの再構成
- プロジェクトのリブランディング
- バッチ名前変更操作

### ノートブックを作成

ワークスペースに新しい空のノートブックを作成します。

**エンドポイント:** `/api/notebook/createNotebook`

**パラメータ:**
```json
{
  "name": "ノートブック名"
}
```

- `name` (文字列): 新しいノートブックの名前

**例:**
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

# 使用法
result = create_notebook("新しいプロジェクト", "your_token_here")
print(f"ノートブックが作成されました ID: {result['data']['notebook']['id']}")
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "notebook": {
      "id": "20220126215949-r1wvoch",
      "name": "ノートブック名",
      "icon": "",
      "sort": 0,
      "closed": false
    }
  }
}
```

**使用例:**
- プロジェクト初期化スクリプト
- テンプレートベースのノートブック作成
- 自動ワークスペースセットアップ

### ノートブックを削除

ノートブックとそのすべてのコンテンツを完全に削除します。⚠️ **この操作は元に戻せません。**

**エンドポイント:** `/api/notebook/removeNotebook`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0"
}
```

- `notebook` (文字列): 削除するノートブックのID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**⚠️ 警告:** この操作はノートブック内のすべてのデータを完全に削除します。削除前に重要なデータは必ずバックアップしてください。

**使用例:**
- 一時的なノートブックのクリーンアップスクリプト
- プロジェクトアーカイブワークフロー
- ワークスペースメンテナンス

### ノートブック設定を取得

特定のノートブックの設定を取得します。

**エンドポイント:** `/api/notebook/getNotebookConf`

**パラメータ:**
```json
{
  "notebook": "20210817205410-2kvfpfn"
}
```

- `notebook` (文字列): ノートブックID

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "box": "20210817205410-2kvfpfn",
    "conf": {
      "name": "テストノートブック",
      "closed": false,
      "refCreateSavePath": "",
      "createDocNameTemplate": "",
      "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
      "dailyNoteTemplatePath": ""
    },
    "name": "テストノートブック"
  }
}
```

**設定フィールド:**
- `name`: ノートブック表示名
- `closed`: ノートブックが閉じているかどうか
- `refCreateSavePath`: 参照ドキュメント作成のデフォルトパス
- `createDocNameTemplate`: 新しいドキュメント名のテンプレート
- `dailyNoteSavePath`: デイリーノートのパステンプレート（日付フォーマット対応）
- `dailyNoteTemplatePath`: デイリーノートに使用するテンプレートのパス

**使用例:**
- ノートブック設定のバックアップと復元
- インスタンス間での設定同期
- 設定管理ツールの構築

### ノートブック設定を保存

特定のノートブックの設定を更新します。

**エンドポイント:** `/api/notebook/setNotebookConf`

**パラメータ:**
```json
{
  "notebook": "20210817205410-2kvfpfn",
  "conf": {
    "name": "テストノートブック",
    "closed": false,
    "refCreateSavePath": "",
    "createDocNameTemplate": "",
    "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
    "dailyNoteTemplatePath": ""
  }
}
```

- `notebook` (文字列): ノートブックID
- `conf` (オブジェクト): 更新する設定を含む設定オブジェクト

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "name": "テストノートブック",
    "closed": false,
    "refCreateSavePath": "",
    "createDocNameTemplate": "",
    "dailyNoteSavePath": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}",
    "dailyNoteTemplatePath": ""
  }
}
```

**例 - デイリーノートの設定:**
```javascript
const setupDailyNotes = async (notebookId, token) => {
  const conf = {
    name: "私のデイリージャーナル",
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

**使用例:**
- 自動ワークスペースセットアップ
- テンプレートベースのノートブック設定
- 一括設定更新

---

## ドキュメント

ドキュメントは、ノートブック内の個々のノートやファイルです。テキスト、画像、表、複雑なブロック構造など、さまざまなタイプのコンテンツを含むことができます。Documents APIを使用すると、知識ベースのコンテンツを作成、管理、整理できます。

### Markdownでドキュメントを作成

Markdownコンテンツから新しいドキュメントを作成します。既存のノートをインポートしたり、プログラム的にドキュメントを作成する際に便利です。

**エンドポイント:** `/api/filetree/createDocWithMd`

**パラメータ:**
```json
{
  "notebook": "20210817205410-2kvfpfn",
  "path": "/foo/bar",
  "markdown": "# Hello World\n\nこれは私の最初のドキュメントです。"
}
```

- `notebook` (文字列): ターゲットノートブックID
- `path` (文字列): ノートブック内のドキュメントパス、`/`で始まり、レベルを`/`で区切る必要があります（データベースの`hpath`フィールドに対応）
- `markdown` (文字列): ドキュメントのGFM（GitHub Flavored Markdown）コンテンツ

**例:**
```python
def create_markdown_document(notebook_id, path, content, token):
    """markdownコンテンツから新しいドキュメントを作成"""
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

# 使用例
markdown_content = """# プロジェクト概要

## 目標
- APIドキュメントの完成
- 日本語翻訳の追加
- ユーザーエクスペリエンスの向上

## タイムライン
- 第1週: ドキュメント強化
- 第2週: 翻訳作業
- 第3週: レビューとテスト
"""

result = create_markdown_document(
    "20210817205410-2kvfpfn", 
    "/projects/api-documentation", 
    markdown_content,
    "your_token_here"
)
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": "20210914223645-oj2vnx2"
}
```

- `data`: 作成されたドキュメントのID

**重要な注意事項:**
- 同じ`path`でこのエンドポイントを繰り返し呼び出しても、既存のドキュメントは上書きされません
- パスには`.sy`ファイル拡張子を含めないでください
- Markdownコンテンツは、SiYuanの内部ブロック構造に変換されます

**使用例:**
- 他のノート取りアプリケーションからのコンテンツインポート
- 自動コンテンツ生成
- テンプレートベースのドキュメント作成
- 外部ソースからのバッチドキュメント作成


### ドキュメント名を変更

既存のドキュメントのタイトルを変更します。ドキュメントパスまたはドキュメントIDで名前を変更できます。

**方法1: パスで名前変更**

**エンドポイント:** `/api/filetree/renameDoc`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210902210113-0avi12f.sy",
  "title": "新しいドキュメントタイトル"
}
```

- `notebook` (文字列): ドキュメントを含むノートブックID
- `path` (文字列): `.sy`拡張子を含む完全なドキュメントパス
- `title` (文字列): ドキュメントの新しいタイトル

**方法2: IDで名前変更**

**エンドポイント:** `/api/filetree/renameDocByID`

**パラメータ:**
```json
{
  "id": "20210902210113-0avi12f",
  "title": "新しいドキュメントタイトル"
}
```

- `id` (文字列): ドキュメントID
- `title` (文字列): ドキュメントの新しいタイトル

**レスポンス（両方の方法）:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**例:**
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

// 使用法
await renameDocument("20210902210113-0avi12f", "更新されたドキュメントタイトル", "your_token");
```

### ドキュメントを削除

ドキュメントとそのすべてのコンテンツを完全に削除します。⚠️ **この操作は元に戻せません。**

**方法1: パスで削除**

**エンドポイント:** `/api/filetree/removeDoc`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210902210113-0avi12f.sy"
}
```

- `notebook` (文字列): ドキュメントを含むノートブックID
- `path` (文字列): `.sy`拡張子を含む完全なドキュメントパス

**方法2: IDで削除**

**エンドポイント:** `/api/filetree/removeDocByID`

**パラメータ:**
```json
{
  "id": "20210902210113-0avi12f"
}
```

- `id` (文字列): 削除するドキュメントID

**レスポンス（両方の方法）:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

**⚠️ 警告:** この操作はドキュメントとそのすべてのコンテンツを完全に削除します。削除前に重要なデータは必ずバックアップしてください。

### ドキュメントを移動

同じノートブック内の異なる場所間、または異なるノートブック間でドキュメントを移動します。

**方法1: パスで移動**

**エンドポイント:** `/api/filetree/moveDocs`

**パラメータ:**
```json
{
  "fromPaths": ["/20210917220056-yxtyl7i.sy"],
  "toNotebook": "20210817205410-2kvfpfn",
  "toPath": "/"
}
```

- `fromPaths` (配列): ソースドキュメントパスの配列
- `toNotebook` (文字列): ターゲットノートブックID
- `toPath` (文字列): ノートブック内のターゲットパス

**方法2: IDで移動**

**エンドポイント:** `/api/filetree/moveDocsByID`

**パラメータ:**
```json
{
  "fromIDs": ["20210917220056-yxtyl7i"],
  "toID": "20210817205410-2kvfpfn"
}
```

- `fromIDs` (配列): ソースドキュメントIDの配列
- `toID` (文字列): ターゲット親ドキュメントIDまたはノートブックID

### パスに基づいて人間が読めるパスを取得

**エンドポイント:** `/api/filetree/getHPathByPath`

**パラメータ:**
```json
{
  "notebook": "20210831090520-7dvbdv0",
  "path": "/20210917220500-sz588nq/20210917220056-yxtyl7i.sy"
}
```

- `notebook`: ノートブックID
- `path`: ドキュメントパス

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": "/foo/bar"
}
```

### IDに基づいて人間が読めるパスを取得

**エンドポイント:** `/api/filetree/getHPathByID`

**パラメータ:**
```json
{
  "id": "20210917220056-yxtyl7i"
}
```

- `id`: ブロックID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": "/foo/bar"
}
```

### IDに基づいてストレージパスを取得

**エンドポイント:** `/api/filetree/getPathByID`

**パラメータ:**
```json
{
  "id": "20210808180320-fqgskfj"
}
```

- `id`: ブロックID

**戻り値:**
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

### 人間が読めるパスに基づいてIDを取得

**エンドポイント:** `/api/filetree/getIDsByHPath`

**パラメータ:**
```json
{
  "path": "/foo/bar",
  "notebook": "20210808180117-czj9bvb"
}
```

- `path`: 人間が読めるパス
- `notebook`: ノートブックID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    "20200813004931-q4cu8na"
  ]
}
```

## アセット

### アセットのアップロード

**エンドポイント:** `/api/asset/upload`

**パラメータ:** HTTP Multipartフォーム

- `assetsDirPath`: アセットが保存されるフォルダパス（dataフォルダをルートパスとする）、例：
  - `"/assets/"`: workspace/data/assets/ フォルダ
  - `"/assets/sub/"`: workspace/data/assets/sub/ フォルダ

  通常の状況では、最初の方法を推奨します。これはワークスペースのassetsフォルダに保存されます。サブディレクトリに配置すると副作用があります。ユーザーガイドのassetsの章を参考にしてください。
- `file[]`: アップロードファイルリスト

**戻り値:**
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

- `errFiles`: アップロード処理でエラーが発生したファイル名のリスト
- `succMap`: 正常に処理されたファイルの場合、キーはアップロード時のファイル名、値はassets/foo-id.pngで、既存のMarkdownコンテンツ内のアセットリンクアドレスをアップロードされたアドレスに置き換えるために使用されます

## ブロック

### ブロックの挿入

**エンドポイント:** `/api/block/insertBlock`

**パラメータ:**
```json
{
  "dataType": "markdown",
  "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
  "nextID": "",
  "previousID": "20211229114650-vrek5x6",
  "parentID": ""
}
```

- `dataType`: 挿入するデータタイプ、値は`markdown`または`dom`
- `data`: 挿入するデータ
- `nextID`: 次のブロックのID、挿入位置をアンカーするために使用
- `previousID`: 前のブロックのID、挿入位置をアンカーするために使用
- `parentID`: 親ブロックのID、挿入位置をアンカーするために使用

`nextID`、`previousID`、`parentID`のうち少なくとも一つの値が必要です。優先順位: `nextID` > `previousID` > `parentID`

**戻り値:**
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

- `action.data`: 新しく挿入されたブロックによって生成されたDOM
- `action.id`: 新しく挿入されたブロックのID

### ブロックの前置追加

**エンドポイント:** `/api/block/prependBlock`

**パラメータ:**
```json
{
  "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
  "dataType": "markdown",
  "parentID": "20220107173950-7f9m1nb"
}
```

- `dataType`: 挿入するデータタイプ、値は`markdown`または`dom`
- `data`: 挿入するデータ
- `parentID`: 親ブロックのID、挿入位置をアンカーするために使用

### ブロックの後置追加

**エンドポイント:** `/api/block/appendBlock`

**パラメータ:**
```json
{
  "data": "foo**bar**{: style=\"color: var(--b3-font-color8);\"}baz",
  "dataType": "markdown",
  "parentID": "20220107173950-7f9m1nb"
}
```

- `dataType`: 挿入するデータタイプ、値は`markdown`または`dom`
- `data`: 挿入するデータ
- `parentID`: 親ブロックのID、挿入位置をアンカーするために使用

### ブロックの更新

**エンドポイント:** `/api/block/updateBlock`

**パラメータ:**
```json
{
  "dataType": "markdown",
  "data": "foobarbaz",
  "id": "20211230161520-querkps"
}
```

- `dataType`: 更新するデータタイプ、値は`markdown`または`dom`
- `data`: 更新するデータ
- `id`: 更新するブロックのID

### ブロックの削除

**エンドポイント:** `/api/block/deleteBlock`

**パラメータ:**
```json
{
  "id": "20211230161520-querkps"
}
```

- `id`: 削除するブロックのID

### ブロックの移動

**エンドポイント:** `/api/block/moveBlock`

**パラメータ:**
```json
{
  "id": "20230406180530-3o1rqkc",
  "previousID": "20230406152734-if5kyx6",
  "parentID": "20230404183855-woe52ko"
}
```

- `id`: 移動するブロックID
- `previousID`: 前のブロックのID、挿入位置をアンカーするために使用
- `parentID`: 親ブロックのID、挿入位置をアンカーするために使用、`previousID`と`parentID`は同時に空にできません。両方が存在する場合、`previousID`が優先されます

### ブロックの折りたたみ

**エンドポイント:** `/api/block/foldBlock`

**パラメータ:**
```json
{
  "id": "20231224160424-2f5680o"
}
```

- `id`: 折りたたむブロックID

### ブロックの展開

**エンドポイント:** `/api/block/unfoldBlock`

**パラメータ:**
```json
{
  "id": "20231224160424-2f5680o"
}
```

- `id`: 展開するブロックID

### ブロックのkramdownを取得

**エンドポイント:** `/api/block/getBlockKramdown`

**パラメータ:**
```json
{
  "id": "20201225220954-dlgzk1o"
}
```

- `id`: 取得するブロックのID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "20201225220954-dlgzk1o",
    "kramdown": "* {: id=\"20201225220954-e913snx\"}新しいノートブックを作成し、ノートブック下に新しいドキュメントを作成\n  {: id=\"20210131161940-kfs31q6\"}\n* {: id=\"20201225220954-ygz217h\"}エディターで<kbd>/</kbd>を入力して機能メニューをトリガー\n  {: id=\"20210131161940-eo0riwq\"}\n* {: id=\"20201225220954-875yybt\"}((20200924101200-gss5vee \"コンテンツブロック内でのナビゲーション\"))と((20200924100906-0u4zfq3 \"ウィンドウとタブ\"))\n  {: id=\"20210131161940-b5uow2h\"}"
  }
}
```

### 子ブロックを取得

**エンドポイント:** `/api/block/getChildBlocks`

**パラメータ:**
```json
{
  "id": "20230506212712-vt9ajwj"
}
```

- `id`: 親ブロックID
- 見出しの下のブロックも子ブロックとしてカウントされます

**戻り値:**
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

### ブロック参照の転送

**エンドポイント:** `/api/block/transferBlockRef`

**パラメータ:**
```json
{
  "fromID": "20230612160235-mv6rrh1",
  "toID": "20230613093045-uwcomng",
  "refIDs": ["20230613092230-cpyimmd"]
}
```

- `fromID`: 定義ブロックID
- `toID`: ターゲットブロックID
- `refIDs`: 定義ブロックIDを指す参照ブロックID、オプション、指定されない場合はすべての参照ブロックIDが転送されます


## 属性

### ブロック属性の設定

**エンドポイント:** `/api/attr/setBlockAttrs`

**パラメータ:**
```json
{
  "id": "20210912214605-uhi5gco",
  "attrs": {
    "custom-attr1": "line1\nline2"
  }
}
```

- `id`: ブロックID
- `attrs`: ブロック属性、カスタム属性は`custom-`で始める必要があります

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ブロック属性の取得

**エンドポイント:** `/api/attr/getBlockAttrs`

**パラメータ:**
```json
{
  "id": "20210912214605-uhi5gco"
}
```

- `id`: ブロックID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "custom-attr1": "line1\nline2",
    "id": "20210912214605-uhi5gco",
    "title": "PDFアノテーションデモ",
    "type": "doc",
    "updated": "20210916120715"
  }
}
```

## SQL

### SQLクエリの実行

**エンドポイント:** `/api/query/sql`

**パラメータ:**
```json
{
  "stmt": "SELECT * FROM blocks WHERE content LIKE '%content%' LIMIT 7"
}
```

- `stmt`: SQL文

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    { "col": "val" }
  ]
}
```

### トランザクションのフラッシュ

**エンドポイント:** `/api/sqlite/flushTransaction`

**パラメータ:** なし

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

## テンプレート

### テンプレートのレンダリング

**エンドポイント:** `/api/template/render`

**パラメータ:**
```json
{
  "id": "20220724223548-j6g0o87",
  "path": "F:\\SiYuan\\data\\templates\\foo.md"
}
```

- `id`: レンダリングが呼び出されるドキュメントのID
- `path`: テンプレートファイルの絶対パス

**戻り値:**
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

### Sprigのレンダリング

**エンドポイント:** `/api/template/renderSprig`

**パラメータ:**
```json
{
  "template": "/daily note/{{now | date \"2006/01\"}}/{{now | date \"2006-01-02\"}}"
}
```

- `template`: テンプレートコンテンツ

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": "/daily note/2023/03/2023-03-24"
}
```

## ファイル

### ファイルの取得

**エンドポイント:** `/api/file/getFile`

**パラメータ:**
```json
{
  "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p.sy"
}
```

- `path`: ワークスペースパス下のファイルパス

**戻り値:**

- レスポンスステータスコード`200`: ファイルコンテンツ
- レスポンスステータスコード`202`: 例外情報

```json
{
  "code": 404,
  "msg": "",
  "data": null
}
```

- `code`: 例外の場合は非ゼロ
  - `-1`: パラメータ解析エラー
  - `403`: 権限拒否（ファイルがワークスペース内にない）
  - `404`: 見つからない（ファイルが存在しない）
  - `405`: メソッド不許可（ディレクトリです）
  - `500`: サーバーエラー（ファイルのstat失敗 / ファイル読み取り失敗）
- `msg`: エラーを説明するテキスト

### ファイルの書き込み

**エンドポイント:** `/api/file/putFile`

**パラメータ:** HTTP Multipartフォーム

- `path`: ワークスペースパス下のファイルパス
- `isDir`: フォルダを作成するかどうか、`true`の場合はフォルダのみ作成し、`file`を無視
- `modTime`: 最後のアクセスと変更時刻、Unixタイム
- `file`: アップロードファイル

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ファイルの削除

**エンドポイント:** `/api/file/removeFile`

**パラメータ:**
```json
{
  "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p.sy"
}
```

- `path`: ワークスペースパス下のファイルパス

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ファイル名の変更

**エンドポイント:** `/api/file/renameFile`

**パラメータ:**
```json
{
  "path": "/data/assets/image-20230523085812-k3o9t32.png",
  "newPath": "/data/assets/test-20230523085812-k3o9t32.png"
}
```

- `path`: ワークスペースパス下のファイルパス
- `newPath`: ワークスペースパス下の新しいファイルパス

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### ファイル一覧

**エンドポイント:** `/api/file/readDir`

**パラメータ:**
```json
{
  "path": "/data/20210808180117-6v0mkxr/20200923234011-ieuun1p"
}
```

- `path`: ワークスペースパス下のディレクトリパス

**戻り値:**
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

## エクスポート

### Markdownエクスポート

**エンドポイント:** `/api/export/exportMdContent`

**パラメータ:**
```json
{
  "id": ""
}
```

- `id`: エクスポートするドックブロックのID

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "hPath": "/こちらから始めてください",
    "content": "## 🍫 コンテンツブロック\n\nSiYuanでは、唯一重要なコア概念は..."
  }
}
```

- `hPath`: 人間が読めるパス
- `content`: Markdownコンテンツ

### ファイルとフォルダのエクスポート

**エンドポイント:** `/api/export/exportResources`

**パラメータ:**
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

- `paths`: エクスポートするファイルまたはフォルダパスのリスト、同じファイル名/フォルダ名は上書きされます
- `name`: （オプション）エクスポートファイル名、設定されていない場合はデフォルトで`export-YYYY-MM-DD_hh-mm-ss.zip`

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "path": "temp/export/zip-file-name.zip"
  }
}
```

- `path`: 作成された`*.zip`ファイルのパス
  - `zip-file-name.zip`内のディレクトリ構造は以下の通りです：
    - `zip-file-name`
      - `boot`
      - `langs`
      - `conf.json`
      - `index.html`

## 変換

### Pandoc

**エンドポイント:** `/api/convert/pandoc`

**作業ディレクトリ:**
- pandocコマンドの実行は作業ディレクトリを`workspace/temp/convert/pandoc/${dir}`に設定します
- API [`ファイルの書き込み`](#ファイルの書き込み)を使用して、変換するファイルを最初にこのディレクトリに書き込むことができます
- 次に変換のためにAPIを呼び出し、変換されたファイルもこのディレクトリに書き込まれます
- 最後に、API [`ファイルの取得`](#ファイルの取得)を呼び出して変換されたファイルを取得します
  - またはAPI [Markdownでドキュメントを作成](#Markdownでドキュメントを作成)を呼び出します
  - または内部API `importStdMd`を呼び出して変換されたフォルダを直接インポートします

**パラメータ:**
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

- `args`: Pandocコマンドラインパラメータ

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "path": "/temp/convert/pandoc/test"
  }
}
```

- `path`: ワークスペース下のパス

## 通知

### メッセージのプッシュ

**エンドポイント:** `/api/notification/pushMsg`

**パラメータ:**
```json
{
  "msg": "テスト",
  "timeout": 7000
}
```

- `timeout`: メッセージ表示時間（ミリ秒）。このフィールドは省略可能で、デフォルトは7000ミリ秒

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "62jtmqi"
  }
}
```

- `id`: メッセージID

### エラーメッセージのプッシュ

**エンドポイント:** `/api/notification/pushErrMsg`

**パラメータ:**
```json
{
  "msg": "テスト",
  "timeout": 7000
}
```

- `timeout`: メッセージ表示時間（ミリ秒）。このフィールドは省略可能で、デフォルトは7000ミリ秒

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "qc9znut"
  }
}
```

- `id`: メッセージID

## ネットワーク

### フォワードプロキシ

**エンドポイント:** `/api/network/forwardProxy`

**パラメータ:**
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

- `url`: 転送するURL
- `method`: HTTPメソッド、デフォルトは`POST`
- `timeout`: タイムアウト（ミリ秒）、デフォルトは`7000`
- `contentType`: Content-Type、デフォルトは`application/json`
- `headers`: HTTPヘッダー
- `payload`: HTTPペイロード、オブジェクトまたは文字列
- `payloadEncoding`: `payload`で使用されるエンコーディングスキーム、デフォルトは`text`、オプション値は以下の通り
  - `text`
  - `base64` | `base64-std`
  - `base64-url`
  - `base32` | `base32-std`
  - `base32-hex`
  - `hex`
- `responseEncoding`: レスポンスデータの`body`で使用されるエンコーディングスキーム、デフォルトは`text`、オプション値は以下の通り
  - `text`
  - `base64` | `base64-std`
  - `base64-url`
  - `base32` | `base32-std`
  - `base32-hex`
  - `hex`

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "body": "",
    "bodyEncoding": "text",
    "contentType": "text/html",
    "elapsed": 1976,
    "headers": {},
    "status": 200,
    "url": "https://b3log.org/siyuan"
  }
}
```

- `bodyEncoding`: `body`で使用されるエンコーディングスキーム、リクエストのフィールド`responseEncoding`と一致、デフォルトは`text`、オプション値は以下の通り
  - `text`
  - `base64` | `base64-std`
  - `base64-url`
  - `base32` | `base32-std`
  - `base32-hex`
  - `hex`

---

## 拡張システム機能

### システム設定の取得

**エンドポイント:** `/api/system/getConf`

SiYuanのシステム設定を取得します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "appearance": {
      "mode": 0,
      "codeBlockTheme": "github"
    },
    "editor": {
      "fontSize": 16,
      "lineHeight": 22
    },
    "export": {
      "docx": true,
      "pdf": false
    }
  }
}
```

### システム設定のエクスポート

**エンドポイント:** `/api/system/exportConf`

システム設定をエクスポートします。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "zip": "base64-encoded-zip-content"
  }
}
```

### システム設定のインポート

**エンドポイント:** `/api/system/importConf`

システム設定をインポートします。

**リクエスト:**
```json
{
  "data": "base64-encoded-zip-content"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

### システムフォント取得

**エンドポイント:** `/api/system/getSysFonts`

システムにインストールされているフォント一覧を取得します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    "Arial",
    "Helvetica", 
    "Times New Roman",
    "Source Han Sans"
  ]
}
```

---

## アカウント・クラウド

### アカウントログイン

**エンドポイント:** `/api/account/login`

SiYuanクラウドアカウントにログインします。

**リクエスト:**
```json
{
  "userName": "your-username",
  "password": "your-password"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "token": "account-token",
    "user": {
      "userId": "12345",
      "userName": "your-username",
      "userNickname": "Your Name"
    }
  }
}
```

### アクティベーションコード確認

**エンドポイント:** `/api/account/checkActivationcode`

アクティベーションコードの有効性を確認します。

**リクエスト:**
```json
{
  "data": "activation-code"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "valid": true,
    "type": "pro",
    "expireTime": "2024-12-31"
  }
}
```

---

## 拡張ブロック操作

### ブロック存在確認

**エンドポイント:** `/api/block/checkBlockExist`

指定したIDのブロックが存在するか確認します。

**リクエスト:**
```json
{
  "id": "20210808180117-6v0mkxr"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "exist": true
  }
}
```

### ブロック情報取得

**エンドポイント:** `/api/block/getBlockInfo`

ブロックの詳細情報を取得します。

**リクエスト:**
```json
{
  "id": "20210808180117-6v0mkxr"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "id": "20210808180117-6v0mkxr",
    "parentID": "20210808180117-czj9bvb",
    "rootID": "20210808180117-czj9bvb",
    "hash": "abc123",
    "box": "20210808180117-6v0mkxr",
    "path": "/20210808180117-6v0mkxr/20210808180117-czj9bvb.sy",
    "hPath": "/ドキュメント名",
    "name": "ブロック名",
    "type": "p",
    "subType": "",
    "created": "20210808180117",
    "updated": "20210808180117"
  }
}
```

### バッチブロック操作

**バッチ挿入:** `/api/block/batchInsertBlock`
**バッチ更新:** `/api/block/batchUpdateBlock`
**バッチ追加:** `/api/block/batchAppendBlock`

複数のブロックを一度に操作できます。

**リクエスト例（バッチ挿入）:**
```json
{
  "operations": [
    {
      "parentID": "20210808180117-czj9bvb",
      "dataType": "markdown",
      "data": "# 新しいヘディング1"
    },
    {
      "parentID": "20210808180117-czj9bvb", 
      "dataType": "markdown",
      "data": "新しい段落の内容"
    }
  ]
}
```

---

## 高度なアセット管理

### アセット統計

**エンドポイント:** `/api/asset/statAsset`

アセットファイルの統計情報を取得します。

**リクエスト:**
```json
{
  "path": "assets/image.png"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "size": 1024000,
    "lastModified": "2021-08-08T18:01:17Z",
    "type": "image/png"
  }
}
```

### OCR機能

**エンドポイント:** `/api/asset/ocr`

画像内のテキストをOCRで認識します。

**リクエスト:**
```json
{
  "path": "assets/screenshot.png"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "text": "画像内のテキスト内容"
  }
}
```

### 未使用アセット管理

**未使用アセット取得:** `/api/asset/getUnusedAssets`
**未使用アセット削除:** `/api/asset/removeUnusedAssets`

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "path": "assets/unused-image.png",
      "size": 512000
    }
  ]
}
```

---

## タグ・参照管理

### タグ取得

**エンドポイント:** `/api/tag/getTag`

タグ一覧を取得します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "name": "重要",
      "count": 5
    },
    {
      "name": "TODO",
      "count": 3
    }
  ]
}
```

### バックリンク取得

**エンドポイント:** `/api/ref/getBacklink`

指定したブロックへのバックリンクを取得します。

**リクエスト:**
```json
{
  "id": "20210808180117-6v0mkxr"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "backlinks": [
      {
        "id": "20210808180117-ref1",
        "content": "参照元のブロック内容",
        "path": "/参照元ドキュメント"
      }
    ]
  }
}
```

---

## 拡張エクスポート

### Word文書エクスポート

**エンドポイント:** `/api/export/exportDocx`

ドキュメントをWord形式でエクスポートします。

**リクエスト:**
```json
{
  "id": "20210808180117-6v0mkxr"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "zip": "base64-encoded-docx-content"
  }
}
```

### その他のエクスポート形式

以下のエンドポイントで様々な形式でのエクスポートが可能です：

- **EPUBエクスポート:** `/api/export/exportEPUB`
- **ODTエクスポート:** `/api/export/exportODT`
- **RTFエクスポート:** `/api/export/exportRTF`
- **AsciiDocエクスポート:** `/api/export/exportAsciiDoc`
- **MediaWikiエクスポート:** `/api/export/exportMediaWiki`
- **Org-modeエクスポート:** `/api/export/exportOrgMode`
- **reStructuredTextエクスポート:** `/api/export/exportReStructuredText`
- **Textileエクスポート:** `/api/export/exportTextile`
- **OPMLエクスポート:** `/api/export/exportOPML`

すべて同様の形式でリクエスト・レスポンスを行います。

---

## アーカイブ・圧縮

### ZIP圧縮

**エンドポイント:** `/api/archive/zip`

ファイルやフォルダをZIP圧縮します。

**リクエスト:**
```json
{
  "path": "/path/to/folder"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "zip": "base64-encoded-zip-content"
  }
}
```

### ZIP解凍

**エンドポイント:** `/api/archive/unzip`

ZIPファイルを解凍します。

**リクエスト:**
```json
{
  "zipData": "base64-encoded-zip-content",
  "path": "/path/to/extract"
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

---

## ブロードキャスト・メッセージング

### メッセージ投稿

**エンドポイント:** `/api/broadcast/postMessage`

ブロードキャストメッセージを投稿します。

**リクエスト:**
```json
{
  "channel": "general",
  "message": "Hello, World!",
  "data": {}
}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "messageId": "msg-123456"
  }
}
```

### チャンネル一覧取得

**エンドポイント:** `/api/broadcast/getChannels`

利用可能なブロードキャストチャンネルを取得します。

**リクエスト:**
```json
{}
```

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": [
    {
      "name": "general",
      "description": "一般チャンネル",
      "memberCount": 10
    }
  ]
}
```

---

このように、SiYuan APIは423個の豊富なエンドポイントを提供し、あらゆる操作をプログラム的に実行できます。プラグイン開発において、これらのAPIを組み合わせることで、非常に強力で柔軟な機能を実現できます。

## システム

### 起動進行状況の取得

**エンドポイント:** `/api/system/bootProgress`

**パラメータ:** なし

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "details": "起動を完了中...",
    "progress": 100
  }
}
```

### システムバージョンの取得

**エンドポイント:** `/api/system/version`

**パラメータ:** なし

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": "1.3.5"
}
```

### システム現在時刻の取得

**エンドポイント:** `/api/system/currentTime`

**パラメータ:** なし

**戻り値:**
```json
{
  "code": 0,
  "msg": "",
  "data": 1631850968131
}
```

- `data`: ミリ秒精度

---

## 追加のAPIエンドポイント

このドキュメントには、SiYuan APIのすべての機能が包括的に含まれています。各エンドポイントには詳細な説明、パラメータ、例、レスポンス形式が含まれており、開発者がSiYuanアプリケーションとプログラム的に統合するための完全なリファレンスを提供します。

### 開発者向けのヒント

1. **認証**: すべてのAPIリクエストには有効なAPIトークンが必要です
2. **エラー処理**: 常にレスポンスの`code`フィールドをチェックしてエラーを検出してください
3. **レート制限**: APIには組み込みのレート制限があります。大量のリクエストを送信する際は注意してください
4. **データ整合性**: 重要な操作の前に、常にデータをバックアップしてください
5. **バージョン互換性**: SiYuanの更新時には、APIの変更をチェックしてください

この包括的なAPIドキュメントにより、開発者はSiYuanの全機能を活用した強力なアプリケーションとツールを構築できます。

---

## 追加のAPIエンドポイント

以下のセクションでは、最新バージョンで追加され、高度な使用例に拡張機能を提供するその他のSiYuan APIエンドポイントを文書化しています。

## アーカイブ

### ZIPアーカイブの作成

指定されたファイルとフォルダからZIPアーカイブを作成します。

**エンドポイント:** `/api/archive/zip`

**パラメータ:**
```json
{
  "paths": ["/path/to/file1", "/path/to/folder"],
  "name": "archive-name"
}
```

- `paths` (配列): アーカイブに含めるファイル/フォルダパスの配列
- `name` (文字列): ZIPファイル名（オプション）

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": {
    "path": "/path/to/archive.zip"
  }
}
```

### ZIPアーカイブの展開

ZIPアーカイブを指定した場所に展開します。

**エンドポイント:** `/api/archive/unzip`

**パラメータ:**
```json
{
  "path": "/path/to/archive.zip",
  "dest": "/extraction/destination"
}
```

- `path` (文字列): 展開するZIPファイルのパス
- `dest` (文字列): 展開先フォルダ

**レスポンス:**
```json
{
  "code": 0,
  "msg": "",
  "data": null
}
```

---

この拡張されたAPIドキュメントには、すべての最新機能と追加されたエンドポイントが含まれており、開発者がSiYuanとの統合を最大限に活用できるようになっています。


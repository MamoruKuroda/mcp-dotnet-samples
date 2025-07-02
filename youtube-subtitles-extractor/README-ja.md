# MCP サーバー: YouTube 字幕抽出ツール

指定されたYouTubeリンクから字幕を抽出する2つのMCPサーバーがあります。両方とも全く同じ機能を提供しますが、一方は公式の[C# SDK](https://www.nuget.org/packages/ModelContextProtocol)を使用した[Azure Container Apps](https://learn.microsoft.com/azure/container-apps/overview)上で動作し、もう一方は[Azure Functions SDK](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker.Extensions.Mcp)を使用した[Azure Functions](https://learn.microsoft.com/azure/azure-functions/functions-overview)上で動作します。

## 概要

このサンプルでは、YouTube動画のURLから字幕を抽出し、要約や分析を行うMCPサーバーを構築・実行する方法を説明します。

### 提供するツール

- `get_available_languages`: 指定されたYouTube動画で利用可能な字幕言語の一覧を取得
- `get_subtitle`: 指定された言語でYouTube動画の字幕を取得

### サポートする実行環境

- **Azure Container Apps版**: 高いスケーラビリティと本格的なコンテナ実行環境
- **Azure Functions版**: サーバーレス実行による効率的なリソース利用

> **注意**: このサンプルアプリで使用している[Aliencube.YouTubeSubtitlesExtractor](https://www.nuget.org/packages/Aliencube.YouTubeSubtitlesExtractor)の制限により、Azureにデプロイされたリモートサーバーはサポートされていません。ローカル実行でのテストのみ可能です。

## 🛠️ はじめに

| サンプル名 | 説明 |
|-------------|-------------|
| [Azure Container Apps での YouTube 字幕抽出](./containerapp/) | [Azure Container Apps](https://learn.microsoft.com/azure/container-apps/overview)上でホストされるMCPサーバーで、指定されたYouTube動画のURLから字幕を抽出します。 |
| [Azure Functions での YouTube 字幕抽出](./functionapp/) | [Azure Functions](https://learn.microsoft.com/azure/azure-functions/functions-overview)上でホストされるMCPサーバーで、指定されたYouTube動画のURLから字幕を抽出します。 |

## 📚 ドキュメント利用ガイド

### このREADMEを読む場合
- 🚀 **プロジェクト概要を素早く理解したい**
- 📖 **どちらのサンプル（Container Apps vs Functions）を選ぶか決めたい**
- 🔍 **YouTube字幕抽出の機能概要を知りたい**

### [詳細セットアップガイド](./MCP-YouTube-Subtitles-Extractor-Setup-Guide.md)を読む場合
- 🐛 **問題が発生してトラブルシューティングが必要**
- 🧪 **実際のテスト結果例を見たい**
- 🔧 **各環境の独立した詳細手順が欲しい**
- 🔍 **MCP Inspector の詳細な使用方法を知りたい**
- 📋 **日本語での具体的な操作例を確認したい**
- ⚙️ **Container Apps版とFunctions版の詳細な違いを理解したい**

> 💡 **ヒント**: まずはこのREADMEで概要を理解し、実際のセットアップや詳細な手順が必要な場合は[詳細セットアップガイド](./MCP-YouTube-Subtitles-Extractor-Setup-Guide.md)をご参照ください。

## 🎯 使用例

### 基本的な使用パターン

1. **YouTube動画の要約**
   ```
   このYouTube動画を5つの要点でまとめてください: https://youtu.be/XwnEtZxaokg
   ```

2. **字幕の言語確認**
   ```
   この動画で利用可能な字幕言語を教えてください: https://youtu.be/XwnEtZxaokg
   ```

3. **特定言語での字幕取得**
   ```
   この動画の英語字幕を取得してください: https://youtu.be/XwnEtZxaokg
   ```

### 想定される用途

- **教育コンテンツの分析**: 講義動画の要点抽出
- **多言語コンテンツの処理**: 国際的な動画の字幕取得
- **アクセシビリティ向上**: 字幕情報の活用
- **コンテンツ要約**: 長時間動画の概要作成

## 技術仕様

### Azure Container Apps版
- **SDK**: [ModelContextProtocol C# SDK](https://www.nuget.org/packages/ModelContextProtocol)
- **実行環境**: コンテナベース
- **スケーリング**: 自動スケーリング対応
- **適用場面**: 本格的なプロダクション環境

### Azure Functions版  
- **SDK**: [Azure Functions MCP SDK](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker.Extensions.Mcp)
- **実行環境**: サーバーレス
- **課金**: 使用量ベース
- **適用場面**: 小規模〜中規模の用途、コスト効率重視

## 前提条件

### 共通要件
- [.NET 9.0 SDK](https://dotnet.microsoft.com/download/dotnet/9.0)
- [Visual Studio Code](https://code.visualstudio.com/)
- [C# Dev Kit 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit)

### Container Apps版追加要件
- [Docker Desktop](https://docs.docker.com/get-started/get-docker/)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)

### Functions版追加要件
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local?pivots=programming-language-csharp#install-the-azure-functions-core-tools)
- [Azure Functions 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)

## 制限事項

### YouTube字幕抽出の制限
- **リモート実行不可**: Azure上にデプロイしたサーバーでは動作しません
- **ローカル実行のみ**: 開発環境での動作確認・テスト用途に限定
- **ライブラリ依存**: [Aliencube.YouTubeSubtitlesExtractor](https://www.nuget.org/packages/Aliencube.YouTubeSubtitlesExtractor)の制限によるもの

### 対応動画形式
- **YouTube動画**: 公開動画のみ対応
- **字幕あり動画**: 自動生成またはアップロード者が追加した字幕
- **地域制限**: 地域制限のある動画は取得できない場合があります

## 関連リンク

- [Model Context Protocol 仕様](https://spec.modelcontextprotocol.io/)
- [Azure Container Apps ドキュメント](https://learn.microsoft.com/azure/container-apps/)
- [Azure Functions ドキュメント](https://learn.microsoft.com/azure/azure-functions/)
- [.NET 9.0 ドキュメント](https://learn.microsoft.com/dotnet/core/whats-new/dotnet-9/overview)

## ライセンス

このプロジェクトは MIT ライセンスの下で提供されています。詳細は [LICENSE](../LICENSE) ファイルをご確認ください。

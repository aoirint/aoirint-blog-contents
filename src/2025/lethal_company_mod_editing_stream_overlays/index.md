---
title: 'Lethal Company Mod Zehs-StreamOverlaysを例としたMod改変手順'
date: '2025-08-21T22:30:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Game
tags:
  - Game
  - Lethal Company
  - Modding
---
# Lethal Company Mod Zehs-StreamOverlaysを例としたMod改変手順

Zehs-StreamOverlaysの手元での改変を試したので、他のMod改変で共通して使える知見として、メモしておきます。

今回は公開を目的としたものではなく、知見も持っていないので、Thunderstoreへの公開は内容に含まれていません。

Zehs-StreamOverlaysのソースコードは、MIT Licenseで配布されており、改変箇所を示すため、一部引用します。

- [GitHub](https://github.com/ZehsTeam/Lethal-Company-StreamOverlays)
- [Thunderstore](https://thunderstore.io/c/lethal-company/p/Zehs/StreamOverlays/)

## バージョン情報

以下のバージョンを想定しています。

- Lethal Company v72 (Build ID: 18916695)
- Windows 11 24H2
- .NET SDK 9.0.201
- BepInEx v5.4.21
- [ZehsTeam/Lethal-Company-StreamOverlays@8ba3632](https://github.com/ZehsTeam/Lethal-Company-StreamOverlays/commit/8ba3632a2ba7b8f5a4e9564d0c737ea80f7e2847)

## BeplnEx プラグイン開発環境の設定

BeplnEx公式チュートリアルの開発環境の設定手順に従って、プラグイン開発環境を構築します。

- [Basic plugin: Setting up the development environment | BepInEx Docs](https://docs.bepinex.dev/articles/dev_guide/plugin_tutorial/1_setup.html)
  - [GitHub: 記事執筆時点のソース@ca4a997](https://github.com/BepInEx/bepinex-docs/blob/ca4a9972db33ba98034ba461aa820b110390241f/articles/dev_guide/plugin_tutorial/1_setup.md)

## 依存関係のセットアップ

- [Thunderstore: LethalConfig](https://thunderstore.io/c/lethal-company/p/AinaVT/LethalConfig/) v1.4.6
- [Thunderstore: ShipInventoryUpdated](https://thunderstore.io/c/lethal-company/p/LethalCompanyModding/ShipInventoryUpdated/) v1.2.13
- [Thunderstore: CSync](https://thunderstore.io/c/lethal-company/p/Sigurd/CSync/) v5.0.1

Thunderstoreの「Manual Download」からzipファイルをダウンロードして、中のDLLファイルを取り出します。

そのままビルドに使うと、以下のようなエラーが発生します。

```plain
$ DOTNET_CLI_UI_LANGUAGE=en dotnet build --configuration Release       
Restore complete (0.5s)
  StreamOverlays failed with 4 error(s) (0.3s)
    D:\workspaces\lethal_company_modding_workspace\Lethal-Company-StreamOverlays\StreamOverlays\Dependencies\ShipInventoryProxy\Patches\ChuteInteractPatch.cs(10,40): error CS0117: 'ChuteInteract' does not contain a definition for 'SpawnItemClientRpc'
    D:\workspaces\lethal_company_modding_workspace\Lethal-Company-StreamOverlays\StreamOverlays\Dependencies\ShipInventoryProxy\Patches\ItemManagerPatch.cs(10,38): error CS0117: 'ItemManager' does not contain a definition for 'UpdateCache'
    D:\workspaces\lethal_company_modding_workspace\Lethal-Company-StreamOverlays\StreamOverlays\Dependencies\ShipInventoryProxy\ShipInventoryProxy.cs(12,39): error CS0122: 'LCMPluginInfo' is inaccessible due to its protection level
    D:\workspaces\lethal_company_modding_workspace\Lethal-Company-StreamOverlays\StreamOverlays\Plugin.cs(16,18): error CS0182: An attribute argument must be a constant expression, typeof expression or array creation expression of an attribute parameter type

Build failed with 4 error(s) in 1.1s
```

`BepInEx.AssemblyPublicizer`を使って、エラーの原因のアクセス制御を無効化したDLLを作成します。これらのDLLはビルド時のみ使用します。

- [GitHub: BepInEx/BepInEx.AssemblyPublicizer](https://github.com/BepInEx/BepInEx.AssemblyPublicizer)

```shell
dotnet tool install -g BepInEx.AssemblyPublicizer.Cli

assembly-publicizer LethalConfig.dll
assembly-publicizer ShipInventoryUpdated.dll
assembly-publicizer com.sigurd.csync.dll
```

`{stem}-publicized.dll`のように出力されたPublicize済みのDLLを`D:\workspaces\lethal_company_modding_workspace\Libraries`以下に配置します。

パスを変更する場合は、`StreamOverlays.csproj`の`LibraryFolder`を変更してください。

## `StreamOverlays.csproj`の編集

ライブラリのディレクトリを変更して、ビルド後のコピー処理は不要なので削除します。
`SteamLibrary`などはビルド後のコピー処理で使われるものなので、変更不要です。

```diff
--- a/StreamOverlays/StreamOverlays.csproj
+++ b/StreamOverlays/StreamOverlays.csproj
@@ -42,7 +42,7 @@

   <PropertyGroup>
     <!-- Mod libraries folder -->
-    <LibraryFolder>D:\Documents\Lethal Company Modding\Mods</LibraryFolder>
+    <LibraryFolder>D:\workspaces\lethal_company_modding_workspace\Libraries</LibraryFolder>

     <!-- Steam library folder -->
     <SteamLibrary>D:\SteamLibrary\steamapps\common</SteamLibrary>
@@ -84,25 +84,4 @@
     <Reference Include="ShipInventoryUpdated"><HintPath>$(LibraryFolder)\LethalCompanyModding-ShipInventoryUpdated\ShipInventoryUpdated.dll</HintPath></Reference>
     <Reference Include="CSync">               <HintPath>$(LibraryFolder)\Sigurd-CSync\com.sigurd.csync.dll                                 </HintPath></Reference>
   </ItemGroup>
-
-  <Target Name="CopyToPluginsFolder" AfterTargets="PostBuildEvent">
-    <Copy DestinationFolder="$(PluginsFolder)" OverwriteReadOnlyFiles="true" SkipUnchangedFiles="true" SourceFiles="$(TargetPath)" />
-  </Target>
-
-  <Target Name="CopyToGalePluginsFolder" AfterTargets="CopyToPluginsFolder">
-    <Copy DestinationFolder="$(GalePluginsFolder)" OverwriteReadOnlyFiles="true" SkipUnchangedFiles="true" SourceFiles="$(TargetPath)" />
-  </Target>
-
-  <Target Name="CopyPublicFolder" AfterTargets="CopyToGalePluginsFolder">
-    <Exec Command="robocopy &quot;$(WebsiteFolder)&quot; &quot;$(GalePluginsFolder)\public&quot; /mir || exit 0" />
-  </Target>
-
-  <Target Name="CopyToOlderVersionsFolders" AfterTargets="CopyToPluginsFolder">
-    <Copy DestinationFolder="$(PluginsFolderV40)" OverwriteReadOnlyFiles="true" SkipUnchangedFiles="true" SourceFiles="$(TargetPath)" />
-    <Copy DestinationFolder="$(PluginsFolderV45)" OverwriteReadOnlyFiles="true" SkipUnchangedFiles="true" SourceFiles="$(TargetPath)" />
-    <Copy DestinationFolder="$(PluginsFolderV49)" OverwriteReadOnlyFiles="true" SkipUnchangedFiles="true" SourceFiles="$(TargetPath)" />
-    <Copy DestinationFolder="$(PluginsFolderV50)" OverwriteReadOnlyFiles="true" SkipUnchangedFiles="true" SourceFiles="$(TargetPath)" />
-    <Copy DestinationFolder="$(PluginsFolderV56)" OverwriteReadOnlyFiles="true" SkipUnchangedFiles="true" SourceFiles="$(TargetPath)" />
-    <Copy DestinationFolder="$(PluginsFolderV62)" OverwriteReadOnlyFiles="true" SkipUnchangedFiles="true" SourceFiles="$(TargetPath)" />
-  </Target>
 </Project>
```

## ビルド

コードを変更したら、ビルドします。

```shell
DOTNET_CLI_UI_LANGUAGE=en dotnet build --configuration Release
```

`bin/Release/netstandard2.1/com.github.zehsteam.StreamOverlays.dll`にDLLが出力されます。

## 使用

r2modmanまたはThunderstore Appを開きます。

r2modmanの場合、`Settings > Profile > Import local mod`から、ビルドしたDLLを選択してインポートします。

---
name: building-native-ui
description: Complete guide for building beautiful apps with Expo Router. Covers fundamentals, styling, components, navigation, animations, patterns, and native tabs.
version: 1.0.1
license: MIT
---

# Expo UI ガイドライン

## リファレンス

必要に応じて以下のリソースを参照してください：

```
references/
  animations.md          Reanimated: 入場・退場・レイアウト・スクロール連動・ジェスチャーアニメーション
  controls.md            ネイティブiOS: Switch, Slider, SegmentedControl, DateTimePicker, Picker
  form-sheet.md          expo-routerのフォームシート: 設定、フッター、背景インタラクション
  gradients.md           experimental_backgroundImageによるCSSグラデーション（New Archのみ）
  icons.md               expo-imageのSF Symbols（sf:ソース）、名前、アニメーション、ウェイト
  media.md               カメラ、オーディオ、ビデオ、ファイル保存
  route-structure.md     ルート規約、動的ルート、グループ、フォルダ構成
  search.md              ヘッダー付き検索バー、useSearchフック、フィルタリングパターン
  storage.md             SQLite, AsyncStorage, SecureStore
  tabs.md                NativeTabs、JSタブからの移行、iOS 26の機能
  toolbar-and-headers.md Stackヘッダーとツールバーボタン、メニュー、検索（iOSのみ）
  visual-effects.md      ブラー（expo-blur）とリキッドグラス（expo-glass-effect）
  webgpu-three.md        WebGPUとThree.jsによる3Dグラフィックス、ゲーム、GPU可視化
  zoom-transitions.md    Apple Zoom: Link.AppleZoomによる流体的ズームトランジション（iOS 18+）
```

## アプリの実行

**重要: カスタムビルドの前に、必ずExpo Goを最初に試してください。**

ほとんどのExpoアプリは、カスタムネイティブコードなしでExpo Goで動作します。`npx expo run:ios` や `npx expo run:android` を実行する前に：

1. **まずExpo Goで試す**: `npx expo start` を実行し、Expo GoでQRコードをスキャン
2. **機能が動作するか確認**: Expo Goでアプリを十分にテスト
3. **必要な場合のみカスタムビルドを作成** - 以下を参照

### カスタムビルドが必要なケース

`npx expo run:ios/android` や `eas build` が必要になるのは以下の場合のみです：

- **ローカルExpoモジュール**（`modules/` 内のカスタムネイティブコード）
- **Appleターゲット**（ウィジェット、App Clip、`@bacons/apple-targets` によるエクステンション）
- **Expo Goに含まれていないサードパーティネイティブモジュール**
- **`app.json` で表現できないカスタムネイティブ設定**

### Expo Goで動作するケース

Expo Goは標準で非常に多くの機能をサポートしています：

- すべての `expo-*` パッケージ（camera, location, notificationsなど）
- Expo Routerナビゲーション
- ほとんどのUIライブラリ（reanimated, gesture handlerなど）
- プッシュ通知、ディープリンクなど

**迷ったら、まずExpo Goを試してください。** カスタムビルドは複雑さが増し、イテレーションが遅くなり、Xcode/Android Studioのセットアップが必要になります。

## コードスタイル

- 未終端の文字列に注意してください。ネストされたバッククォートは必ずエスケープし、引用符のエスケープを忘れないでください。
- import文は必ずファイルの先頭に配置してください。
- ファイル名は必ずケバブケースを使用してください（例: `comment-card.tsx`）
- ルートの移動や構造変更時は、古いルートファイルを必ず削除してください
- ファイル名に特殊文字を使用しないでください
- tsconfig.jsonでパスエイリアスを設定し、リファクタリング時は相対インポートよりエイリアスを優先してください。

## ルート

詳細なルート規約は `./references/route-structure.md` を参照してください。

- ルートは `app` ディレクトリに配置します。
- コンポーネント、型、ユーティリティをappディレクトリ内に同居させないでください。これはアンチパターンです。
- アプリには必ず "/" にマッチするルートを含めてください。グループルート内でも構いません。

## ライブラリの優先順位

- React Nativeから削除されたモジュール（Picker, WebView, SafeAreaView, AsyncStorage）は使用しないでください
- レガシーの expo-permissions は使用しないでください
- `expo-av` ではなく `expo-audio` を使用
- `expo-av` ではなく `expo-video` を使用
- SF Symbolsには `expo-symbols` や `@expo/vector-icons` ではなく、`expo-image` の `source="sf:name"` を使用
- react-nativeの SafeAreaView ではなく `react-native-safe-area-context` を使用
- `Platform.OS` ではなく `process.env.EXPO_OS` を使用
- `React.useContext` ではなく `React.use` を使用
- 組み込みの `img` 要素ではなく `expo-image` の Image コンポーネントを使用
- リキッドグラス背景には `expo-glass-effect` を使用

## レスポンシブ対応

- ルートコンポーネントは必ずスクロールビューでラップしてレスポンシブに対応
- `<SafeAreaView>` の代わりに `<ScrollView contentInsetAdjustmentBehavior="automatic" />` を使用し、よりスマートなセーフエリアインセットを実現
- `contentInsetAdjustmentBehavior="automatic"` は FlatList や SectionList にも適用してください
- Dimensions API の代わりにflexboxを使用
- 画面サイズの取得には `Dimensions.get()` ではなく、**必ず** `useWindowDimensions` を使用

## 動作

- expo-hapticsをiOSで条件付きで使用し、より心地よい体験を実現
- React Nativeの `<Switch />` や `@react-native-community/datetimepicker` などの組み込みハプティクス付きビューを使用
- ルートがStackに属する場合、最初の子要素はほぼ必ず `contentInsetAdjustmentBehavior="automatic"` を設定したScrollViewにすべき
- ページに `ScrollView` を追加する場合、ほぼ必ずルートコンポーネント内の最初のコンポーネントにすべき
- 検索バーの追加にはStack.Screenオプションの `headerSearchBarOptions` を優先
- コピー可能なデータを含むテキストには `<Text selectable />` propを使用
- 大きな数値は1.4Mや38kのようにフォーマットすることを検討
- webviewまたはExpo DOMコンポーネント内でない限り、'img'や'div'などの組み込み要素を使用しないでください

# スタイリング

Apple Human Interface Guidelinesに従ってください。

## 一般的なスタイリングルール

- marginやpaddingスタイルよりflex gapを優先
- 可能な限りmarginよりpaddingを優先
- セーフエリアは必ず考慮してください。Stackヘッダー、タブ、またはScrollView/FlatListの `contentInsetAdjustmentBehavior="automatic"` を使用
- 上部と下部の両方のセーフエリアインセットを必ず考慮してください
- スタイルの再利用が効率的でない限り、StyleSheet.createではなくインラインスタイルを使用
- 状態変化にはenteringとexitingアニメーションを追加
- カプセル形状でない限り、角丸には `{ borderCurve: 'continuous' }` を使用
- ページ上のカスタムテキスト要素の代わりに、**必ず**ナビゲーションスタックのタイトルを使用
- ScrollViewのパディングには、ScrollView自体のpaddingではなく `contentContainerStyle` のpaddingとgapを使用（クリッピングを軽減）
- CSSとTailwindはサポートされていません - インラインスタイルを使用してください

## テキストスタイリング

- 重要なデータやエラーメッセージを表示するすべての `<Text/>` 要素に `selectable` propを追加
- カウンターには整列のため `{ fontVariant: 'tabular-nums' }` を使用

## シャドウ

CSSの `boxShadow` スタイルpropを使用してください。レガシーのReact Nativeシャドウやelevationスタイルは**絶対に使用しないでください**。

```tsx
<View style={{ boxShadow: "0 1px 2px rgba(0, 0, 0, 0.05)" }} />
```

'inset'シャドウもサポートされています。

# ナビゲーション

## Link

ルート間のナビゲーションには 'expo-router' の `<Link href="/path" />` を使用してください。

```tsx
import { Link } from 'expo-router';

// 基本的なリンク
<Link href="/path" />

// カスタムコンポーネントのラッピング
<Link href="/path" asChild>
  <Pressable>...</Pressable>
</Link>
```

可能な限り `<Link.Preview>` を含めてiOSの慣例に従ってください。ナビゲーションを強化するため、コンテキストメニューとプレビューを頻繁に追加してください。

## Stack

- `_layout.tsx` ファイルで**必ず**スタックを定義してください
- ネイティブナビゲーションスタックには 'expo-router/stack' の Stack を使用

### ページタイトル

Stack.Screenオプションでページタイトルを設定：

```tsx
<Stack.Screen options={{ title: "Home" }} />
```

## コンテキストメニュー

Linkコンポーネントに長押しコンテキストメニューを追加：

```tsx
import { Link } from "expo-router";

<Link href="/settings" asChild>
  <Link.Trigger>
    <Pressable>
      <Card />
    </Pressable>
  </Link.Trigger>
  <Link.Menu>
    <Link.MenuAction
      title="Share"
      icon="square.and.arrow.up"
      onPress={handleSharePress}
    />
    <Link.MenuAction
      title="Block"
      icon="nosign"
      destructive
      onPress={handleBlockPress}
    />
    <Link.Menu title="More" icon="ellipsis">
      <Link.MenuAction title="Copy" icon="doc.on.doc" onPress={() => {}} />
      <Link.MenuAction
        title="Delete"
        icon="trash"
        destructive
        onPress={() => {}}
      />
    </Link.Menu>
  </Link.Menu>
</Link>;
```

## Linkプレビュー

ナビゲーションを強化するためリンクプレビューを頻繁に使用してください：

```tsx
<Link href="/settings">
  <Link.Trigger>
    <Pressable>
      <Card />
    </Pressable>
  </Link.Trigger>
  <Link.Preview />
</Link>
```

リンクプレビューはコンテキストメニューと併用できます。

## モーダル

画面をモーダルとして表示：

```tsx
<Stack.Screen name="modal" options={{ presentation: "modal" }} />
```

カスタムモーダルコンポーネントを構築するよりもこちらを優先してください。

## シート

画面を動的フォームシートとして表示：

```tsx
<Stack.Screen
  name="sheet"
  options={{
    presentation: "formSheet",
    sheetGrabberVisible: true,
    sheetAllowedDetents: [0.5, 1.0],
    contentStyle: { backgroundColor: "transparent" },
  }}
/>
```

- `contentStyle: { backgroundColor: "transparent" }` を使用すると、iOS 26以降で背景がリキッドグラスになります。

## 一般的なルート構造

タブとスタックを組み合わせた標準的なアプリレイアウト：

```
app/
  _layout.tsx — <NativeTabs />
  (index,search)/
    _layout.tsx — <Stack />
    index.tsx — メインリスト
    search.tsx — 検索ビュー
```

```tsx
// app/_layout.tsx
import { NativeTabs, Icon, Label } from "expo-router/unstable-native-tabs";
import { Theme } from "../components/theme";

export default function Layout() {
  return (
    <Theme>
      <NativeTabs>
        <NativeTabs.Trigger name="(index)">
          <Icon sf="list.dash" />
          <Label>Items</Label>
        </NativeTabs.Trigger>
        <NativeTabs.Trigger name="(search)" role="search" />
      </NativeTabs>
    </Theme>
  );
}
```

共有グループルートを作成して、両方のタブから共通画面にプッシュできるようにします：

```tsx
// app/(index,search)/_layout.tsx
import { Stack } from "expo-router/stack";
import { PlatformColor } from "react-native";

export default function Layout({ segment }) {
  const screen = segment.match(/\((.*)\)/)?.[1]!;
  const titles: Record<string, string> = { index: "Items", search: "Search" };

  return (
    <Stack
      screenOptions={{
        headerTransparent: true,
        headerShadowVisible: false,
        headerLargeTitleShadowVisible: false,
        headerLargeStyle: { backgroundColor: "transparent" },
        headerTitleStyle: { color: PlatformColor("label") },
        headerLargeTitle: true,
        headerBlurEffect: "none",
        headerBackButtonDisplayMode: "minimal",
      }}
    >
      <Stack.Screen name={screen} options={{ title: titles[screen] }} />
      <Stack.Screen name="i/[id]" options={{ headerLargeTitle: false }} />
    </Stack>
  );
}
```

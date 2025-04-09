---
title: Jetpack ComposeのModifier適用順序を理解する
description: Jetpack ComposeのModifierは適用順序によってUIの見た目や挙動が大きく変わります。本記事では、Modifierの順序がUIにどのような影響を与えるかを図や具体例を交えて分かりやすく解説します。
duration: 10min
date: 2025-4-8
lang: ja-JP
draft: true
tag: Jetpack Compose,Modifier,Kotlin
image:
  src: "/ogp/modifier-order.png"
  alt: "Jetpack ComposeにおけるModifierの適用順序の違いを示す図解"
---

## はじめに
Jetpack ComposeはAndroidのUIを簡潔に構築できるフレームワークであり、その中でも`Modifier`は、レイアウトの調整や装飾、インタラクションの追加など幅広い用途で活用される重要な機能です。

しかし、`Modifier`の適用順序を誤ると、意図しないレイアウト崩れや想定外の動作を引き起こす可能性があります。本記事では、`Modifier`の適用順序がUIにどのような影響を与えるのかを図や具体例を交えながら、なるべく初学者の方にも理解しやすい形で解説していきます。

なお、`Modifier`とは何かや基本的な使い方については公式ドキュメントが詳しく解説しているので、以下のリンクを参考にしてください。
https://developer.android.com/develop/ui/compose/modifiers?hl=ja

# Modifierは適用順序が重要
冒頭でも述べたように、`Modifier`の適用順序を誤ると意図しないレイアウト崩れや想定外の動作を引き起こす可能性があります。
`Modifier`は上から順に適用されるため、どの順番で設定するかによって見た目や動作が大きく変わります。

ここでは、Textコンポーネントに背景色（`background`）とパディング（`padding`）を適用するシンプルな例を見ていきます。

### backgroundを先に適用した場合

```kotlin
    Text(
        text = "Hello World!", modifier = modifier
            .background(Color.Red)
            .padding(16.dp)
    )
```

<img src='https://storage.googleapis.com/zenn-user-upload/47feb4f09602-20250316.png'>

上記の場合、まずは`background`が評価され、Textのテキストサイズに応じて色がつきます。その後`padding`が評価され、Textに対して`padding`が追加されることによって全体のサイズが拡張される（背景色に影響することによって大きく見える）。

次に`background`と`padding`の順序を逆にしてみます。

### paddingを先に適用した場合

```kotlin
    Text(
        text = "Hello World!", modifier = modifier
            .padding(16.dp)
            .background(Color.Red)
    )
```

<img src='https://storage.googleapis.com/zenn-user-upload/1ae2865b0ec7-20250316.png'>

すると、先ほどとは異なり、先に`padding`が評価され、背景色のサイズ自体は変わりません（`padding`が背景色に影響しないので大きさが変わらない）。
このように`Modifier`は、前後を入れ替えるだけで簡単にUIが変わってしまうため、複雑なレイアウトを組む際には適切な知識を組んでいく必要があります。

## なぜ順番が影響を与えるのか？

その理由は、Jetpack Compose の UI は「UIツリー」という構造で管理されており、Modifier はこのツリー内で「ラップ構造（Wrapper）」として適用される ためです。

では、この UIツリーがどのように構築され、Modifier の適用順がどのように影響を与えるのかを具体的に見ていきましょう。

✅ UIツリーとは？

Jetpack Compose の UI は、以下の 3つのフェーズ で構築されます。

1️⃣ コンポーズフェーズ（Composition Phase）d
•Composable 関数が評価され、UIツリーが生成される
•ここで、Text() や Box() などの 組み込みコンポーザブル（Built-in Composable） が UIツリー内に追加される

2️⃣ レイアウトフェーズ（Layout Phase）
•親コンポーネントが子コンポーネントに「どのくらいのサイズで描画するか」を決める
•fillMaxSize(), size(100.dp), padding(16.dp) などの レイアウト関連の Modifier は、このフェーズで影響を受ける

3️⃣ 描画フェーズ（Draw Phase）
•UIツリーの情報を元に、実際に画面に描画する
•background(Color.Blue), clip(RoundedCornerShape(16.dp)), border(2.dp, Color.Black) などの 描画関連の Modifier は、このフェーズで適用される

[UI ツリー内の修飾子](https://developer.android.com/develop/ui/compose/layouts/constraints-modifiers?hl=ja#modifiers-ui)に説明があるように、「`Modifier`はUIツリー内でノードをラップ」します。簡単に説明すると、`Modifier`を適用しているComposableを包み込むように適用されるということです。

## Modifierを組む上でどのように意識したら良いか？
`レイアウト`→`配置`→`装飾`→`その他（クリックイベントやアニメーション）`
上記の順で`Modifier`を適用していきましょう。

## 具体例

Modifierの組み合わせ適用順を間違えたときの影響
background() + padding()背景色の適用範囲が意図しないものになる
size() + clip()size() を後に適用すると clip() の影響を受けない
fillMaxSize() + padding()fillMaxSize() を後に適用すると padding() が無視される
clickable() + background()clickable() を先に適用すると、タップ範囲が狭くなる

### 5つのカテゴリごとに解説

## Modifierの適用順序

| 順番 | カテゴリ         | 例                                      |
|------|--------------|--------------------------------------|
| 1️⃣  | **レイアウト系** | `size()`, `fillMaxSize()`, `weight()`  |
| 2️⃣  | **配置系**     | `padding()`, `align()`, `offset()`   |
| 3️⃣  | **描画系**     | `background()`, `border()`, `clip()` |
| 4️⃣  | **動作系**     | `clickable()`, `pointerInput()`      |
| 5️⃣  | **アニメーション系** | `alpha()`, `graphicsLayer()`         |

## カテゴリ別の実装サンプル

## まとめ

## 参考文献
[制約と修飾子の順序](https://developer.android.com/develop/ui/compose/layouts/constraints-modifiers?hl=ja)
[JetpackComposeのModifierの順序について](https://zenn.dev/mitohato/articles/b9840ea54d66cd)

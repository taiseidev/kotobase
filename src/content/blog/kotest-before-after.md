---
title: kotestのbefore,afterについてまとめてみた
description: kotestで使えるbefore/after系のライフサイクル関数をまとめて解説。使い分けに迷う方に向けて、実際のコード例とともに詳しく紹介します。
duration: 10min
date: 2025-3-29
lang: ja-JP
draft: true
tag: kotlin,kotest,自動テスト
---

## はじめに

kotestに限らずテストコードを書く際にbefore,afterといったテスト準備やクリーンアップ処理を書くことがあると思いますが、種類が多くてどれを使えばいいのか分からなかったので今回まとめてみました。
どのように活用できるかを実際のコードを交えながら解説していくので、是非kotestを使ったテストコードを書く際の参考にしていただけたら嬉しいです！

### version
kotest: v5.9.1

## 対象者
```
- Kotlinでテストを書く際にbeforeやafterの使い方を学びたい方
- Kotestを使いこなしたい初心者〜中級者向け
- テストの前後処理を効率的に行いたい方
```

## kotestのbefore,afterの種類

[Lifecycle hooks](https://kotest.io/docs/framework/lifecycle-hooks.html) <br>
上記の公式ドキュメントを確認すると、こんなに沢山のbefore,after関数が用意されていることが分かります。

| 関数名            |
|-------------------|
| beforeContainer    |
| afterContainer     |
| beforeEach         |
| afterEach          |
| beforeAny          |
| afterAny           |
| beforeTest         |
| afterTest          |
| beforeSpec         |
| afterSpec          |
| finalizeSpec       |
| beforeInvocation   |
| afterInvocation    |

> 公式ドキュメントでは***prepareSpec***というメソッドが記載されていますが、[Sanitize prepareSpec and finalizeSpec behavior ](https://github.com/kotest/kotest/pull/1617/files#diff-cead89fdee3b0f2c28eb11479c5e4e8a7124877fa30ee93b591c58e75f5e8652)でDeprecatedになり、現在は使用できないです。代わりに***beforeSpec***を使うことを推奨されています。

次の章から各々がどういった動作をするかを解説していきます。

### 前提

今回はkotestの中でもDescribeSpecを使って解説してきますが、他のkotestの書き方を使っても基本的には動作に変わりはありません。
DescribeSpecを使ったことがない方のためにDescribeSpecの簡単な使い方を解説します。

DescribeSpecは、テストケースを階層構造で整理しやすいBDDスタイルのSpecです。
テスト対象の機能や振る舞いを describe {} ブロックで表現し、その中で条件ごとの前提を context {} で区切り、個別の動作確認をit{}で記述するのが特徴です。
簡単に説明すると、***describe***にテスト対象のクラスや関数名を記入、***context***でテスト条件を指定、***it***で結果を記載し個別のテストを行なっていく形になります。

```kotlin
class SampleTest : DescribeSpec({
    describe("ユーザー登録機能") {
        context("入力が正しい場合") {
            it("登録に成功すること") {
                // アサーション
            }
        }

        context("メールアドレスが無効な場合") {
            it("登録に失敗すること") {
                // アサーション
            }
        }
    }
})
```

このように、describe → context → it の構造にすることで、テストの意図を自然な言葉で表現でき、可読性の高いテストを書くことができます。

## 1. Spec単位で1回だけ実行

|関数名|説明|
|---|---|
|beforeSpec|Specインスタンスが生成されたあと、最初に1回だけ実行される処理。DBの初期化や一時ファイル作成などに使用|
|afterSpec|Spec内の全てテストが終わった後に1回だけ実行される。リソース解放やログ出力に使用|
|finalizeSpec|afterSpecの後に呼ばれる|

```kotlin
class KotestLifecycleDemo : DescribeSpec({
    beforeSpec {
        println("[beforeSpec] Specの最初に1回だけ実行")
    }

    afterSpec {
        println("[afterSpec] Specの最後に1回だけ実行")
    }

    finalizeSpec {
        println("[finalizeSpec] Spec終了後のクリーンアップ処理")
    }

    describe("ライフサイクルテスト") {

        it("テストケース1") {
            println("→ テストケース1 実行中")
        }

        it("テストケース2") {
            println("→ テストケース2 実行中")
        }
    }
})
```

```
[beforeSpec] Specの最初に1回だけ実行
→ テストケース1 実行中
→ テストケース2 実行中
[afterSpec] Specの最後に1回だけ実行
[finalizeSpec] Spec終了後のクリーンアップ処理
```

Specとはkotestにおけるテストクラスのインスタンスのことを指します。
つまり、今回のケースだと***KotestLifecycleDemo***インスタンスが作成された際に***beforeSpec***が実行され、すべてのテストケースが終了した際に***afterSpec***が実行されます。
***finalizeSpec***はあまり使う機会はないかもしれませんが、***afterSpec***が実行された後に実行されます。

上記の用途としては、***beforeSpec***でDBのインスタンスを立ち上げて、***afterSpec***でDBの接続を切るといった使い方ができると思います。

## 2. 各テストの直前・直後に実行されるもの

|関数名|説明|
|---|---|
|beforeEach|各テスト関数の直前に毎回実行。共通の前準備に使用する|
|afterEach|各テスト関数の直後に毎回実行。テストの後処理やクリーンアップに使用する|

```kotlin
class KotestLifecycleDemo : DescribeSpec({
    beforeEach {
        println("[beforeEach] 各テスト単位の前に実行")
    }

    afterEach {
        println("[afterEach] 各テスト単位の後に実行")
    }

    describe("ライフサイクルテスト") {

        it("テストケース1") {
            println("→ テストケース1 実行中")
        }

        it("テストケース2") {
            println("→ テストケース2 実行中")
        }
    }
})
```

```
[beforeEach] 各テスト単位の前に実行
→ テストケース1 実行中
[afterEach] 各テスト単位の後に実行
[beforeEach] 各テスト単位の前に実行
→ テストケース2 実行中
[afterEach] 各テスト単位の後に実行
```

***beforeEach***と***afterEach***はシンプルで、各テストが実行される前後に実行されます。

## 3. Container（describe/contextなどのブロック）実行前後

|関数名|説明|
|---|---|
|beforeContainer|describeやcontextなどのブロックの直前に実行|
|afterContainer|ブロックのすべてのテストが終わった後に実行|

```kotlin
class KotestLifecycleDemo : DescribeSpec({
    beforeContainer {
        println("[beforeContainer] Containerの前に実行")
    }

    afterContainer {
        println("[afterContainer] Containerの後に実行")
    }

    describe("ライフサイクルテスト - describe ブロック") {

        it("テストケース1") {
            println("→ テストケース1 実行中")
        }

        it("テストケース2") {
            println("→ テストケース2 実行中")
        }

        context("ライフサイクルテスト - context ブロック") {

            it("テストケース3") {
                println("→ テストケース3 実行中")
            }

            it("テストケース4") {
                println("→ テストケース4 実行中")
            }
        }
    }

    describe("テストケース5") {
        it("テストケース5") {
            println("→ テストケース5 実行中")
        }
    }

    describe("テストケース6") {}
})
```

```
[beforeContainer] Containerの前に実行
→ テストケース1 実行中
→ テストケース2 実行中
[beforeContainer] Containerの前に実行
→ テストケース3 実行中
→ テストケース4 実行中
[afterContainer] Containerの後に実行
[afterContainer] Containerの後に実行
[beforeContainer] Containerの前に実行
→ テストケース5 実行中
[afterContainer] Containerの後に実行
```

***beforeContainer***と***afterContainer***は、各***describe***や***context***ブロックの中に実行対象となるテスト（***it***）がある場合に限り、そのブロックの実行前後に1回ずつ実行されます。<br>
注意点として、ブロック内に有効なテストが1つもない場合（すべてスキップなど）には呼ばれません。

## 4. 全てのTestType毎に呼ばれるもの

|関数名|説明|
|---|---|
|beforeAny|最初のテストやコンテナのどこよりも前に1回だけ実行。ライフサイクルの一番最初に実行|
|afterAny|最後のテストやコンテナの全てが終わった後に1回だけ実行。ログや全体のサマリ出力に便利|

```kotlin
class KotestLifecycleDemo : DescribeSpec({
    beforeAny {
        println("[beforeAny] Spec評価の最初に1回だけ実行")
    }

    afterAny {
        println("[afterAny] Specの全テストが終わった後に1回だけ実行")
    }

    describe("ライフサイクルテスト - describe ブロック") {

        it("テストケース1") {
            println("→ テストケース1 実行中")
        }

        it("テストケース2") {
            println("→ テストケース2 実行中")
        }

        context("ライフサイクルテスト - context ブロック") {

            it("テストケース3") {
                println("→ テストケース3 実行中")
            }

            it("テストケース4") {
                println("→ テストケース4 実行中")
            }
        }
    }
})
```

```
[beforeContainer] Containerの前に呼ばれます
→ テストケース1 実行中
→ テストケース2 実行中
[beforeContainer] Containerの前に呼ばれます
→ テストケース3 実行中
→ テストケース4 実行中
[afterContainer] Containerの後に呼ばれます
[afterContainer] Containerの後に呼ばれます
```

## 5. テストの各 invocation（繰り返し）に対して呼ばれるもの

|関数名|説明|
|---|---|
|beforeTest|各テストの開始前に呼ばれる。beforeEachと似ているが、内部的なフック|
|afterTest|各テストの終了後に呼ばれる。afterEachと似ているが、テスト全体の結果などにアクセスできる|
|beforeInvocation|テスト関数が複数回（リトライ、反復など）呼ばれる場合に。各実行の直前に呼ばれる。iは何回メカのインデックス。|
|afterInvocation|各invocationの後に呼ばれる。複数回実行するテスト（例えばinvocations = 5）の後処理に使える。|

```kotlin
class KotestLifecycleDemo : DescribeSpec({

    beforeContainer {
        println("[beforeContainer] Containerの前に呼ばれます")
    }

    afterContainer {
        println("[afterContainer] Containerの後に呼ばれます")
    }

    describe("ライフサイクルテスト - describe ブロック") {

        it("テストケース1") {
            println("→ テストケース1 実行中")
        }

        it("テストケース2") {
            println("→ テストケース2 実行中")
        }

        context("ライフサイクルテスト - context ブロック") {

            it("テストケース3") {
                println("→ テストケース3 実行中")
            }

            it("テストケース4") {
                println("→ テストケース4 実行中")
            }
        }
    }
})
```

```
[beforeContainer] Containerの前に呼ばれます
→ テストケース1 実行中
→ テストケース2 実行中
[beforeContainer] Containerの前に呼ばれます
→ テストケース3 実行中
→ テストケース4 実行中
[afterContainer] Containerの後に呼ばれます
[afterContainer] Containerの後に呼ばれます
```

## まとめ

```
- beforeとafterはテストの前後処理を自動化するために使い、beforeTest/afterTest、beforeEach/afterEach、beforeAll/afterAllと使い分けが可能です。
- beforeEachやafterEachを使うことで、個別のテストケースの前後で処理を行うことができ、テストが独立して実行されるようになります。
- beforeAllやafterAllは、全体的なセットアップやクリーンアップを行うために使用します。
```

テストの前後処理を適切に使いこなすことで、テストコードの可読性や保守性が向上し、効率的にテストを実行できるようになります。是非、beforeやafterを活用して、テストの品質を向上させましょう。

-----------

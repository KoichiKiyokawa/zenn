---
title: "openapi-typescriptとtype-festでtrpcみたいな補完を実現する"
emoji: "🧙‍♀️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rest-api", "openapi-typescript", "type-fest", "trpc"]
published: false
---

# 概要
TODO: ここに補完が効いている様子をGifで貼り付ける

# 使用技術
- ky (任意のfetchライブリで可)
- SWR
- [openapi-typescript] (https://github.com/drwpow/openapi-typescript)
- type-fest (便利な型定義の詰め合わせ)

# 解説
## type-festのGetについて
ネストしたプロパティを参照するときに便利。
```ts
type NestedA = {
  a: {
    b: {
      c: 1
    }
  }
}

Get<NestedA, 'a.b.c'> // => 1
```
これだけだと`NestedA['a']['b']['c']`とするのと何も変わらないのでは？と思われるかもしれません。しかし、以下の場合はどうでしょう。

```ts
// TODO: openapiから生成された型でpostがなく、一辺倒に掘り出せない場合の例
```

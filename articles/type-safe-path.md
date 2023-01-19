---
title: 【TypeScript】型パズルと関数数行で、型安全なリンクを実現する
emoji: "💪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "型安全リンク"]
published: true
---

# はじめに

```tsx
<a href="/users1/posts/2"></a>
```

お気づきでしょうか。リンクをタイポしてリンク切れを起こしてしまっています。正しくは、`users`と`1`の間にスラッシュが入ります。
リンクはそもそも人間が手打ちするには長過ぎます。

どうやってリンク切れを防止しましょう。リンク切れ自体は E2E テストなりで防げるかもしれません。それでも、もっと早いタイミングで気づけるに越したことはないです。さらに言えば、長い文字列をタイポの恐怖に怯えながら、なんの補助もなしに打つのは精神衛生上良くないです。

そこで、型安全なリンクの出番です。

# この記事の実現イメージ

この記事では、buildPath(path: `パス文字列`, params: `パスパラメータやクエリ`)のような関数を、バチバチに補完が効く状態にする手法を提案します。

## 補完が聞いている様子

![補完が聞いている様子](/images/type-safe-path/completion.gif)

## 型チェックのイメージ

```ts
// ✅
buildPath("/posts")

// ✅
buildPath("/posts/:id", { id: 1 })

// ❌ パスパラメータが指定されていません
buildPath("/posts/:id")
```

またこの記事では、react-router-dom や vue-router のようなルーティングライブラリを想定していますが、後述の方法で Next.js, Nuxt.js, SvelteKit のようなファイルベースのルーティングにも対応可能です。

# 他の手法との比較

[pathpida](https://github.com/aspida/pathpida)という型安全リンクを実現するライブラリをご存じの方も多いかもしれません。非常に便利で、Next.js などの pages/ディレクトリから以下のようなオブジェクトを生成してくれます。

```ts
export const pagesPath = {
  posts: {
    _id: (id: string | number) => ({
      $url: (url?: { hash?: string }) => ({
        pathname: "/posts/[id]" as const,
        query: { id },
        hash: url?.hash,
      }),
    }),
  },
}

export type PagesPath = typeof pagesPath
```

使い勝手は次のとおりです。

```ts
import { pagesPath } from "../lib/$path"

// `/post/[id]`
pagesPath.post._pid(1).$url()
```

react-router-dom や vue-router を使う際、pathpida が生成してくれるようなオブジェクトを自前で書くのも一つの方法です。この記事ではこれを`オブジェクト方式`と呼ぶことにします。
提案手法と`オブジェクト方式`を様々な指標で比較したのが次の表です。
| | 提案手法 | オブジェクト方式 |
| -- | -- | -- |
| 補完 | ✅ 長いパスもあいまい検索のように絞り込める | 何回も`.`を打って、1 階層ずつ掘り進んでいく必要がある
| バンドルサイズ | ✅ 短い関数だけなので、パスが膨大になっても大きくならない | ⚠️ パスが膨大になると、巨大なオブジェクトが生成される

提案手法はオブジェクト方式の辛みをある程度解決していると考えます。

# 提案手法の実装

まず、パスとそのパスにおけるクエリを以下のように定義しておきます。この型定義はパスが増えるたびに追加していきます。

```ts
type Paths = {
  "/posts": { query: { q: string } }
  "/posts/:id": { query?: never } // クエリを取らない場合はこのように書きます
  // ...
}
```

つぎに、ちょっとした型パズルを用意します

```ts
type GetParams<Path> = Path extends `${string}:${infer P}/${infer Rest}`
  ? P | GetParams<Rest>
  : Path extends `${string}:${infer P}`
  ? P
  : never
```

（難しいと感じる方は[【TypeScript】infer を理解する](https://zenn.dev/kotamaki/articles/1bef9e8ce000e3)をご参照ください。）
このパズルは以下のような結果を吐き出します。

```ts
// GetParams<"/posts"> => never
// GetParams<"/posts/:id"> => "id"
// GetParams<"/posts/:id/comments/:commentId"> => "id" | "commendId"
```

さて、メインの`buildPath`関数は以下のように実装します。

```ts
function buildPath<Path extends keyof Paths>(
  path: Path,
  ...params: GetParams<Path> extends never
    ? [params?: Paths[Path] & { hash?: string }]
    : [
        params: Record<GetParams<Path>, string | number> &
          Paths[Path] & { hash?: string }
      ]
): string {
  const [param] = params
  if (param === undefined) return path
  return (
    path.replace(/:(\w+)/g, (_, key) => (param as any)[key]) +
    (param.query ? "?" + new URLSearchParams(param.query).toString() : "") +
    (param.hash ? "#" + param.hash : "")
  )
}
```

params 引数の型が何やら複雑ですね。これは「パスパラメータがあるときだけ`buildPath`の第２引数を必須にする」というのを実現しています。
(参考：[【TypeScript】関数で 1 つめの引数に応じて 2 つめの引数のオプショナルを切り替える](https://zenn.dev/kiyoshiro9446/scraps/3927451da029a0))
(一部、苦肉の策で`as any`を用いています。よりよい方法、募集中です。)

なお、react-router-dom などを使うときはルーティングの定義のために、`/posts/:id`という文字列自体が欲しくなります。

```tsx
<Route element={<PostShow />} path="/posts/:id">
```

これを解決するために、以下の関数を定義します。

```ts
const echoPath = <Path extends keyof Paths>(path: Path) => path
```

受け取ったパスをただ返すだけの関数ですが、`buildPath`と同様、パスの補完が効いてくれます。

```tsx
<Route element={<PostShow />} path={echoPath("/posts/:id"))}>
                                             {// ^ 補完が効いてくれる}
```

# おまけ Next.js などのファイルベースルーティングへの対応

glob などを用いてディレクトリ構造から`/posts`, `/posts/:id`のようなパス文字列を生成すれば今回の手法が使えます。
拙作ですが [type-safe-path](https://github.com/KoichiKiyokawa/type-safe-path) というライブラリを作成しました。ファイルベースルーティングを採用しているフレームワーク向けに、CLI から型定義を生成し、提案手法と似たような `buildPath` を提供します。

# まとめ

少し多めの型定義と数行程度の関数で型安全リンクを実現する方法を解説しました。この手法には以下のメリットがあります。

- 補完が使いやすい: あいまい検索のように絞り込めます
- バンドルサイズが軽量：型定義は JavaScript に変換することで消えてくれるため、ランタイムには少量の関数しか残らない

今回のコードは[こちらの TypeScript Playground](https://www.typescriptlang.org/play?#code/C4TwDgpgBACghsAFgZygXigbwFBT1AcgHoBXZCAJ2QIC4soBHEykAfjoDsIA3SqAXwA0ufMTKVkRGgEsAJrXpMW7KF14UBw-IVLkqUuUTAB7ZMGp1MjZhTacefISLxi9kmbKOnzUk2YCS8pbWyvbqms6ENOIUgQpWSrYqao5aotF6gUYUxgBm0gA2EPEhtsEMdGYU0hwA5gICkcQ0iMa1xeU2dqoOGvzY-aCQUADGiBAjANbosAgoUBAAHsAQHLKoAEoTxhSyADxVNbWCil0qcBwgAgB8UKxQwBTMUHS5cAXk2EXAUHB0YxNphhHsxsNghtAAOIQYDwChwAC2yD28CQtwwqMQC2Wq3WUAABgASTCHOr8GjEmq5PgwfhESkcakaLZmfj4yL3GBQAA+UGhsLg8KRexZwGukTomOxKzWqCJJMeR3JDKZsDZHNgEp66jBuRIHBGwGkxg4UAARiRCrJMSi5tLcahJhAQHlZkhkNcABSRMBzSVzNJ4AB0Id9QuQdH5cMRyMxtyWMrxKQokW09wA2mGYypMch05iALpQABk9EQcGQiBUpPq-ALqfwdEzgpjdC2Ix2+yjLeFcZONZ5qhICLNlFupdz+bmRdLVnLlerirqAnrAEpKkv6jhtB2OGYoM2hUWMFmkZFpLkoJ7T+g0Bh9bIIPkuLJV1AKDCSBRTb6kJEP8AX4-nMQYfmABRwCMECelInoADoAO4ANSrkQxxXgA+icTogG+aC3NePa-KgFy4emOEFm+SENngnqEUKQaJLhdyEKwBBQEhPQIVAACqGwADIAMoQIKYzRki9GIoxXSrkGwDGIJm6em+dAEAQVE0VekkIkG86IG+9wEAAxOxnGnrpFZYqp6kDGCu77hMrRShgtpIPaspQDhrq5l6v6IP6SB4bcfl2SayDGEUQYFG0noWlamKeq4EgGJ4fjmAQJxWHIdAAIwCKuq7YPZEUQFFMVxQU1pzIlGSUFkYA5PkRQZfQMSBLlJxMeUqmtO07H8PlhXFZF0W1LFlqVQlSX6B4XhmO4aVxJlUDZVAOUnItsi5YNQA)で試せます

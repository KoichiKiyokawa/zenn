---
title: "remeda 〜TypeScriptユーティリティライブラリの結論〜"
emoji: "🛠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "ユーティリティライブラリ", "lodash"]
published: true
---

# はじめに

JS界隈のユーティリティライブラリといえば、[Lodash](https://lodash.com/)が有名です。
しかし、Lodashはバンドルサイズが大きすぎるという問題があり、毛嫌いしている人も多いです。
https://bundlephobia.com/package/lodash

そんななか、Twitterの一部界隈で注目を集めているユーティリティライブラリが[remeda](https://remedajs.com/)です。

# 特徴

## 軽量

すべての関数をimportしてもたった5.3kBです。
https://bundlephobia.com/package/remeda
さらに、ES Module形式が提供されているため、使っていない関数はビルド時にtree shakeで除外できます。
バンドルサイズが重要なフロントエンドのプロジェクトにも導入しやすいです。

## pipe 関数が優秀

`pipe`関数が優秀で、TypeScriptの補完がバチバチに効きます。

![補完が効いている様子](/images/utility-remeda/pipe.gif)
(コードは[公式ドキュメント](https://remedajs.com/)より引用)

ちなみにLodashの場合は以下のようなメソッドチェーンでパイプする都合上、すべての関数が読み込まれてしまって、バンドルサイズが激増してしまいます。

```ts:Lodashの場合
_(users)
  .filter(x => x.gender === 'f')
  .groupBy(x => x.age)
  .value()
```

(コードは[公式ドキュメント](https://remedajs.com/)より引用)

それに対し、remedaはtree shakingしやすいAPIで設計されており、使用した関数のみがバンドルに含まれます(今回の例でいうと`pipe`, `filter`, `groupBy`のみ)。

## JSDoc が非常に丁寧

![jsdoc](/images/utility-remeda/jsdoc.png)
JSDocが丁寧に書かれているおかげで、使おうとしている関数の説明がその場で読めます。もうドキュメントサイトとエディタを往復する必要はありません。

# さいごに

軽量で使いやすく、一部界隈で注目を集めているremedaをご紹介しました。
プロジェクトでユーティリティライブラリが欲しくなったときに選択肢の1つとして覚えておくと、いつか役立つかもしれません。

---
title: "Remeda 〜TypeScriptユーティリティライブラリの結論〜"
emoji: "🛠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "ユーティリティライブラリ", "lodash"]
published: true
---

# はじめに

JS界隈のユーティリティライブラリといえば、[Lodash](https://lodash.com/)が有名です。
しかし、Lodashはバンドルサイズが大きすぎるという問題があり、毛嫌いしている人も多いです。
https://bundlephobia.com/package/lodash

そんななか、Twitterの一部界隈で注目を集めているユーティリティライブラリが[Remeda](https://remedajs.com/)です。

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

それに対し、Remedaはtree shakingしやすいAPIで設計されており、使用した関数のみがバンドルに含まれます(今回の例でいうと`pipe`, `filter`, `groupBy`のみ)。

また、Remedaの`pipe`は遅延評価に対応しており、愚直に関数を順番に適用するのではなく、イテレート回数が少なくなるように実行してくれます。

```ts:遅延評価の例
// ユニークな値を先頭から3つ取り出す
const arr = [1, 2, 2, 3, 3, 4, 5, 6];

const result = R.pipe(
  arr,
  R.map(x => {
    console.log('iterate', x);
    return x;
  }),
  R.uniq(),
  R.take(3)
); // => [1, 2, 3]

/**
 * 本来なら、mapのcallback関数が8回実行されそうだが、、、
 *
 * Console output:
 * iterate 1
 * iterate 2
 * iterate 2
 * iterate 3
 *
 * 4回に抑えられている
 * /
```

(コードは[公式ドキュメント](https://remedajs.com/)より引用)

## JSDoc が非常に丁寧

![jsdoc](/images/utility-remeda/jsdoc.png)
JSDocが丁寧に書かれているおかげで、使おうとしている関数の説明がその場で読めます。もうドキュメントサイトとエディタを往復する必要はありません。

# さいごに

軽量で使いやすく、一部界隈で注目を集めているRemedaをご紹介しました。
プロジェクトでユーティリティライブラリが欲しくなったときに選択肢の1つとして覚えておくと、いつか役立つかもしれません。

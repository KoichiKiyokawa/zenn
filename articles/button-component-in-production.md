---
title: Reactプロダクトにおける最強のButtonコンポーネントを考える
emoji: "💪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "component"]
published: false
---

# 要件

- aタグ、summaryタグなど任意のタグとして振る舞える

# プロダクトでButtonコンポーネントを汎用的にする必要はあるのか

- 開発序盤であれば、汎用化しないのもあり
  - 俗に言う「早すぎる抽象化はやめろ」
  - 各ページでスタイルが冗長化しても良い
  - 冗長化が3, 4箇所以上になったら、汎用化する
- デザイナーとも相談。デザイナーによっては、「ボタンとリンクで同じUIにしない」という人もいるはず。
  - プロダクトによってももちろん変わりそう
- ただし、迷ったら汎用化しておくのをおすすめする。
  - 後述するが、かなり実装コストは低めなので、早い段階でペイできるはず。

ここで、1つの実装例をとりあげる。この例を通して「汎用的なコンポーネント」というイメージを共有しつつ、汎用的にしなかった場合、どんなデメリットがあるかを考えていく。
誰もが一度は思うつく実装法として、`href`引数が渡された時だけ、aタグとして振る舞う実装がある。

```tsx
<Button>普通のボタン</Button> // -> <button>普通のボタン</button>
<Button href="...">リンクボタン</Button> // -> <a href="...">リンクボタン</a>
```

このパターンのデメリットとして、「`next/link`(内部リンク)と`a`(外部リンク)を使い分けたい」という場合に困ることがあげられる。一応、`external`引数がある場合は外部リンクにする、みたいな方法はある。しかし、今度はsummaryタグとしても振る舞いたくなったらどうするか。そのたびに引数を増やしてButtonコンポーネントの中身を変える必要があるが現実的ではない。なぜなら、プロダクトコードで何箇所にも使われているButtonコンポーネントの実装に手を加えるのは、たとえテストが充実していても気が引けるからだ。その結果、全く同じ見た目の別コンポーネントができてしまい、Buttonコンポーネントを作った意味が薄れていく。(これは、SOLID原則でいうとOpen-Closed原則に反するのが原因)。
:::message
ただし、プロダクトによっては内部リンクしかない場合もある。その場合はしばらくこの実装法でも良いかもしれない。
:::

したがって、`a`タグ、`summary`タグ、`Link`コンポーネントなど、どの要素としても振る舞える汎用性を持ったButtonコンポーネントを早すぎず遅すぎないフェーズで作成するのを推奨したい。ただし、汎用性のために実装が複雑化して、メンテコストが上がったら本末転倒である。次の章からは、ある程度実装が簡単で汎用性も担保した実装パターンを見ていく。

# 汎用的なButtonコンポーネントの実装パターン

## `as`パターン

Chakra UIなどで使われているパターン。
少々複雑な型パズルを必要とするため実装コストが高い。コンポーネントを使う際、型エラーも非常にわかりづらい。
UIライブラリの作者が実装するのは良いが、プロダクトで使うのはおすすめしない。

## `render`パターン

[jsx-a11y/anchor-has-content
](https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/1adec3517fc2c9797212ca4d38858deed917e7be/docs/rules/anchor-has-content.md)に反するのが大きなデメリット
![Eslintでエラーが出る様子](/images/button-component-in-production/eslint-error.png)

## `element`パターン

`render`パターンの亜種。

```tsx
<Button element={<a href="...">...</a>} />
```

- button以外のタグとして振る舞いたい場合は、`element`引数を指定する。
- `element`引数を指定すると、`children`は渡せなくなる。

デメリットとして、elementにわたす要素が長くなると見通しが悪くなる。

## `asChild`パターン

### デメリット

冒頭で、buttonタグの中にaタグを入れるのは仕様上許されていないと述べた。

```tsx
<Button asChild>
  <a href="...">...</a>
</Button>
```

見た目上、「buttonタグの中にaタグを入れている感」がすごい。HTML仕様が身にしみている人からすると、気持ち悪く感じるはず。
見た目上の問題は気にしなければ良いのだが、実装の際、**`asChild`を指定し忘れた場合、仕様違反のコードになってしまう**というのは大きな罠と言える。
ただし、これを除けば目立ったデメリットがなく、個人的には一番おすすめのパターン。

## インナーコンポーネントパターン

実装するにあたって、難しい知識が一切必要ない。将来のエンジニアの知識量に依存せず、保守しやすいというメリットはある。
しかし、単体では使えず、他のタグと組み合わせるのが前提という、不完全なコンポーネントを作ることになる点を受け入れられるかどうかがポイント。
強くおすすめはしない。

```tsx
<button type="button">
  <ButtonInner variant="primary">ボタン</ButtonInner>
</button>

<a href="...">
  <ButtonInner variant="primary">ボタン</ButtonInner>
</a>
```

ちなみに、aタグの子要素にdivはおけないので、`ButtonInner`コンポーネントを実装する際はspanタグを使う必要があることに注意。

# 参考

- [ReactコンポーネントでレンダリングされるHTML要素の種類を変更可能にするためのパターン](https://yuheiy.com/2023-06-03-react-changeable-element-type-patterns)
  - 名記事。ReactのUIライブラリにおけるButtonコンポーネントの実装パターンを4種類くらい紹介し、それぞれのメリデメが比較されている。

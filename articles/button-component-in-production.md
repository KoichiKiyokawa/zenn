---
title: Reactプロダクトにおける最強のButtonコンポーネントを考える
emoji: "💪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "component"]
published: false
---

# 要件

- aタグ、summaryタグなど任意のタグとして振る舞える

# プロダクトでButtonコンポーネントを

# パターン

## `as`パターン

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

## インナーコンポーネントパターン

実装するにあたって、難しい知識が一切必要ない。将来のエンジニアの知識量に依存せず、保守しやすい。

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

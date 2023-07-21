---
title: "Next.js + Vitestの組み合わせでnext/imageでエラーがでる問題の対処"
emoji: "🏞️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "vitest", "vitePlugin"]
published: true
---

# はじめに

https://github.com/vercel/next.js/issues/45350

このissueに回答したが、日本語でも記事を残しておく。

# 問題の概要

```tsx
// app/page.tsx
import Image from "next/image";
import logo from "@/public/vercel.svg";

export default function Home() {
  return <Image src={logo} alt="Vercel Logo" />;
}
```

上記のように、`next/image`を使っており、かつ画像を静的importしているコンポーネントをテストする。

```tsx
import { test, expect } from "vitest";
import { render } from "@testing-library/react";
import Home from "./page";

test("can render", () => {
  const { asFragment } = render(<Home />);
  expect(asFragment()).toMatchInlineSnapshot();
});
```

すると、以下のようなエラーが出る。

```
stderr | app/page.test.tsx > can render
Error: Uncaught [Error: Image with src "/@fs/@/public/vercel.svg" is missing required "width" property.]
```

# 原因

Next.jsでは、画像ファイルをimportした際、次のようなオブジェクトになる。

```tsx
import logo from "@/public/vercel.svg"; // logo -> { src: '画像のパス'; width: 100; height: 100 }
```

これによってwidthやheightがImageコンポーネントへ自動的に渡り、開発者がわざわざ画像のwidthやheightを指定しなくても良くなる(という認識)。
一方で、Viteでは、以下のように文字列になる。

```tsx
import logo from "@/public/vercel.svg"; // logo -> '画像のパス'
```

この違いにより、`{ src: string; width: number; height: number }`
というオブジェクトが渡されることを想定している`Image`コンポーネントにstringを渡しているせいでエラーがでていた。

# 解決策

Viteのプラグインを作り、Next.jsの「画像をimportしたらオブジェクトを返す」という挙動を再現する。

```ts
function stubNextAssetImport() {
  return {
    name: "stub-next-asset-import",
    transform(_code: string, id: string) {
      // idにはimportしようとしているファイルのパスが入る。例：/User/hoge/project/public/vercel.svg
      if (/(jpg|jpeg|png|webp|gif|svg)$/.test(id)) {
        const imgSrc = path.relative(process.cwd(), id); // プロジェクトルートから画像ファイルまでのパスを計算
        return {
          code: `export default { src: '${imgSrc}', height: 1, width: 1 }`,
        };
      }
    },
  };
}
```

使い方は以下の通り。

```tsx
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  //                     👇👇👇
  plugins: [react(), stubNextAssetImport()],
  test: {
    environment: "jsdom",
  },
});
```

Viteのプラグインの書き方は以下の公式ドキュメントで学んだ。
https://ja.vitejs.dev/guide/api-plugin.html

---
title: "spyOnとmockの使い分けで迷ったら思考停止でspyOnにする"
emoji: "🧙‍♀️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["test", "typescript"]
published: false
---

# 概要

jestやvitestなどで、テスト対象の依存関数などをモックするとき、主に`spyOn`と`mock`の2種類があります。
筆者のようにこの使い分けで迷う人も少なくないはず。
この記事では両者を比較して、最終的には`spyOn`だけを使えば良いことを説明します。

# 比較

以下の`fetchUser`関数をテストするとします。

```ts
// apiはエンドポイントへリクエストを行うメソッド
import { api } from "./api";

async function fetchUser(id: string) {
  return api(`/users/${id}`);
}
```

※なお、エンドポイントへのリクエストをモックするのであればmswを使ったほうが良いという説もありますが、あくまで簡易的な例としてお考えください。

## spyOn のメリット

spyOnは自動で型補完が効きます。
spyOnはrestoreできます(もとの実装を復活させられます)

## mock のデメリット

jest.mock()に渡したパスが側っても、自動で変更されません。

---
title: "【TypeScript】DIをしないという選択のメリデメを考察する"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# MEMO

- DIしなくてもJestのモックを使うことで、「DBへ接続せずに単体テスト」は可能
- ただし、import文が増えてくると「何をモックすればよいか」がわかりづらくなってしまう。

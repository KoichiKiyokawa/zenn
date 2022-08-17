---
title: "REST API脳の人のためのGraphQL入門"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rest-api", "graphql"]
published: false
---

## 対象者
- REST APIをすでに知っている
- REST APIと比較しながらGraphQLを学びたい

## スキーマの書き方
TODO: openapiのyamlとgraphqlのスキーマの比較

### REST APIの場合
```yaml
```
### GraphQLの場合

```graphql
type User {
  id: ID!
  name: String!
  age: Int!
}

type Query {
  user: User
}
```
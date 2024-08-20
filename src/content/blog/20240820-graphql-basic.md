---
title: "GraphQL 學習筆記 - 概念"
description: "介紹前後端都應該知道的GraphQL概念"
pubDate: "2024-08-20"
heroImage: "/banner/graphql.png"
---

GraphQL 是一種前端與後端的溝通框架，透過名為 Schema Definition Language(SDL)將資料定義為 **node** 與 **edge** 構築出一個串連所有資料的 **graph schema**，透過 server 給予的 GraphQL 進入點，client 可以在這個 **graph** 中取得剛好符合我們需求的資料，從而避免 over fetching 所造成時間與流量的浪費，改善使用者體驗，另外 GraphQL 也會根據 schema 進行資料型別驗證，強化了 data fetching 時的型別安全性。

## RESTful 的問題

假定現在有一組 RESTful API

```plaintext
/users
/users/:id
/vehicles
/vehicles/:id
```

以及他們的資料型別

```typescript
type User = {
  id: string;
  name: string;
  vehicles: string[]; // Vehicle.id[]
};

type Vehicle = {
  id: string;
  type: "CAR" | "SCOOTER";
};
```

如果我們因應業務需求需要這樣的結構

```typescript
type UserDetail = {
  id: string;
  name: string;
  vehicles: Vehicle[];
};
```

按照 RESTful API 的架構，我們勢必要 fetch `/users/:id` 和 `/vehicles/:id` 數次，在撰寫邏輯把取得的資料組合成我們需要的結構，而 GraphQL 只需要一次 fetch 行為**，**就能直接取得我們所需要的所有資料與組合好的結構，讓我們讓我們看看它是怎麼達成的。

## GraphQL 如何解決問題

在 GraphQL 中，會先透過一套名為 Schema Definition Language(SDL)撰寫的 schema 定義資料的型別與關聯性  
schema.gql

```graphql
type User {
  id: ID!
  name: String!
  vehicles: [Vehicle]!
}

enum VehicleType {
  CAR
  SCOOTER
}

type Vehicle {
  id: ID!
  type: VehicleType!
  model: String!
}

type Query {
  users: [User!]!
  user(id: ID!): User
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserResponse!
}

input CreateUserInput {
  name: String!
}

type CreateUserResponse {
  code: Int!
  message: String!
  success: Boolean!
  user: User
}
```

我們看到 4 種 SDL 的 type：**Scalar**、**Object**、**Enum、Input**

- **Scalar**：SDL 的基礎型別，預設五種分別為 `ID` 、 `String` 、 `Int` 、 `Float` 、 `Boolean` 。
- **Object**：只要以 `type` 關鍵字建立的都屬於這類別，用來描特定的資料結構。
- **Enum**：用來儲存限定文字內容，通常用全大寫英文字母儲存。
- **Input** ：某些操作需要參數，但 GraphQL 並不能巢狀定義 type，因此當參數變得複雜，可以用 **Input** 來描述參數的 type ，同時使用 `input` 定義的型別不能當作回傳值，只能用在參數的位置 。

```graphql
type Query {
  users: [User!]!
  user(id: ID!): User
}
```

`type Query` 對整個 schema 來說是必要的型別，用來描述 client 可以查詢的進入點。  
`users` 與 `user` 是查詢的進入點名稱。  
`[User!]!` 表示 `users` 這個進入點回傳的資料型別是 `User` 的 Array，括號外的 `!` 表示 `users` 這個進入點回傳的資料不能為 `null`，括號內的 `!` 表示 `User` 的 Array 不能為空。  
`user(id: ID!): User` 表示 `user` 這個進入點接收一個 `id` 參數，參數型別為 `ID`，且參數不能為 `null`，回傳的資料型別是 `User`。

```graphql
type Mutation {
  createUser(input: CreateUserInput!): CreateUserResponse!
}
```

`type Mutation` 用來描述 client 可以進行資料修改的操作，`createUser` 是一個 mutation 的進入點，接收一個 `input` 參數，參數型別為 `CreateUserInput`，且參數不能為 `null`，回傳的資料型別是 `CreateUserResponse`。

## Client Side

根據 schema，我們就可以在 client 透過 GraphQL 提供的 query 語法來取得我們所需要的資料

```graphql
query GetUsers {
  users {
    id
    name
    vehicles {
      id
      type
    }
  }
}
```

這邊的意思是建立一個名為 `GetUsers` 的 query，透過 `users` 這個進入點取得 `id` 、 `name` 、 `vehicles` 這三個欄位的資料，其中 `vehicles` 這個欄位是一個 Array，我們可以在 `vehicles` 下取得 `id` 、 `type` 這兩個欄位的資料。

如果我們只需要使用者資料，不需要車輛資料，我們可以這樣寫

```graphql
query GetUsers {
  users {
    id
    name
  }
}
```

GraphQL 會根據我們的需求，只取得我們所需要的資料，避免 over fetching 的問題。

```graphql
query GetUser {
  user(id: "1") {
    id
    name
    vehicles {
      id
      type
    }
  }
}

"""apollo建議寫法"""
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    vehicles {
      id
      type
    }
  }
}

```

透過 `user` 這個進入點，我們可以傳入 `id` 參數取得特定使用者的資料，這邊的 `$id` 是一個變數，透過變數的方式來傳遞參數，這樣可以避免硬編碼，提高程式碼的可讀性、可重複使用性， `mutation` 的寫法與 `query` 類似，只是在 query 的地方改成 mutation。

```graphql
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    code
    message
    success
    user {
      id
      name
    }
  }
}
```

基礎的 GraphQL 概念就是這樣，接下來會分開介紹 client-side 與 server-side 的實作。

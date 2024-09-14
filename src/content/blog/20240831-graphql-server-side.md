---
title: "GraphQL 學習筆記 - 後端"
description: "介紹如何在後端透過Apollo Server來建立GraphQL API"
pubDate: "2024-08-31"
heroImage: "/banner/graphql.png"
---

GraphQL 可以用多種語言實現，下面範例使用 Typescript 與 Apollo 展示。

## 初始化

### 使用 terminal 進行套件安裝

```bash
npm install @apollo/server graphql graphql-tag
```

### 建立 server

```typescript
import { ApolloServer } from "@apollo/server";
import { startStandaloneServer } from "@apollo/server/standalone";
import { typeDefs } from "./schema";
import { resolvers } from "./resolvers";

const server = new ApolloServer({
  typeDefs,
  resolvers,
});

// 要使用 top-level await 需要 在 package.json 中設定 "type": "module"
const { url } = await startStandaloneServer(server, { port: 4000 });

console.log(`Server ready at ${url}`);
```

這樣我們就完成了一個簡單的 GraphQL Server，下面會逐步介紹如何建置 schema 與 resolver。

### 定義 schema

在 GraphQL 中，我們可以在 JS/TS 檔案中透過 literal string 定義 schema，也可以只用 `.graphql` 檔案來定義 schema。

schema.ts

```typescript
import { gql } from "graphql-tag";

export const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    vehicles: [Vehicle]!
  }
`;

// 也可以不透過 gql 來定義 schema，#graphql 是為了讓 editor 能夠辨識
export const typeDefs = `#graphql
  type User {
    id: ID!
    name: String!
    vehicles: [Vehicle]!
  }
`;
```

schema.graphql

```graphql
type User {
  id: ID!
  name: String!
  vehicles: [Vehicle]!
}
```

三種方式基本上等價，在引入方式會有些許不同，我們可以根據需求選擇使用哪種方式，這裡使用 `schema.ts` 作為範例。

#### 註解

在 schema 中，我們可以透過 `"` 來加入註解，這樣可以讓我們的 schema 更容易理解。

```typescript
export const typeDefs = gql`
  """
  使用三個雙引號來加入區塊註解
  """
  type User {
    "使用雙引號來加入單行註解"
    id: ID!
    name: String!
    vehicles: [Vehicle]!
  }
`;
```

在 Apollo Explorer 中，註解會被顯示在 schema 的說明中，讓使用者更容易理解 schema 的用途。

### 定義 resolver

schema 只是定義了資料的型別與關聯性，我們需要透過 resolver 來實現資料的取得。

例如我們的 schema 是

```typescript
const typeDefs = `#graphql
  type Hello {
    message: String!
  }

  type Query {
    hello: Hello!
  }
`;
```

上面這段 schema 我們定義了查詢的 `Query` object，裡面包含一個進入點 `hello`，這個進入點會回傳一個 `Hello` object，裡面包含一個 `message` 字串，要完成整個查詢的行為，我們需要透過 **resolver** 來實現。

```typescript
const resolvers = {
  Query: {
    hello: () => {
      return {
        message: "Hello World",
      };
    },
  },
};
```

resolver 會是 schema 的映射，每個 schema 上定義的資料都要有對應的取得方式，上面這段程式碼的結構是，先在 resolvers 中定義 `Query` 屬性告訴 GraphQL 如何解析 schema 中的 `Query` 查詢行為，因為我們的 schema 中有一個 `hello` 進入點，所以我們也要在 resolvers 中定義 `hello` 進入點的解析行為，resolver 本身是一個 function，這個 function 必須回傳與 schema 中定義的回傳值相同的結構，如範例中 `hello` 進入點回傳的結構是 `{ message: String }`，因此 resolver 也要回傳符合定義的 `{ message: 'Hello World' }`。

#### Resolver 參數

```typescript
const typeDefs = gql`
  type User {
    id: ID!
    name: String!
  }

  type Query {
    user(id: ID!): User
  }
`;
```

與先前的例子不同， `user` 進入點需要一個 `id` 參數，讓我們看看如何實現這個 resolver

```typescript
const resolvers = {
  Query: {
    user: async (parent, args, context, info) => {
      const user = await getUserById(args.id);
      return user;
    },
  },
};
```

resolver 可以接收四種參數：

- parent: 上一層的資料，通常在巢狀結構中會用到
- args: 進入點的參數
- context: 用來取得共享資料，例如認證資訊或資料庫連線
- info: 包含 schema 的資訊，通常不會用到

在 schema 中， `user` 需要一個 `id` 參數，這個參數我們可以透過 `args` 取得，藉此完成我們的查詢行為，另外當我們需要等待非同步行為時，我們可以透過 `async` 來定義 resolver。

#### Resolver Chains

讓我們再來看一個更實際的例子

```typescript
const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    vehicles: [Vehicle]!
  }

  type Vehicle {
    id: ID!
    model: String!
  }

  type Query {
    user(id: ID!): User
  }
`;
```

假設他們的在資料庫的型別如下

```typescript
type UserModel = {
  id: string;
  name: string;
};

type VehicleModel = {
  id: string;
  model: string;
  ownerId: string; // User.id
};
```

resolvers 中的 `user` 必須有辦法解析 schema 中 `vehicles` 這個欄位，但 db 中的 `UserModel` 是不包含 `vehicles` 的，要獲取 `vehicles` 直覺上我們可以這樣做

```typescript
const resolvers = {
  Query: {
    user: async (parent, args, context, info) => {
      const user = await getUserById(args.id);
      const vehicles = await getVehiclesByOwnerId(user.id);
      return {
        ...user,
        vehicles,
      };
    },
  },
};
```

我們完成了 schema 的解析，但這樣有個問題，GraphQL 讓我們能自由選擇需要的欄位，因此 `vehicles` 並不是必要內容，但每次我們在取得 `user` 時都會去取得 `vehicles`，這樣會造成不必要的資源浪費，讓我們看看如何解決這個問題。

首先我們在 `resolvers` 下定義一個 `User` 屬性告訴 GraphQL 這邊有 `User` 的解析行為

```typescript
const resolvers = {
  User: {},
  Query: {
    user: async (parent, args, context, info) => {
      const user = await getUserById(args.id);
      return user;
    },
  },
};
```

接著我們在 `User` 下定義 `vehicles` 的解析行為

```typescript
const resolvers = {
  User: {
    vehicles: async () => {},
  },
  Query: {
    user: async (parent, args, context, info) => {
      const user = await getUserById(args.id);
      return user;
    },
  },
};
```

如同先前提到的，resolver 會接收四個參數，其中第一個參數是 `parent`，這這個情況下，`vehicles` 的 parent 就是 `User`，我們可以透過 `parent` 取得 `User` 的資料，進而取得 `vehicles`。

```typescript
const resolvers = {
  User: {
    vehicles: async (parent) => {
      const vehicles = await getVehiclesByOwnerId(parent.id); // parent.id 就是 User.id
      return vehicles;
    },
  },
  Query: {
    user: async (parent, args, context, info) => {
      const user = await getUserById(args.id);
      return user;
    },
  },
};
```

至此我們完成了 `vehicles` 的解析拆分，這樣的好處是，當我們查詢 `user` 時，只有在需要 `vehicles` 時才會觸發 `User.vehicles` 這個 resolver，節省不必要的 fetch 行為。

讓我們重新審視一下上面的例子，在我們取得完整 `User` data 的過程中，GraphQL 實際上依序呼叫了

```plaintext
Query.user() -> User.id()
             -> User.name()
             -> User.vehicles() -> Vehicle.id()
                                -> Vehicle.model()
```

GraphQL 會根據我們查詢的 query 結構不斷向下解析，直到遇到 Enum 或 Scalar 為止，這樣的行為稱為 **Resolver Chains**。

#### context

在前面 resolver 的範例中，已經使用到 `parent` 與 `args`，現在我們來看看 `context`。

在介紹 `context` 的同時，要另外介紹一個 Apollo Server 推薦的工具 `RESTDataSource`，這個工具可以讓我們更方便的與 RESTful API 串接，並且為我們實現 cache 與 error handling 等功能。

安裝

```bash
npm install @apollo/datasource-rest
```

/src/datasources/users-api.ts

```typescript
import { RESTDataSource } from "@apollo/datasource-rest";

export class UsersAPI extends RESTDataSource {
  baseURL = "https://path/to/api";

  getUsers() {
    return this.get("/users");
  }
}
```

使用 RESTDataSource 的好處，我們可以在單一檔案中簡單的管理 API，並且 RESTDataSource 會自動處理根據我們的設定使用 cache 提高性能，並且在發生錯誤的時候在 Error 中包含更多資訊。

接著我們要在 server 中使用這個 API

/src/index.ts

```typescript
import { UsersAPI } from "./datasources/users-api";

const { url } = await startStandaloneServer(server, {
  port: 4000,
  context: async () => {
    const { cache } = server;
    return {
      dataSources: {
        usersAPI: new UsersAPI({ cache }),
      },
    };
  },
});
```

這樣我們就可以在 resolver 中使用 `context` 取得我們的 API

```typescript
const resolvers = {
  Query: {
    users: async (_parent, _args, { dataSources }) => {
      return dataSources.usersAPI.getUsers();
    },
    user: async (_parent, { id }, { dataSources }) => {
      return dataSources.usersAPI.getUserById(id);
    },
  },
};
```

至此我們完成了 GraphQL 的設定，Mutation 行為與 Query 行為基本上是一樣的，需要在 resolvers 中定義對應的 `Mutation` 屬性，並且回傳指定的結構。

### Typescript

與 client side 一致，server side 也可以透過 codegen 來自動生成型別定義，幫助我們完成更嚴謹的型別檢查。

安裝

```bash
npm install -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-resolvers
```

初始化

```bash
npx graphql-code-generator init
```

完成後會在 root 得到一個類似這樣的檔案 `codegen.ts`

```typescript
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  overwrite: true,
  schema: "src/schema.ts",
  generates: {
    "src/generated/types.ts": {
      plugins: ["typescript", "typescript-resolvers"],
      config: {},
    },
  },
};

export default config;
```

接著調整 package.json

```json
{
  "scripts": {
    "codegen": "graphql-codegen --config codegen.ts"
  }
}
```

執行

```bash
npm run codegen
```

運行結束後我們就可以在 `src/generated/types.ts` 中看到自動生成的型別定義。

這時讓我們回到 resolvers

```typescript
import { Resolvers } from "./generated/types.js";

export const resolvers: Resolvers = {
  // ...
};
```

我們現在可以為 resolvers 添加 `Resolvers` 型別，這是根據我們的 schema 自動生成的型別，這樣我們就可以在 resolvers 中使用更嚴謹的型別檢查，但我們還有最後一塊拼圖需要補上。

```typescript
  User: {
    vehicles: async (parent, _args, { dataSources }) => {
      return dataSources.vehiclesAPI.getUserVehicles(parent.id);
    },
  },
```

如果我們讓指標 hover `dataSources` 會發現 TS 無法辨識 `vehiclesAPI`，這是因為 codegen 並無法自動辨識我們的 `dataSources`，讓我們來看看如何解決這個問題。

我們要在 `src/context.ts` 中定義 `context` 的型別

```typescript
import { UsersAPI } from "./datasources/users-api";

export type ContextValue = {
  dataSources: {
    usersAPI: UsersAPI;
  };
};
```

接著更新 'codegen.ts'

```typescript
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  overwrite: true,
  schema: "src/schema.ts",
  generates: {
    "src/generated/types.ts": {
      plugins: ["typescript", "typescript-resolvers"],
      config: {
        contextType: "../context#ContextValue", // 添加這行，這個路徑是相對於生成的檔案
      },
    },
  },
};

export default config;
```

重新運行 codegen

```bash
npm run codegen
```

再回到 resolvers 我們就可以看到 TS 已經辨識 `vehiclesAPI` 了。

再讓我們看回到 resolvers 和 datasource

/src/datasources/users-api.ts

```typescript
import { RESTDataSource } from "@apollo/datasource-rest";

export class UsersAPI extends RESTDataSource {
  baseURL = "https://path/to/api";

  getUsers() {
    return this.get("/users");
  }
}
```

如果我們把指標 hover `get()` 會發現 Typescript 給我們的 type 是

```typescript
(method) RESTDataSource<CacheOptions>.get<any>(path: string, request?: GetRequest<CacheOptions>): Promise<any>
```

我們收到的是 `Promise<any>`，這樣會讓我們失去了型別檢查的好處，要解決這個問題，我們需要為 `RESTDataSource` 添加型別。

/src/models.ts

```typescript
export type UserModel = {
  id: string;
  name: string;
};
```

/src/datasources/users-api.ts

```typescript
import { RESTDataSource } from "@apollo/datasource-rest";
import { UserModel } from "../models";

export class UsersAPI extends RESTDataSource {
  baseURL?: string = "http://localhost:3210";

  getUsers() {
    return this.get<UserModel[]>("/users");
  }
}
```

再次 hover `get()` type 轉變為

```typescript
(method) RESTDataSource<CacheOptions>.get<UserModel[]>(path: string, request?: GetRequest<CacheOptions>): Promise<UserModel[]>
```

Typescript 現在可以辨識 `get()` 回傳的型別了。

但回到 resolvers，我們會發現 Typescript 在 `users` 出現警告，

/src/resolvers.ts

```typescript
const resolvers = {
  Query: {
    users: async (_parent, _args, { dataSources }) => {
      return dataSources.usersAPI.getUsers();
    },
  },
};
```

```
類型 UserModel 不可指派給類型 User
```

這是因為在預設狀況下，codegen 會使用 schema 中的型別為 users 建立接收資料的型別，

我們可以在 `/generated/types.ts` 中找到

```typescript
export type User = {
  __typename?: "User";
  id: Scalars["Int"]["output"];
  name: Scalars["String"]["output"];
  vehicles: Array<Maybe<Vehicle>>;
};
```

\_\_typename 是 Apollo Server 自動為我們生成的欄位，用來辨識當前物件的型別，他是 optional 的因此與我們的 `UserModel`不衝突，但 `vehicles` 屬性是必要的，必須接收到一個 Array，我們的 `UserModel` 無法滿足這個欄位，所以 Typescript 會報錯。

要解決這個問題讓我們再次打開 `codegen.ts`

```typescript
const config: CodegenConfig = {
  overwrite: true,
  schema: "src/schema.ts",
  generates: {
    "src/generated/types.ts": {
      plugins: ["typescript", "typescript-resolvers"],
      config: {
        contextType: "../context.js#ContextValue",
        /* 新增區塊 */
        mappers: {
          User: "../models.js#UserModel",
        },
        /* 新增區塊 */
      },
    },
  },
};
```

這樣 codegen 在產生 `Resolvers` type 的時候就會使用我們指定的型別，而不是根據 schema 生成的型別。

重新執行 codegen

```bash
npm run codegen
```

再次回到 resolvers，Typescript 就不會報錯了。

至此我們完成了 Apollo Server 的建置。

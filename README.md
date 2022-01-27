## 用語

### relay-compiler

テンプレートリテラルで書いた GraphQL を`relay-runtime`上で実行可能な形式に変換してくれる。

## relay-runtime

実際にリクエストを実行しているクライアント。
フェッチ、読み取り、正規化を担当する。

## relay-react

React 上で`relay-runtime`使えるようにしてる。

## 基本構文

```ts
import { graphql } from "relay-runtime";
export default graphql`

  // $xxxはjs側からから受け取る引数
  mutation sample_Mutation($id: Number!, $name: String!) {

    // jsから受け取った引数をmutationの引数にセット
    UpdateUserName(_set: { id: $id, name: $name }) {
      // mutationの返り値
      succeed
    }
  }
`;
```

なぜ動くかというと、DB の構造が全て schema に定義されているから。
作成は、

```bash
yarn run schema
```

## schema の見方

```schema.graphql
type mutation_root {
  UpdateUserName(_set: SendContactMail!): UpdateUserNameResponse
}

// 引数の型定義はinputから始まる
input SendContactMail {
  id: Number!
  name: String!
}

type UpdateUserNameResponse {
  succeed: Boolean!
}

```

## mutation

作成した mutation の型定義は、

```bash
yarn run relay
```

すると`__generated__`以下に型定義フォルダが作成される。

## React Relay

```tsx
import { useMutation } from 'react-relay';

export const Sample = () => {
  const [commitMutation] = useMutation<updateUserName_Mutation>(mutation);
  commitMutation({
    id: 1,
    name: "桃山みらい",
  )},
  onCompleted() {
    alert("done!")
  },
  onError() {
    alert("error!")
  },
}
```

## Fragment へ引数を渡す

### GraphQL 側

```ts
import { graphql } from "relay-runtime";

export default graphql`
  query sampleQuery($company_code: String!) {
    ...fragment_Item @arguments(id: $id)
  }
`;

export const fragment = graphql`
  fragment fragment_Item on query_root // relay特有の名前
  @argumentDefinitions(id: { type: "String" }) {
    company_reviews_connection(
      where: { id: { _eq: $id } }
    ) {
      edges {
        node {
          title
        }
      }
    }
  }
`;
```

### js 側

```tsx
const data = useQuery(sampleQuery, {
  id: "momoyama",
});
```

https://relay.dev/docs/api-reference/graphql-and-directives/#arguments

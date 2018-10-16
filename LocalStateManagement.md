# Useful APIs

## Basic Usage

- [Link](https://www.apollographql.com/docs/react/essentials/local-state.html)

```js
import ApolloClient from "apollo-boost";
import { defaults, resolvers } from "./resolvers";
// apollo-link-state의 clientState가 이미 apollo-boost에 내장되어있으니 바로 써도 된다. 이쯤되니 그냥 apollo-boost만 알아도 될것같은 느낌...!

const client = new ApolloClient({
  uri: `https://someURL_u_wanna_fetch`,
  clientState: {
    defaults,
    resolvers,
    typeDefs
  }
});
```

- defaults : Client 가 처음 initialized 될 때 Apollo Cache 에 넣고싶은 기본값

- resolvers : Query, Mutation 처리를 Client 단에서 하기 위한 함수들을 넣어준다.

- typeDefs : Client 단에서 쓸 type 의 데이터 형식들을 정의해준다.
  - schema, Query, Mutation, 개인이 정의한 데이터 타입(Note)

## Updating Local Data

- Two Methods

  - 첫째는 평소에 Nicolas(Nomad Coders Teacher)이 항상 하던대로 Mutation 과 Resolver 을 이용하는 방법이다. Docs 에 따르면 이 방법은 이미 캐시에 존재하는 데이터를 manipulate 할 때 쓰면 좋다고 한다.
  - 두번째 방법은 아래 나와있는것처럼 Query tag 나 ApolloConsumer 을 이용한 직접 쓰기 방식이다. 캐시에 아직 존재하지 않는 데이터 같은 경우에는 이를 사용하는게 더 나을듯?
  - 간단하게 얘기하면 client.writeData == setState 같은 느낌 / resolvers => redux 같은 관리방식(But much better :D)

- Using ApolloConsumer

```js
import React from "react";
import { ApolloConsumer } from "react-apollo";

import Link from "./Link";
// 이전과 마찬가지로 ApolloConsumer을 이용하여 직접 client에 데이터를 적을 수도 있다.
const FilterLink = ({ filter, children }) => (
  <ApolloConsumer>
    {client => (
      <Link
        onClick={() => client.writeData({ data: { visibilityFilter: filter } })}
      >
        {children}
      </Link>
    )}
  </ApolloConsumer>
);
```

- Using Query Tag

```js
import React from "react";
import { Query } from "react-apollo";
import gql from "graphql-tag";

import Link from "./Link";

// 이렇게 client를 붙여줘야 캐시에서 찾는다! 매번 까먹는데 까먹지 말기...
const GET_VISIBILITY_FILTER = gql`
  {
    visibilityFilter @client
  }
`;

// Remember to set an initial value for visibilityFilter with defaults
const FilterLink = ({ filter, children }) => (
  <Query query={GET_VISIBILITY_FILTER}>
    {({ data, client }) => (
      <Link
        onClick={() => client.writeData({ data: { visibilityFilter: filter } })}
        active={data.visibilityFilter === filter}
      >
        {children}
      </Link>
    )}
  </Query>
);
```

## Resolvers

```js
export const resolvers = {
  Mutation: {
    toggleTodo: (_, variables, { cache, getCacheKey }) => {
      const id = getCacheKey({ __typename: "TodoItem", id: variables.id });
      const fragment = gql`
        fragment completeTodo on TodoItem {
          completed
        }
      `;
      const todo = cache.readFragment({ fragment, id });
      const data = { ...todo, completed: !todo.completed };
      cache.writeData({ id, data });
      return null;
    }
  }
};
```

## Queries

- 마찬가지로 @client decoration 만 붙여주면 된다.

```js
import React from "react";
import { Query } from "react-apollo";
import gql from "graphql-tag";

import Todo from "./Todo";

const GET_TODOS = gql`
  {
    todos @client {
      id
      completed
      text
    }
    visibilityFilter @client
  }
`;

const TodoList = () => (
  <Query query={GET_TODOS}>
    {({ data: { todos, visibilityFilter } }) => (
      <ul>
        {getVisibleTodos(todos, visibilityFilter).map(todo => (
          <Todo key={todo.id} {...todo} />
        ))}
      </ul>
    )}
  </Query>
);
```

## Defaults

- Default values that u wanna use!

```js
const defaults = {
  todos: [],
  visibilityFilter: "SHOW_ALL"
};
```

## TypeDefs(Schema)

- Client 단의 state 를 위한 Type Definitions

```js
const typeDefs = `
  type Todo {
    id: Int!
    text: String!
    completed: Boolean!
  }

  type Mutation {
    addTodo(text: String!): Todo
    toggleTodo(id: Int!): Todo
  }

  type Query {
    visibilityFilter: String
    todos: [Todo]
  }
`;
```

## Local Data + Remote Data

```js
// @client decoration을 붙여주고
const GET_DOG = gql`
  query GetDogByBreed($breed: String!) {
    dog(breed: $breed) {
      images {
        url
        id
        isLiked @client
      }
    }
  }
`;

// 해당 필드를 resolver에 추가적으로 포함시킨다.
const resolvers = {
  Image: {
    isLiked: () => false
  }
};
```

# Useful APIs

## Basic Usage

- [Link](https://www.apollographql.com/docs/react/essentials/mutations.html)

```js
import React from 'react';
import { Mutation } from "react-apollo";
import { gql } from "apollo-boost";

const ADD_TODO = gql`
  mutation AddTodo($title: String!, $content: String!) {
    addTodo(title: $title, content: $content) {
      id
      title
      content
    }
  }
`

// Mutation function 말고도 Mutation이 끝난 후의 data, 끝나기 전의 loading, error 등을 받아올 수 있다.
const AddTodo = () => (
  <Mutation mutation={ADD_TODO}>
    {(addTodo, {data, loading, error}) => (
      <div>
        <form onSubmit={e => {
          e.preventDefault();
          addTodo({
            variables: {title: this.title, content: this.content
            });
        }}>
          <input onChange={sth} value={this.title}/>
          <input onChange={sth} value={this.content}/>
        </form>
      </div>
    )}
  </Mutation>
)
```

## 기타 알아봤으면 하는 API 들

- Mutation tag
  - optimisticResponse : 서버에서 응답이 오기 전에 mutation 으로 최적화 처리를 한다! 이건 꼭 봐봐야할듯.
  - refetchQueries : mutation 이 끝난 후 [{query, variables}]를 받아서 원하는 쿼리를 자동으로 보내주는 함수다. awaitRefetchQueries 랑 같이 보면 좋을듯.
  - context : Apollo Link 와 관련된것같은데 잘 모르겠음...
- Render Prop
  - Mutation Function(addTodo 같은거임) : 첫째 인자로 오는거다. variables, optimisticResponse 만 넣어줄 수 있는 함수다.
  - Mutation Result(둘째 인자)
    - data, loading, error, called, client 가 온다.
    - client 는 ApolloClient 인스턴스가 오므로 여기에서 client.writeData / client.readQuery 등의 작업을 할 수 있다.

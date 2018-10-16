# Useful APIs

## Basic Usage

- [Link](https://www.apollographql.com/docs/react/essentials/queries.html)

```js
import { Query } from "react-apollo";
import { gql } from "apollo-boost";
const id = 3;
<Query
  query={gql`
  {
    ...
  }
`}
  variables={{ id }}
>
  {({ loading, data, error }) => beautiful_thing}
</Query>;
```

## Polling & Refetching

```js
const dont = true or false;
const intervalTime = 500;
// refetch는 유저가 원할 때 다시 fetch를 해주는 역할을 한다.
// refetch 말고도 networkStatus도 있는데 크게 쓸것같진 않다. 1~8까지 있음.
<Query
  query={...}
  variables={...}
  skip={!dont}
  pollInterval={intervalTime}
>
  {({loading, data, error, refetch}) => {
    if (loading) return null;
    if (error) Error("Error occured");
    return (
      <>
        <div>{data.beautiful}</div>
        <button onClick={() => refetch()}>Refetch with the variables & queries</button>
      </>
    )
  }}
</Query>
```

## Manually Firing a Query

```js
import React, { Component } from "react";
import { ApolloConsumer } from "react-apollo";

// Query는 자동으로 쿼리 call을 날리는데 이게 싫다면
// ApolloConsumer안에 있는 client를 이용해서 직접 쿼리를 날려라.
// client.query(sth)

class DelayedQuery extends Component {
  state = { dog: null };

  onDogFetched = dog => this.setState(() => ({ dog }));

  render() {
    return (
      <ApolloConsumer>
        {client => (
          <div>
            {this.state.dog && <img src={this.state.dog.displayImage} />}
            <button
              onClick={async () => {
                const { data } = await client.query({
                  query: GET_DOG_PHOTO,
                  variables: { name: "dogname" }
                });
                this.onDogFetched(data.dog);
              }}
            >
              Manually Refetch :)
            </button>
          </div>
        )}
      </ApolloConsumer>
    );
  }
}
```

## 기타 알아봤으면 하는 API 들

- Query tag
  - partialRefetch
  - onCompleted
  - ssr(nextjs 에서 필요할것같다.)
  - children : query 의 결과에 따라서 돌려주는거다. 지금 그냥 우리가 하고있는거임!
- Render Prop
  - fetchMore : pagination 을 구현할 때 쓰인다고 함.
  - updateQuery : query 의 결과에 해당하는 캐시값을 변경시킬 때 쓰인다고 한다.

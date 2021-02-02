---
title: GraphQL과 Apollo Client 학습 정리
layout: post
date: '2021-02-03 01:17:00'
author: 진혀쿠
tags: GraphQL Apollo Client
cover: "/assets/instacode.png"
categories: WEB
---

## 왜 GraphQL을 써야 하는가?

---

1. 강력한 스키마 타입

    서버에 스키마가 존재해야 클라이언트에서 해당 스키마를 사용할 수 있게 설계되어 있다. 일치하는 스키마가 없을 경우 에러가 뜨기 때문에 잘못된 스키마 정의 때문에 개발자가 고생할 필요가 없다. 또한 API 명세가 자동으로 생성되기 때문에 따로 작성할 필요가 없는 것 역시 장점이다. 

2. Overfetching과 Underfetching 방지

    RESTful API에서 자주 발생하는 문제를 꼽으라면 단연 Overfetching과 Underfetching이 뽑힌다. 고정된 endpoint로 요청을 보내기 때문에 정해진 데이터를 받아와야 하고 그 과정에서 문제가 발생한다. GraphQL에서는 원하는 데이터를 지정할 수 있기 때문에 위의 문제를 해결할 수 있다.

3. 생산적이다

    GraphQL의 라이브러리인 Apollo에서는 chaching , realtime 또는 optimistic UI updates 를 자유롭게 활용할 수 있다. 이를 통해 다른 추가 작업 없이 보다 효율적이고 생산적으로 개발을 진행할 수 있다.

4. API 방식

    GraphQL은 Schema Stitching이라는 기법을 통해 여러 API를 하나의 API로 만들 수 있다. 따라서 기존의 API가 여러 번 요청을 보내야 했던 일에 대해 하나의 요청으로 해결할 수 있게 만들어준다.

5. 넓은 생태계

    많은 사람이 GraphQL을 사용, 사랑하는 만큼 충분히 넓은 개발 생태계가 있다. 즉 정보를 얻을 수 있는 방법이 많으며 좋은 라이브러리가 많이 존재한다!

## Apollo란

---

GraphQL은 하나의 형식, 즉 명세일 뿐이기 때문에 이를 구현할 방법이 필요한데 그 중 하나가 Apollo이다.

Apollo는 Backend와 Frontend 모두를 지원하며 사용하기 쉽고 많은 기능들을 제공하기 때문에 널리 쓰인다.

## GraphQL 사용법 정리

---

## Queries and Mutations

### Field

GraphQL은 객체에 대한 특정 필드를 요청하는 것이 매우 간단하다. 다음의 예제를 보자.

```jsx
{
  hero {
    name
  }
}
```

```jsx
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

쿼리와 결과가 정확히 동일한 형태인 것을 알 수 있는데 이는 서버에서 클라이언트가 요청하는 필드를 정확히 알고 있기 때문이다.

필드는 객체를 참조할 수도 있다.

```jsx
{
  hero {
    name
    # 쿼리에 주석을 쓸 수도 있습니다!
    friends {
      name
    }
  }
}
```

```jsx
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

friends라는 객체를 쿼리에 포함시킨 것을 볼 수 있다.

### Arguments

GraphQL은 필드에 인자를 전달할 수 있다.

```jsx
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

```jsx
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 5.6430448
    }
  }
}
```

REST에서는 요청에 쿼리 파라미터나 URL 세그먼트 같은 단일 인자들만 전달할 수 있었다. 하지만 GraphQL에서는 모든 필드와 중첩된 객체가 인자를 가질 수 있기 때문에 여러 번의 API fetch 대신 한 번의 요청으로 처리가 가능하다.

### Alias

같은 쿼리에 다른 인자가 들어가는 경우는 한 번에 처리 할 수 없기 때문에 Alias를 사용하여 이를 구분할 수 있다.

```jsx
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

```jsx
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

hero 필드를 요청할 때 다른 인자를 넘겨주는 경우 쿼리 앞에 별칭을 적어줌으로써 데이터를 받아 올 때 별칭으로 받아오는 것을 확인할 수 있다.

### Fragment

쉽게 설명하면 재사용이 가능한 쿼리라고 보면 된다. 코드의 중복을 방지할 수 있는 방법이다.

```jsx
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}
```

```jsx
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "friendsConnection": {
        "totalCount": 4,
        "edges": [
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          },
          {
            "node": {
              "name": "C-3PO"
            }
          }
        ]
      }
    },
    "rightComparison": {
      "name": "R2-D2",
      "friendsConnection": {
        "totalCount": 3,
        "edges": [
          {
            "node": {
              "name": "Luke Skywalker"
            }
          },
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          }
        ]
      }
    }
  }
}
```

comparisonFields를 통해 하나의 쿼리로 2가지 정보를 가져온 예제이다.

또한 쿼리나 뮤테이션에 선언된 변수는 프래그먼트에 접근할 수 있다. 

### Operation name

Query를 작성할 때 보통 Operation name을 생략하고 적곤 했는데 실제 개발시에는 적어주는 것이 좋다. 작업 타입은 query, mutation, subscription이 올 수 있고 작업 이름은 쿼리에 대해 적절한 이름을 붙여주면 된다. 우리가 함수명이나 변수명에 의미를 부여하는 것처럼 이것 역시 비슷하게 보면 된다.

```jsx
// Operation name이 존재하는 쿼리
query HeroNameAndFriends {
  hero {
    name
    friends {
      name
    }
  }
}

// Operation name이 존재하지 않는 쿼리
{
	hero {
		name
		friends {
			name
		}
	}
}
```

### Variable

argument를 꼭 쿼리 내에 작성해야 하는 것은 아니다. 아래의 예제와 같이 변수를 선언한 후 변수에 어떤 값을 보내 줄 것인지 정할 수 있다. 

```jsx
query HeroNameAndFriends($episode: Episode = JEDI) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}

// VARIABLES
{
  "episode": "JEDI"
}
```

변수 정의는 위 쿼리에서 `($episode: Episode)` 부분이다. 변수명 + 타입의 형태로 변수 정의가 이루어진다. 또한 위의 예제와 같이 기본값도 지정이 가능하다.

선언된 모든 변수는 scalar, enums, input object type이어야 한다.

**스칼라 타입 종류**

- `Int`: 부호가 있는 32비트 정수.
- `Float`: 부호가 있는 부동소수점 값.
- `String`: UTF-8 문자열.
- `Boolean`: `true` 또는 `false`.
- `ID`: 기본적으로는 string이지만 고유 식별자를 나타내기 위해 쓰인다.

**enums 예제 (얄코 강의 55분을 참고하면 사용법을 알기 좋아요!)**

```jsx
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}

// 다음과 같은 enum을 선언한 후 type의 자료형에 Episode를 넣어주면 된다.
```

input object type은 사용자가 지정한 type이라고 생각하면 된다.

### Directives(지시어)

지시어를 통해 쿼리의 구조와 형태를 동적으로 변경할 수도 있다.

```jsx
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}

// Variables
{
  "episode": "JEDI",
  "withFriends": false
}
```

```jsx
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

- `@include(if: Boolean)`: 인자가 `true` 인 경우에만 이 필드를 결과에 포함합니다.
- `@skip(if: Boolean)` 인자가 `true` 이면 이 필드를 건너뜁니다.

위와 같이 지시어를 통해 정해진 쿼리에서 원하는 결과를 받아오는 것을 볼 수 있다.

### Mutation

GraphQL에서 Mutation은 GET 요청을 제외한 나머지 요청들을 뜻한다.

```jsx
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}

// Variables
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

```jsx
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```

createReivew 필드가 새로 생성된 리뷰의 stars와 commentary를 반환하는 것을 볼 수 있다.

뮤테이션은 쿼리와 마찬가지로 여러 필드를 포함할 수 있는데 `쿼리 필드는 병렬로 실행되는 반면 뮤테이션 필드는 하나씩 차례대로 실행된다.` 따라서 이전 요청이 다음 요청보다 무조건 먼저 완료된다.

## Schema & Type

### Type system

GraphQL은 스키마를 통해 유효성이 검사된 후 쿼리가 실행이 된다. 따라서 쿼리에 이를 정확하게 적어줘야 한다.

### Object types and fields

```jsx
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

- `Character` 는 *GraphQL 객체 타입* 입니다. 즉, 필드가 있는 타입입니다. 스키마의 대부분의 타입은 객체 타입입니다.
- `name` 과 `appearIn` 은 `Character` 타입의 *필드* 입니다. 즉 `name` 과 `appearIn` 은 GraphQL 쿼리의 `Character` 타입 어디서든 사용할 수 있는 필드입니다.
- `String` 은 내장된 *스칼라* 타입 중 하나입니다. 이는 스칼라 객체로 해석되는 타입이며 쿼리에서 하위 선택을 할 수 없습니다. 스칼라 타입은 나중에 자세히 다룰 것입니다.
- `String!` 은 필드가 *non-nullable* 임을 의미합니다. 즉, 이 필드를 쿼리할 때 GraphQL 서비스가 항상 값을 반환한다는 것을 의미합니다. 타입 언어에서는 이것을 느낌표로 나타냅니다.
- `[Episode]!` 는 `Episode` 객체의 *배열(array)* 을 나타냅니다. 또한 *non-nullable* 이기 때문에 `appearIn` 필드를 쿼리할 때 항상(0개 이상의 아이템을 가진) 배열을 기대할 수 있습니다.

### Argument

객체 타입의 모든 필드는 Argument를 가질 수 있다.

```jsx
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

모든 인수에는 이름이 있으며 위와 같이 기본값을 설정할 수 있다.

### Query Type & Mutation Type

스키마 대부분의 타입은 일반 객체 타입이지만 스키마 내에는 특수한 두 가지 타입이 있다.

```jsx
schema {
  query: Query
  mutation: Mutation
}
```

모든 GraphQL 서비스는 query 타입을 가지며 mutation 타입은 가질 수도 있고 가지지 않을 수도 있다. 이러한 타입은 GraphQL 쿼리의 entry point를 정의하기에 특별하다고 볼 수 있다.

### Lists and Non-Null

! 가 붙는다는 것은 null값을 반환받지 않겠다는 의미이다. 다음 표를 참고하면 좀 더 쉽게 이해할 수 있다.

![Apollo%20%E1%84%86%E1%85%B5%E1%86%BE%20GraphQL%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%201336662fa75e4368b58a97e5cbc75c4a/Untitled.png](Apollo%20%E1%84%86%E1%85%B5%E1%86%BE%20GraphQL%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%201336662fa75e4368b58a97e5cbc75c4a/Untitled.png)

### Interface (얄코 3-3을 참고하면 좋아요)

유사한 객체 타입을 만들기 위한 공통 필드 타입이다.

추상 타입으로 다른 타입에 implement 되기 위한 타입이다.

```jsx
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```

Character 를 구현한(implements) 모든 타입은 이러한 인자와 리턴 타입을 가진 정확한 필드를 가져야한다는 것을 의미한다.

### Union

한 배열 안에 한 가지 이상의 정보를 반환하고 싶을 때가 있다. 그럴 때 사용하면 좋다.

```jsx
union SearchResult = Human | Droid | Starship

// SearchResult는 값으로 Human, Droid, Starship을 받을 수 있다.
```

```jsx
// 위의 union을 반환받기 위해 아래와 같은 resolver를 작성 할 수 있다.
const resolvers = {
	Query: {
		SearchResult: () => {
			return [Human + Droid + Starship]
		}
	}
}
```

```jsx
// Query 예제
query {
  searchResults {
    ... on Human {
      id
      name
    }
    ... on Droid {
      id
      team
    }
		... on Starship {
			id
			comment
		}
  }
}
```

### Input Type

뮤테이션에서 유용하게 사용될 수 있는 Input Type은 말 그대로 입력값을 전달할 때 사용된다.

```jsx
input ReviewInput {
  stars: Int!
  commentary: String
}
```

```jsx
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}

VARIABLES
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

input type으로 ReviewInput을 선언한 후 mutation 쿼리에 이를 인자로 전달하는 예제이다.

## Apollo Client

---

Context API를 사용할 때 처럼 ApolloProvider로 React app을 감싸주면 하위 컴포넌트에서 언제든지 graphQL을 사용할 수 있다.

```jsx
import React from 'react';
import { render } from 'react-dom';

import { ApolloProvider } from '@apollo/client';

function App() {
  return (
    <ApolloProvider client={client}>
      <div>
        <h2>My first Apollo app 🚀</h2>
      </div>
    </ApolloProvider>
  );
}

render(<App />, document.getElementById('root'));
```

### useQuery hook

데이터를 요청할 때 useQuery를 사용할 수 있다.

useQuery에는 3가지 상태가 존재하는데 loading, error, data이다.

상태명 그대로 loading은 데이터를 요청하는 중일 때, error는 데이터 반환이 되었는데 에러가 있을 때를 나타내며 성공적으로 데이터가 반환되면 data에 값이 담기게 된다.

```jsx
// 다음과 같이 쿼리를 정의한 후
import { gql, useQuery } from '@apollo/client';

const GET_DOGS = gql`
  query GetDogs {
    dogs {
      id
      breed
    }
  }
`;

// 컴포넌트 내에서 useQuery hook을 사용하여 데이터를 받아올 수 있다.
const { loading, error, data } = useQuery(GET_DOGS);

```

```jsx
function Dogs({ onDogSelected }) {
  const { loading, error, data } = useQuery(GET_DOGS);

  if (loading) return 'Loading...';
  if (error) return `Error! ${error.message}`;

  return (
    <select name="dog" onChange={onDogSelected}>
      {data.dogs.map(dog => (
        <option key={dog.id} value={dog.breed}>
          {dog.breed}
        </option>
      ))}
    </select>
  );
}
// 컴포넌트 내에서 useQuery를 사용한 예제
// 각 상태에 따라 어떤 View를 보여줄 건지 쉽게 코드를 짤 수 있다.
```

Apollo Client에서는 서버로부터 데이터를 받아오면 자동으로 결과를 local에 caching한다. 따라서 같은 요청이 발생했을 때 캐시를 통해 빠르게 결과를 가져올 수 있다.

### useMutation hook

useMutation hook은 mutate function과 뮤테이션 실행에 따른 상태를 나타내는 오브젝트 필드를 갖는 tuple을 반환한다.

mutation function은 mutation을 실행시키는 함수이다. 반환 받은 값을 통해 언제든지 mutation을 발생시킬 수 있다.

useMutation hook 자체가 mutation을 발생시키는 것이 아니며 mutation function을 활용해 우리가 코드를 짜야 한다.

mutation 상태 객체에는 mutation function이 불려졌는지, mutation의 결과가 현재 loading인지를 나타내는 boolean 값들이 들어있다.

```jsx
import { gql, useMutation } from '@apollo/client';

const ADD_TODO = gql`
  mutation AddTodo($type: String!) {
    addTodo(type: $type) {
      id
      type
    }
  }
`;
```

```jsx
function AddTodo() {
  let input;
  const [addTodo, { data }] = useMutation(ADD_TODO);

  return (
    <div>
      <form
        onSubmit={e => {
          e.preventDefault();
          addTodo({ variables: { type: input.value } });
          input.value = '';
        }}
      >
        <input
          ref={node => {
            input = node;
          }}
        />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  );
}
```

### Updating single existing entity

Mutation이 발생하면 서버쪽 데이터가 바뀌게 되는데 이때 클라이언트쪽 캐시값도 갱신을 해줘야 한다. 만약 mutation이 single existing entity를 update 한다면 캐시도 자동으로 업데이트가 된다. 이렇게 하기 위해서는 수정된 필드의 값과 id를 함께 반환해야 한다. 

Apollo Client는 기본적으로 이러한 기능을 가지고 있다.

Apollo Client는 entities의 cache를 id로 하기 때문에 아이디와 대응되는 개체에 대해 어떤 값이 바뀌었는지 알 수 있다.

application의 UI 역시 cache의 변화에 따라 바로 바뀐다.

### Making all other cache updates

mutation이 발생했을 때 cache 값이 자동으로 갱신되지 않는데 useMutation에 update function을 포함시키는 것으로 해결할 수 있다.

백엔드에서 바뀐 정보와 클라이언트 cache를 동일하게 만드는 것이 update function이 목적이다.

```jsx
function AddTodo() {
  let input;
  const [addTodo] = useMutation(ADD_TODO, {
    update(cache, { data: { addTodo } }) {
      cache.modify({
        fields: {
          todos(existingTodos = []) {
            const newTodoRef = cache.writeFragment({
              data: addTodo,
              fragment: gql`
                fragment NewTodo on Todo {
                  id
                  type
                }
              `
            });
            return [...existingTodos, newTodoRef];
          }
        }
      });
    }
  });
```

update function은 cache 오브젝트를 전달한다. cache 오브젝트는 Apollo Client cache를 표현한다. 이 오브젝트는 readQuery, writeQuery, readFragment, writeFragment 그리고 modify API를 제공한다.

API 사용법 : [https://www.apollographql.com/docs/react/caching/cache-interaction/](https://www.apollographql.com/docs/react/caching/cache-interaction/)

### Loading과 error 상태 추적

useQuery와 마찬가지로 loading과 error 상태를 받아올 수 있다.

```jsx
function Todos() {
  const { loading: queryLoading, error: queryError, data } = useQuery(
    GET_TODOS,
  );

  const [
    updateTodo,
    { loading: mutationLoading, error: mutationError },
  ] = useMutation(UPDATE_TODO);

  if (queryLoading) return <p>Loading...</p>;
  if (queryError) return <p>Error :(</p>;

...
}
```

## 참고 자료

- 사용 이유 : [https://analogcoding.tistory.com/174](https://analogcoding.tistory.com/174)
- 얄코 강의 : [https://www.youtube.com/watch?v=9BIXcXHsj0A&feature=youtu.be](https://www.youtube.com/watch?v=9BIXcXHsj0A&feature=youtu.be)
- 스키마 스티칭 : [https://www.apollographql.com/blog/graphql-schema-stitching-8af23354ac37/](https://www.apollographql.com/blog/graphql-schema-stitching-8af23354ac37/)
- GraphQL 공식문서 : [https://graphql.org](https://graphql.org/learn)/
- GraphQL 한글 문서 : [https://graphql-kr.github.io/learn](https://graphql-kr.github.io/learn)
- apollographql 공식문서 : [https://www.apollographql.com](https://www.apollographql.com/docs/react/get-started/)/
- apollo API 문서 : [https://www.apollographql.com/docs/react/api/react/hooks/](https://www.apollographql.com/docs/react/api/react/hooks/)
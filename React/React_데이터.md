## React로 데이터 다루기

### useState를 쓰는 이유

예시로 살펴보자. <br />
잘못된 ex.

```jsx
// ❌ 이렇게 하면 안 됨
function Counter() {
  let count = 0; // 일반 변수

  const handleClick = () => {
    count++; // 값은 바뀌지만
    console.log(count); // 콘솔에는 찍히지만 화면은 그대로
  };

  return (
    <div>
      <p>카운트: {count}</p> {/* 항상 0 */}
      <button onClick={handleClick}>증가</button>
    </div>
  );
}
```

이렇게 하면, <br />

1. 변수값이 바뀌어도 컴포넌트가 다시 렌더링되지 않음.
2. 함수가 다시 실행될 때마다 변수가 초기값으로 리셋됨 (let count = 0;이 계속 실행됨)

근데, useState로 하면,

```jsx
const [count, setCount] = useState(0);
```

1. 상태 보존

   - 컴포넌트가 리렌더링되어도 값이 유지됨
   - React가 내부적으로 상태를 기억하고 관리

2. 자동 리렌더링

   - setState 함수를 호출하면 컴포넌트가 자동으로 다시 렌더링

3. 비동기 업데이트
   - 상태 변경이 비동기적으로 처리됨
   - 성능 최적화

#### `옳은 코드`

```jsx
function Counter() {
  import { useState } from "react";

  const [count, setCount] = useState(0);

  const handleClick = () => {
    //setCount(count + 1);
    //혹은
    setCount((prevCount) => prevCount + 1); //이게 더 권장됨
  };

  //아니면 이렇게도 된다.
  const handleClick = () => {
    setCount((prevCount) => {
      const newCount = prevCount + 1;
      console.log(newCount);
      return newCount;
    });
  };

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={handleClick}>증가</button>
    </div>
  );
}
```

### fetch / axios

✅ fetch란? <br />

- **브라우저 내장 함수**로, 서버에 HTTP 요청을 보낼 수 있음

- GET, POST, PUT, DELETE 등 다양한 요청을 보낼 수 있음. 기본값은 `GET`

- Promise를 반환하기 때문에 `.then()` 또는 `async/await`와 함께 사용.

✅ HTTP 요청

- 웹 브라우저(클라이언트)와 서버가 정보를 주고받을 때 쓰는 약속된 규칙(프로토콜)

- "나 이 정보 좀 줘" ! , "이거 저장해줘 !" , "이거 삭제해줘 !" 같은 명령을 서버에 보내는 행위

- 쉽게말해, 웹에서 서버한테 어떤 작업을 해달라고 보내는 편지. 그리고 그 편지를 `fetch`나 `axios` 같은 도구로 보냄

✅ `GET` `POST` `PUT` `PATCH` `DELETE` ?

- GET: 읽기 (Read)

  ```javascript
  // 사용자 목록 조회
  fetch("/api/users");
  ```

  - 서버에서 데이터를 읽어옴
  - 데이터를 변경하지 않음
  - URL에 파라미터 포함 가능

- POST: 생성 (Create)

  ```javascript
  // 새 사용자 생성
  fetch("/api/users", {
    method: "POST",
    body: JSON.stringify({ name: "홍길동", email: "hong@email.com" }),
  });
  ```

  - 서버에 새로운 데이터 추가
  - 요청 본문(body)에 데이터 포함

- PUT: 수정 (Update)

  ex. 해당 아이디 수정 <br />

  ```javascript
  // 사용자 정보 전체 업데이트
  fetch("/api/users/1", {
    method: "PUT",
    body: JSON.stringify({ name: "김철수", email: "kim@email.com" }),
  });
  ```

  - 특정 리소스를 완전히 교체
  - 전체 데이터를 새로 덮어씀

- PATCH: 부분 수정

  ```javascript
  fetch("/api/users/1", {
    method: "PATCH",
    body: JSON.stringify({
      name: "김철수",
    }),
  });
  ```

  이렇게 하면 이름만 수정된다.

- DELETE: 삭제 (Delete)

  ex. 해당 아이디 삭제 <br />

  ```javascript
  // 사용자 삭제
  fetch("/api/users/1", {
    method: "DELETE",
  });
  ```

  - 특정 리소스를 제거

✅ async/await란 ? <br />

- Promise를 더 읽기 쉬운 문법으로 다루기 위한 키워드.

- await는 비동기 작업이 끝날 때까지 기다림.

- async 함수 내부에서만 await 사용 가능.

ex.

```jsx
// api.js 파일에서
export async function getReviews() {
  try {
    const response = await fetch("https://example.com/api/reviews");

    if (!response.ok) {
      throw new Error(`서버 오류: ${response.status}`);
    }

    const data = await response.json(); //json 형식으로 파싱
    return data;
  } catch (error) {
    console.error("리뷰를 불러오는 중 오류 발생:", error);
    return { reviews: [] }; // fallback 데이터
  }
}
```

#### `response.json()` 이란 ?

- response는 서버에서 온 응답(response)객체.
- `response.json()`은 응답 본문(body)을 JSON (JavaScript Object Notation) 형식으로 변환해주는 메서드.
- 즉, 서버는 일반 텍스트 형태로 데이터를 보내기 때문에, 우리가 바로 객체처럼 쓰려면 `json()`으로 **파싱(해석)** 해야 함.

=> `response.json()` = "서버 응답을 JS 객체로 바꿔줘 ! "

#### `fallback 데이터`란 ?

- 네트워크 오류 등으로 실제 데이터를 못 받아올 경우, 대신 제공하는 기본값
- 데이터 없을 때, 최소한의 형태라도 보여주자! 라는 개념.

왜 씀 ? -> 예를 들어, 리뷰가 하나도 안 떠 있으면 화면이 깨지니까 빈 배열이라도 반환해서 렌더링이 되게 하려는 의도. <br />

`getReviews() 사용할 때`

```jsx
// App.jsx 또는 사용하고 싶은 곳
import { useEffect, useState } from "react";
import { getReviews } from "./api";

function App() {
  const [items, setItems] = useState([]);

  useEffect(() => {
    async function fetchData() {
      const { reviews } = await getReviews(); // 구조 분해 할당 (Destructuring), 서버가 어떤 형태로 오느냐에 따라 2가지형태로 받을 수 있음.
      setItems(reviews);
    }

    fetchData();
  }, []);

  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.content}</li>
      ))}
    </ul>
  );
}
```

1. 서버에서 data에 reviews 속성이 있는 경우

```json
{
  "reviews": [
    { "id": 1, "content": "좋아요" },
    { "id": 2, "content": "별로예요" }
  ],
  "total": 2
}
```

```jsx
const { reviews } = await getReviews();
```

2. 서버에서 바로 배열이 오는 경우

```json
[
  { "id": 1, "content": "좋아요" },
  { "id": 2, "content": "별로예요" }
]
```

```jsx
const [items, setItems] = useState([]);

const data = await getReviews(); // 전체 데이터 받기
setItems(data); // 바로 사용
```

⚠️ 주의할 점 <br />

- `await response.json()`을 빼먹고 그냥 `response.json()`만 쓰면 `Promise` 객체가 반환돼서 원하는 데이터가 아님. <br />
  = `await`을 붙여줘야 "결과 나올 떄까지 기다렸다가 꺼내줘" 라는 뜻이 된다.

✅ 올바른 사용 (✔️ await 사용) <br />

```jsx
async function fetchData() {
  const response = await fetch("https://example.com/api/data");
  const data = await response.json(); // JSON 데이터를 실제로 꺼내옴
  console.log(data); // 👉 실제 데이터 출력됨 (예: { name: "양씨" })
}
```

❌ 잘못된 사용 (await 빠짐) <br />

```jsx
async function fetchData() {
  const response = await fetch("https://example.com/api/data");
  const data = response.json(); // ❌ 아직 결과가 안 나왔는데 data에 Promise 객체가 담김
  console.log(data); // 👉 Promise { <pending> } 이렇게 나옴
}
```

- `await`은 반드시 `async` 함수 안에서만 사용 가능.

- `useEffect` 안에서 직접 `async` 함수 선언 불가 → 내부에 함수를 만들어서 호출해야 함.

```jsx
useEffect(async () => {}); // 이렇게 말고,

useEffect(() => {
  async function fetchData() {
    const res = await fetch("https://example.com/api");
    const data = await res.json();
    console.log(data);
  }

  fetchData(); // 이렇게 호출해야 함
}, []);
```

#### fetch의 POST 요청 예시.

```jsx
async function postData() {
  const response = await fetch("https://example.com/api/send", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ name: "홍길동", age: 30 }),
  });

  const result = await response.json();
  return result;
}
```

✅ headers란?

- HTTP 요청/응답의 부가 정보를 담는 공간.

- 예시:

  - Content-Type: 서버에게 "보내는 데이터의 형식"을 알려줌

  - Authorization: 토큰 인증 등

✅ Content-Type: application/json

- "보내는 데이터가 JSON 형식이에요!" 라고 서버에 알려줌

- 만약 이걸 안 넣으면 서버가 데이터를 어떻게 읽을지 몰라서 에러 날 수도 있음

✅ body + JSON.stringify?

```js
body: JSON.stringify({ name: "홍길동", age: 30 }),
```

- `body`: 실제로 서버에 전달할 데이터 본문

- `JSON.stringify()`: 자바스크립트 객체를 JSON 문자열로 바꿔주는 함수 <br />
  → 서버는 문자열 형태의 JSON을 받기 때문에 꼭 필요함!

```js
{ name: "홍길동" } → '{"name":"홍길동"}'
```

### `axios` 기본 사용

✅ axios는 무엇인가?

- HTTP 요청을 보내기 위한 외부 라이브러리

✅ axios 설치

```bash
npm install axios
```

#### `fetch` VS `axios`

| 기능                    | fetch                              | axios                     |
| ----------------------- | ---------------------------------- | ------------------------- |
| 기본 제공 여부          | 브라우저 내장                      | 외부 라이브러리 설치 필요 |
| JSON 자동 파싱          | ❌ 수동으로 `response.json()` 호출 | ✅ 자동으로 파싱          |
| 요청/응답 취소          | ❌ 불편함                          | ✅ `CancelToken`으로 가능 |
| 응답 코드 검사          | ❌ 수동으로 해야 함                | ✅ 자동으로 오류 처리     |
| 타임아웃 설정           | ❌ 없음                            | ✅ 가능                   |
| 인터셉터(요청 가로채기) | ❌ 없음                            | ✅ 가능                   |

✅ axios 사용 예시

```jsx
import axios from "axios";

export async function getReviews() {
  try {
    const response = await axios.get("https://example.com/api/reviews");
    return response.data; // 자동 JSON 파싱됨
  } catch (error) {
    console.error("axios 오류:", error);
    return { reviews: [] }; // fallback
  }
}
```

#### axios 실전 예시

프로젝트 규모가 커지면, `axios` 설정을 요청마다 반복하지 않도록 공통 인스턴스를 만들어 쓰는게 일반적임. <br />
`axios.create()` 로 공통 설정해서 코드 간결화함.

```jsx
// 📁 api/instance.js
import axios from "axios";

const api = axios.create({
  baseURL: "https://example.com/api",
  timeout: 5000,
  headers: {
    "Content-Type": "application/json",
  },
});

export default api;
```

```jsx
// 📁 api/review.js
import api from "./instance";

export async function getReviews() {
  const response = await api.get("/reviews"); // baseURL 자동 적용됨
  return response.data;
}
```

#### axios POST 요청

```jsx
import axios from "axios";

export async function sendReview(review) {
  try {
    const response = await axios.post(
      "https://example.com/api/reviews",
      review
    );
    return response.data;
  } catch (error) {
    console.error("POST 실패:", error);
    return null;
  }
}
```

#### HTTP Status Code 정리

| 분류                     | 상태코드                      | 이름           | 의미 및 실무 활용                             |
| ------------------------ | ----------------------------- | -------------- | --------------------------------------------- |
| ✅ 성공 (2xx)            | **200 OK**                    | 성공           | 요청 성공. 가장 일반적인 응답                 |
|                          | **201 Created**               | 생성됨         | POST 요청으로 리소스 생성 성공                |
|                          | **204 No Content**            | 콘텐츠 없음    | 성공했지만 응답 본문이 없음 (ex. DELETE)      |
| 🚦 리다이렉트 (3xx)      | **301 Moved Permanently**     | 영구 이동      | 주소가 영구 변경됨. SEO에도 영향              |
|                          | **302 Found**                 | 임시 이동      | 다른 위치로 임시 이동. 로그인 후 리디렉션 등  |
|                          | **304 Not Modified**          | 변경 없음      | 캐시된 데이터가 그대로일 때 사용. 성능 최적화 |
| ❌ 클라이언트 오류 (4xx) | **400 Bad Request**           | 잘못된 요청    | 파라미터 누락, 형식 오류 등                   |
|                          | **401 Unauthorized**          | 인증 필요      | 로그인 안 됐거나 토큰 없음                    |
|                          | **403 Forbidden**             | 금지됨         | 권한은 있지만 접근 불가                       |
|                          | **404 Not Found**             | 없음           | 리소스나 경로가 존재하지 않음                 |
| 🔥 서버 오류 (5xx)       | **500 Internal Server Error** | 내부 서버 오류 | 서버 내부 문제. 개발자 확인 필요              |

- 200, 201, 204 → 정상 작동 흐름 (조회/생성/삭제)

- 301, 302, 304 → 페이지 이동/캐시 관련

- 400, 401, 403, 404 → 클라이언트 쪽 실수나 인증 문제

- 500 → 서버 터짐

### 논리 연산자 `!` (NOT) 정리

ex. <br />

```jsx
const [show, setShow] = useState(false);
const handleClick = () => setShow(!show);
```

- !는 논리 NOT 연산자
- 값을 Boolean으로 변환한 뒤, 그 값을 반전시킴

```js
!false → true
!true → false
!"" → true  // 빈 문자열은 falsy
!"hello" → false
```

❗ 잘못 이해하면 안 되는 포인트 <br />
❌ "불린값이니까 뒤집는 거 아냐?" <br />
✅ "아니, 먼저 불린값으로 변환한 뒤에 뒤집는 거야!" <br />

```js
const handleClick = () => setShow(!show);
// -> show가 true면 false로, false면 true로 바뀜
```

#### 🔁 같은 동작을 하는 다른 방식들

```js
// 삼항 연산자
const handleClick = () => setShow(show ? false : true);

// if문
const handleClick = () => {
  if (show) {
    setShow(false);
  } else {
    setShow(true);
  }
};
```

#### 실전 예시

```js
const name = "";
if (!name) {
  console.log("이름을 입력해주세요"); // 빈 문자열 → falsy → true로 반전 → 실행됨
}

const items = [];
if (!items.length) {
  console.log("아이템이 없습니다"); // items.length = 0 → falsy → true → 실행됨
}

const user = null;
if (!user) {
  console.log("로그인이 필요합니다"); // null → falsy → true → 실행됨
}
```

#### 조건부 렌더링에서의 `!` / `truthy-falsy` 활용

❌ 렌더링 되지 않는 값들

- `null`

- `undefined`

- `false`

- `true`

✔ 이걸 이용하면 조건부 렌더링이 매우 깔끔해짐 <br />

```jsx
{
  isLoggedIn && <WelcomeMessage />;
} // 로그인 되어있을 때만 렌더링
{
  user ? <Profile /> : null;
} // user가 있을 때만 렌더링
```

#### 헷갈리는 케이스들

| JSX 표현 | truthy/falsy | 화면에 표시됨?  | 설명                           |
| -------- | ------------ | --------------- | ------------------------------ |
| `{0}`    | falsy        | ✅ `"0"` 보임   | 숫자는 렌더링 가능             |
| `{NaN}`  | falsy        | ✅ `"NaN"` 보임 | NaN도 문자열처럼 처리됨        |
| `{" "}`  | truthy       | ✅ (공백 표시)  | 공백이지만 표시됨              |
| `{[]}`   | truthy       | ⛔ (표시 안 됨) | 내용이 없어서 출력 결과가 없음 |

이게 왜 헷갈림 ? <br />

1. `truthy/falsy`판단과 JSX 렌더링 여부가 완전히 일치하지 않기 때문

   - 예를 들어, `0`은 falsy지만 `{0}`은 화면에 `"0"`으로 나온다 → 직관적으로 이해 안 됨
   - `[]`는 truthy지만 `{[]}`는 아무것도 안 나온다 → 어라? 보여야 되는 거 아닌가?

2. `&&` 같은 조건부 렌더링에서 falsy 값이 안 보이길 기대했는데 출력됨

```
{
  product.stock && <span>재고: {product.stock}개</span>
}
```

- `product.stock = 0`일 경우 → `0 && <span>...</span>` → `0`이 반환됨. <br />
  falsy 값이지만, JSX에서 `0`은 숫자니까 화면에 찍히고, `<span>` 자체는 렌더링 되지 않음 <br />
  -> 화면에 `0` 하나만 뜨는 이상한 결과가 생김

#### `해결`

"falsy인데 보여야 하는 값은 필터링하고, truthy인데 렌더링 안 되는 건 보완해주기" <br />

### 1. falsy인데 보여야 하는 값은 필터링하기.

예: `product.stock = 0`일 때 `"재고: 0개"`가 안 보이는 문제

```
{product.stock && <span>재고: {product.stock}개</span>}
```

→ `<span>` 자체가 렌더링되지 않아서, "재고: 0개"가 아닌 그냥 `0`만 화면에 찍힘<br />
→ 이는 사용자 입장에서 매우 어색한 결과 (디자인 깨짐, 문맥 없음)

해결 1 : 삼항연산자 쓰기

```
{
  product.stock === 0 ? (
    <span>재고: 0개</span>
  ) : product.stock ? (
    <span>재고: {product.stock}개</span>
  ) : null;
}
```

해결 2 : 타입 명시

```
{typeof product.stock === "number" && <p>재고: {product.stock}개</p>}
```

### 2. truthy인데 렌더링 안 되는 건 보완해주기

예: 빈 배열 `[]`

ex. <br />

```
{comments && comments.map((c) => <p>{c.text}</p>)}
```

-> `comments = []` 일 때, 조건은 truthy => 실행됨 <br />
-> 근데 `.map()` 결과도 `[]`니까 결국 아무것도 안보임 <br />

해결 : 내용유무 명시적으로 체크

```
{comments.length > 0 ? <CommentList /> : <p>댓글이 없습니다</p>}
```

### 이전 State 값을 참조하면서 State를 변경하는 경우, 무조건 콜백함수를 써야한다!

잘못되진 않았지만 위험할 수 있는 ex. <br />

```jsx
const [count, setCount] = useState(0);

const handleAddClick = async () => {
  await addCount(); // 비동기 작업 (시간이 걸림)
  setCount(count + 1); // ❌ 문제 발생 가능!
};
```

문제 :

- await addCount를 하는동안 count값이 다른곳에서 바뀔 수도 있음

해결 :

- 콜백함수를 쓰면 항상 최신 값을 보장받을 수 있다 !

#### 여기서 잠깐, 이게 왜 콜백함수임 ?

- React의 `setState` 함수에 인자로 함수를 넘기는 거니까, 콜백함수다. <br />

[콜백함수 개념 보러가기](../JavaScript/비동기-자바스크립트.md#콜백-callback)

```jsx
const [count, setCount] = useState(0);

const handleAddClick = async () => {
  await addCount();
  setCount((prevCount) => prevCount + 1); // React가 최신 state를 prevCount에 넣어서 함수로 넘겨준다.
};
```

`다양한 예시들`

```jsx
// ❌ 위험할 수 있음
setCount(count + 1);

// ✅ 항상 안전함
setCount((prevCount) => prevCount + 1);

// ❌ 위험할 수 있음
setTodos([...todos, newTodo]);

// ✅ 항상 안전함
setTodos((prevTodos) => [...prevTodos, newTodo]);
```

### 입력 폼

`e.target`이란 ?
이벤트가 발생한 HTML 요소 <br />
예를 들어, `input`, `select`, `textarea` 등에서 사용자가 입력/선택한 값을 가져올 때 씀. <br />

- `e.target` -> 이벤트가 발생한 요소 자체 (`<input>` , `<select>`)
- `e.target.value` -> 그 요소에 입력된 현재 값

ex. 폼 입력 값 가져오기

```jsx
function handleChange(e) {
  console.log("입력된 값:", e.target.value); //input 필드에 있는 값을 출력함
}

<input type="text" onChange={handleChange} />;
```

### 제어 컴포넌트 VS 비제어 컴포넌트

| 구분           | 제어 컴포넌트                           | 비제어 컴포넌트                             |
| -------------- | --------------------------------------- | ------------------------------------------- |
| 값 관리 주체   | React (state 사용)                      | DOM (Ref 사용)                              |
| 실시간 값 추적 | 가능 (렌더링 시마다 최신값 유지)        | 불가능 (필요할 때만 직접 접근)              |
| 사용 목적      | 대부분의 폼 입력 처리                   | 초기값만 설정하거나, 파일/포커스 제어 등    |
| 장점           | React 흐름 내에서 일관된 상태 관리 가능 | 렌더링 없이 값 접근 가능 (성능적 유리함)    |
| 단점           | 잦은 렌더링 발생                        | React 외부에서 값 제어, 예측 어려울 수 있음 |

ex. 제어 컴포넌트

```jsx
function ControlledInput() {
  const [value, setValue] = useState("");

  return (
    <input
      value={value} // React state로 값 제어
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

ex. 비제어 컴포넌트

```jsx
import { useRef } from "react";

function UncontrolledInput() {
  const inputRef = useRef();

  const handleSubmit = () => {
    console.log(inputRef.current.value); // DOM에서 직접 값 가져오기
  };

  return (
    <div>
      <input ref={inputRef} /> {/* React가 값 관리 안함 */}
      <button onClick={handleSubmit}>제출</button>
    </div>
  );
}
```

#### `useRef` ?

- DOM 요소에 직접 접근하거나, 값을 저장하는 데 사용하는 Hook
- `ref.current`를 통해 해당 DOM 노드에 접근 가능
- 렌더링과 무관하므로, 값이 바뀌어도 컴포넌트가 재렌더링되지 않음

```jsx
import { useRef, useEffect } from "react";

function Example() {
  const inputRef = useRef();

  useEffect(() => {
    inputRef.current.focus(); // 마운트 시 input에 자동 포커스
  }, []);

  return <input ref={inputRef} />;
}
```

### 사이드 이펙트 (부작용)

: 어떤 함수나 연산의 수행 결과로 시스템의 상태가 예상치 못하게 변경되는 현상. <br />
그러니까, 함수를 실행하는데, 외부의 상태가 바뀌는 함수를 보고 "**사이드 이펙트가 있다**"고 한다.

`useEffect`는 컴포넌트 렌더링 이후, **사이드 이펙트를 수행하거나 정리할 때 사용하는 Hook**이다.  
대표적으로 API 호출, DOM 노드 직접 변경 등에 쓰이고, '**동기화**'에 쓰면 유용한 경우가 많음.
(외부의 데이터와 컴포넌트 안에 데이터를 일치시키고 싶을 때 쓴다) <br />

#### `여러 예시들`

1. 페이지 정보 변경

```jsx
useEffect(() => {
  document.title = title; // 페이지 데이터를 변경
}, [title]);
```

2. 네트워크 요청

```jsx
useEffect(() => {
  fetch("https://example.com/data") // 외부로 네트워크 리퀘스트
    .then((response) => response.json())
    .then((body) => setData(body));
}, []);
```

3. 데이터 저장

```jsx
useEffect(() => {
  localStorage.setItem("theme", theme); // 로컬 스토리지에 테마 정보를 저장
}, [theme]);
```

주의할 점 : 사이드 이펙트를 만들고, 정리함수 잊지않기 ! <br />
콜백을 한 번 실행했으면, 정리 함수도 반드시 한 번 실행된다고 생각하자. <br />

정리함수의 일반적인 형태는 <br/>

```jsx
useEffect(() => {
  // 사이드 이펙트 실행

  return () => {
    // cleanup: 컴포넌트가 언마운트될 때 실행됨
  };
}, [의존성]);
```

정리함수에서의 주의할 점은 함수 자체를 리턴해야 한다는 것. <br />

```jsx
return clearInterval(id); // XX
```

이렇게 리턴하면 결과값을 리턴하는 것이기 때문에 잘못된 방법임. <br />

```jsx
return () => clearInterval(id); // OO
```

요렇게 리턴하자 ! <br />

`정리함수 ex.`

```jsx
import { useEffect, useState } from "react";

function Timer() {
  const [second, setSecond] = useState(0);

  useEffect(() => {
    const timerId = setInterval(() => {
      console.log("타이머 실행중 ... ");
      setSecond((prevSecond) => prevSecond + 1);
    }, 1000);
    console.log("타이머 시작 🏁");

    return () => {
      clearInterval(timerId); //setInterval 이 만든 "반복 작업"을 정지시킬 때 쓰는함수
      console.log("타이머 멈춤 ✋");
    };
  }, []);

  return <div>{second}</div>;
}

function App() {
  const [show, setShow] = useState(false);

  const handleShowClick = () => setShow(true);
  const handleHideClick = () => setShow(false);

  return (
    <div>
      {show && <Timer />}
      <button onClick={handleShowClick}>보이기</button>
      <button onClick={handleHideClick}>감추기</button>
    </div>
  );
}

export default App;
```

### 자주 보이는 경고 : exhaustive-deps

: 디펜던시를 빠짐없이 넣어주세요 ! 라는 경고 메시지임. <br />
리액트에서는 `Prop`이나 `State`와 관련된 값은 되도록이면 빠짐없이 디펜던시에 추가해서
항상 최신 값으로 useEffect 나 useCallback 을 사용하도록 권장하고 있음.

ex.

```jsx
import { useEffect, useState } from "react";

function App() {
  const [count, setCount] = useState(0);
  const [num, setNum] = useState(0);

  const addCount = () => {
    setCount((c) => c + 1);
    console.log(`num: ${num}`);
  };

  const addNum = () => setNum((n) => n + 1);

  useEffect(() => {
    console.log("timer start");
    const timerId = setInterval(() => {
      addCount();
    }, 1000);

    return () => {
      clearInterval(timerId);
      console.log("timer end");
    };
  }, []);

  return (
    <div>
      <button onClick={addCount}>count: {count}</button>
      <button onClick={addNum}>num: {num}</button>
    </div>
  );
}

export default App;
```

코드설명 : <br />

- 아래 코드는 num 버튼을 누르면 num 스테이트 값이 증가되고,

- count 버튼을 누르면 count 스테이트 값을 증가시키는 컴포넌트임.

- 이때 count 스테이트 값을 증가시키면서 콘솔에는 num 스테이트 값을 출력함.

- useEffect Hook에서는 1초마다 addCount 함수를 실행하는 타이머를 실행.

이 예시에서 잘못된 점 : useEffect가 처음 한 번만 실행되고, 그때 만들어진 addCount 함수는 num = 0일 때의 값을 계속 기억하기 때문에, num이 바뀌어도 옛날 함수를 계속 사용함. (num이 0만 나옴)

#### (잘못된)해결방법1 : 의존성 배열에 함수 추가

```jsx
useEffect(() => {
  console.log("timer start");
  const timerId = setInterval(() => {
    addCount(); // 새로운 addCount 함수 사용
  }, 1000);

  return () => {
    clearInterval(timerId);
    console.log("timer end");
  };
}, [addCount]); // 👈 addCount가 바뀔 때마다 실행
```

잘못된 점 : React에서는 컴포넌트가 렌더링 될 때마다 함수 전체가 다시 실행됨. 그러면, addCount에서 setCount 호출할 때마다 값이 바뀌니까, addCount는 매번 새로 만들어지고, 타이머가 계속 재시작됨. <br />
렌더링될때마다 새 함수 생성 -> 참조(주소)가 달라짐 -> 계속 다른 함수로 인식하니까 useEffect는 '함수가 바뀌었네?' 하고 계속 이펙트 재실행 -> 새 타이머 계속 시작. <br />

```memo
타이머 실행 → setCount → 리렌더링 → 새 addCount → 새 타이머 → 타이머 실행 → ... 계속 반복
```

#### 옳은 해결방법2 : useCallback으로 함수 기억시키기.<br />

`useCallback` ? : 함수를 매번 새로 만들지 않고, 재사용하게 해주는 Hook

```jsx
import { useCallback, useEffect, useState } from "react";

function App() {
  const [count, setCount] = useState(0);
  const [num, setNum] = useState(0);

  const addCount = useCallback(() => {
    setCount((c) => c + 1);
    console.log(`num: ${num}`);
  }, [num]);

  const addNum = () => setNum((n) => n + 1);

  useEffect(() => {
    console.log("timer start");
    const timerId = setInterval(() => {
      addCount();
    }, 1000);

    return () => {
      clearInterval(timerId);
      console.log("timer end");
    };
  }, [addCount]);

  return (
    <div>
      <button onClick={addCount}>count: {count}</button>
      <button onClick={addNum}>num: {num}</button>
    </div>
  );
}

export default App;
```

이렇게 `useCallback`을 사용하면 함수를 매번 생성하는 게 아니라, 리액트에다 함수를 기억해 둘 수 있고, 이때 리액트는 `useCallback`의 디펜던시 값이 바뀔 때만 함수를 새로 만든다. (여기서는 num 값이 바뀔 때만 새로 실행됨)

#### 권장 해결책3 : 파라미터 사용하기

: 함수 자체를 "순수하게" 만들어서 외부 상태에 의존하지 않게 하자.

```jsx
import { useEffect, useState } from "react";

function App() {
  const [count, setCount] = useState(0);
  const [num, setNum] = useState(0);

  // ✅ 순수 함수: 외부 상태에 의존하지 않음

  const addCount = (log) => {
    setCount((c) => c + 1);
    console.log(log);
  };

  const addNum = () => setNum((n) => n + 1);

  useEffect(() => {
    console.log("timer start");
    const timerId = setInterval(() => {
      addCount(`num ${num}`); // 🎯 현재 num 값을 직접 전달
    }, 1000);

    return () => {
      clearInterval(timerId);
      console.log("timer end");
    };
  }, [num]);

  return (
    <div>
      <button onClick={addCount}>count: {count}</button>
      <button onClick={addNum}>num: {num}</button>
    </div>
  );
}

export default App;
```

왜 더 좋은 방법인가 ? <br />

- `useCallback` 방식은 함수 자체는 메모이징되지만, 언제 재생성되는지 추적하려면 `addCount`의 의존성 배열을 봐야 하기 때문에 맥락이 한 단계 감춰져 있음 <br />
  반면, 파라미터 방식은 num이라는 값을 직접 이펙트에 넣어서 '언제 동작할지'가 더 명시적으로 드러남.

- 함수 재사용성이 좋음

- 테스트하기 쉬움

근데, 전달해야하는 파라미터가 여러개라면 ..? -> 그럼 `useCallback`이 더 옳은 방법일 수 있다.

⚖️ `파라미터 전달 방식` vs `useCallback` 언제 쓸까?

✅ 파라미터 전달 방식이 적절한 경우 <br />

| 조건                            | 설명                                        |
| ------------------------------- | ------------------------------------------- |
| 외부 상태값이 1\~2개 정도       | 함수에서 사용하는 값이 적을 때              |
| 파라미터로 넘기는 게 자연스러움 | ex. `addUser(name, age)` 처럼 직관적인 경우 |
| 함수 목적이 명확하고 단순함     | 로직이 짧고, 사이드 이펙트가 복잡하지 않음  |

장점: 추적이 명확하고, 순수 함수로 만들 수 있어 테스트가 쉬움 <br />
단점: 파라미터가 많아지면 오히려 가독성 떨어짐 <br />

✅ useCallback이 적합한 경우 <br />

| 조건                                          | 설명                                            |
| --------------------------------------------- | ----------------------------------------------- |
| 외부 상태를 여러 개 사용                      | ex. `count`, `num`, `isActive`, `theme` 등      |
| 함수에 전달해야 할 값이 많음                  | 파라미터가 많아지면 재사용성/호출 가독성 떨어짐 |
| 함수 로직이 복잡하거나 다른 컴포넌트에 전달됨 | 이벤트 핸들러, props로 내려보낼 때 등           |

장점: 외부 상태를 캡슐화하여 전달할 수 있고, prop 최적화에도 유리함 <br />
단점: 의존성 배열을 신경 써야 하며, 메모이제이션 오히려 과할 수도 있음 <br />

### Hook

규칙 <br />

1. 반드시 리액트 컴포넌트 함수 안에서 사용해야 함
2. 컴포넌트 함수의 최상위에서만 사용 (조건문, 반복문안에서 못씀)

#### `useState`

이전 State 참조해서 state 변경하기 <br />

1. 함수형 업데이트

```jsx
const [count, setCount] = useState(0);

// 이전 값을 참조해서 업데이트
setCount((prevCount) => prevCount + 1);
setCount((prev) => prev * 2);
```

2. 객체 state 변경

```jsx
const [user, setUser] = useState({ name: "", age: 0 });

// 이전 객체를 spread하고 특정 필드만 변경
setUser((prevUser) => ({
  ...prevUser,
  age: prevUser.age + 1,
}));
```

3. 배열 state 변경

```jsx
const [items, setItems] = useState([
  { id: 1, completed: false },
  { id: 2, completed: false },
]);
// 이전 배열에 새 항목 추가
setItems((prevItems) => [...prevItems, newItem]);

const targetId = 2;

// 특정 인덱스 아이템 변경
setItems((prevItems) =>
  prevItems.map(
    (item) => (item.id === targetId ? { ...item, completed: true } : item) // item id를 같은거를 찾아서 같으면 completed를 true로 바꿔주고, 아니면 item 그대로 반환
  )
);
```

#### `Custom Hook`

`useAsync`

: 비동기 함수의 로딩, 에러 처리를 하는 데 사용할 수 있는 함수.
asyncFunction을 실행할 때, 로딩 상태(pending), 에러 상태(error)를 자동으로 관리해주는 훅

```jsx
function useAsync(asyncFunction) {
  const [pending, setPending] = useState(false);
  const [error, setError] = useState(null);

  const wrappedFunction = useCallback(
    async (...args) => {
      setPending(true);
      setError(null);
      try {
        return await asyncFunction(...args);
      } catch (error) {
        setError(error);
      } finally {
        setPending(false);
      }
    },
    [asyncFunction]
  );

  return [pending, error, wrappedFunction];
}
```

Hook 설명 <br />

`wrappedFunction(...args)` 구조

```js
const wrappedFunction = useCallback(
  async (...args) => {
    return await asyncFunction(...args);
  },
  [asyncFunction]
);
```

- `wrappedFunction`은 인자를 몇 개든 받을 수 있는 함수
- 받은 인자들은 전부 `args` 배열로 들어감
- 그걸 다시 `asyncFunction(...args)`로 넘기니까, 결국 원래 asyncFunction이 원하는 방식 그대로 호출됨.

예시 1: 인자 0개인 함수일 때

```js
async function fetchData() {
  // 아무 인자 없이 동작
}

const [pending, error, wrappedFunction] = useAsync(fetchData);

await wrappedFunction(); // ✅ 인자 없이 호출
```

→ 내부에서 이렇게 작동함:

```js
// 내부적으로
await fetchData();
```

예시 2: 인자 1개

```js
async function getUser(id) {
  return await fetch(`/users/${id}`);
}

const [pending, error, wrappedFunction] = useAsync(getUser);

await wrappedFunction(42); // ✅ id = 42
```

→ 내부적으로는:

```js
await getUser(42); // args = [42]
```

#### 사용할 때

```jsx
import { useState } from "react";
import useAsync from "./useAsync"; // 위 코드가 정의된 파일

async function fetchUser(id) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
  if (!res.ok) throw new Error("불러오기 실패");
  return res.json();
}

function UserInfo({ userId }) {
  const [user, setUser] = useState(null);

  const [pending, error, loadUser] = useAsync(fetchUser);

  const handleClick = async () => {
    const userData = await loadUser(userId);
    setUser(userData);
  };

  return (
    <div>
      <button onClick={handleClick}>유저 정보 불러오기</button>
      {pending && <p>로딩 중...</p>}
      {error && <p>에러 발생: {error.message}</p>}
      {user && <pre>{JSON.stringify(user, null, 2)}</pre>}
    </div>
  );
}
```

#### 왜 씀 ?

매번 비동기 로직 쓸 때마다

```js
try {
  setPending(true);
  ...
} catch (e) {
  setError(e);
} finally {
  setPending(false);
}
```

이걸 반복하기 너무 귀찮고, 코드가 지저분해짐.

#### `참고`

: 자주 사용하는 훅들은 앞에 use... 를 붙혀서 다른 개발자들이 Hook이라는 걸 알수 있게 해줘야한다.

### 리액트 Context

: 여러 곳에서 쓰이는 데이터를 내려주고 싶을 때가 있는데, 컴포넌트의 단계가 많다면 prop drilling이 일어남. 이걸 해결하기 위해 사용하는 기능. <br />
=> 여러 컴포넌트에서 공통으로 사용하는 데이터를 전역적으로 관리하는 방법

`prop drilling` ?

```jsx
function App() {
  const user = { name: "김철수" };
  return <Parent user={user} />;
}

function Parent({ user }) {
  return <Child user={user} />;
}

function Child({ user }) {
  return <div>안녕하세요, {user.name}님!</div>;
}
```

- `App → Parent → Child`로 `user`를 계속 props로 넘겨야 함 (불필요하게 거쳐야함)
- 이게 바로 **prop drilling(프로퍼티 뚫기)** 문제

✅ Context로 해결

```jsx
function App() {
  return (
    <UserContext.Provider value={{ name: "김철수" }}>
      <Parent />
    </UserContext.Provider>
  );
}
```

```jsx
function Child() {
  const user = useContext(UserContext);
  return <div>안녕하세요, {user.name}님!</div>;
}
```

- 중간 컴포넌트 `Parent`는 더 이상 props를 받을 필요 없음

- `Child`는 context를 통해 전역 데이터처럼 바로 가져다 씀!

#### `어떻게?`

: `createContext`라는 함수를 통해 만들기.

```jsx
import { createContext } from "react";

const LocaleContext = createContext("ko"); //이렇게 기본값을 넣어도되고 안넣어도 된다.
```

#### `범위?`

: `Context` 객체에 있는 `Provider` 라는 컴포넌트로 정해줄 수 있음. <br />
= ✅ Provider가 감싸고 있는 범위 안에서만 Context 사용 가능!

```jsx
import { createContext } from "react";

const LocaleContext = createContext("ko");

function App() {
  return (
    <div>
      ... 바깥의 컴포넌트에서는 LocaleContext 사용불가
      <LocaleContext.Provider value="en">
        ... Provider 안의 컴포넌트에서는 LocaleContext 사용가능
      </LocaleContext.Provider>
    </div>
  );
}
```

사용할 때는 `useContext`로 값 가져오기. <br />

실제 파일 구조 예시.

🔹 1단계: Context 생성

```jsx
// contexts/UserContext.js 에서
import { createContext } from "react";

export const UserContext = createContext();
```

`createContext()`는 “React에게 앞으로 우리가 이런 전역 상태를 관리할 거다”라고 알려주는 역할. <br />
아직은 실제 값이 없고, 그냥 껍데기만 만든 상황. <br />

🔹 2단계: Provider로 값 내려주기

```jsx
//App.js에서 (Provider 설정)
import { UserContext } from "./contexts/UserContext";

function App() {
  const user = { name: "김철수", age: 25 };

  return (
    <UserContext.Provider value={user}>
      <Parent />
    </UserContext.Provider>
  );
}
```

이 `Provider`가 "이 안에 있는 모든 컴포넌트들은 user라는 값을 context로 받아 쓸 수 있게 해준다"는 의미. <br />
즉, “범위”는 Provider가 감싸고 있는 부분으로 한정됨. <br />

🔹 3단계: 하위 컴포넌트에서 `useContext`로 값 꺼내기

```jsx
// components/Child.js에서 context사용
import { useContext } from "react";
import { UserContext } from "../contexts/UserContext";

function Child() {
  const user = useContext(UserContext);
  return <div>안녕하세요, {user.name}님!</div>;
}

export default Child;
```

이제 `Child` 컴포넌트는 직접 props로 아무것도 받지 않아도, `UserContext` 안에 있는 값을 꺼내 쓸 수 있음. <br />

🔹 4단계: 중간 컴포넌트(Parent)가 props 안 넘겨줘도 됨

```jsx
// Parent에서 뿌려줄 때
import Child from "./Child";

function Parent() {
  return <Child />; // user를 직접 props로 전달하지 않아도 됨!
}

export default Parent;
```

이게 바로 **prop drilling**을 막는 효과. <br />
중간 컴포넌트가 props를 전달할 필요 없이, `Child`가 직접 context에서 받아버림! <br />

🔹 5단계: 더 깔끔하게 커스텀 훅으로 만들기

```jsx
// contexts/UserContext.js

import { createContext, useContext } from "react";

// 1. Context 생성
const UserContext = createContext();

// 2. 커스텀 훅 생성
export const useUser = () => {
  const context = useContext(UserContext);

  // 3. 에러 처리 (Provider 밖에서 사용했을 때)
  if (context === undefined) {
    throw new Error("useUser는 UserProvider 안에서만 사용할 수 있습니다");
  }

  return context;
};

// 4. Provider 컴포넌트도 export
export const UserProvider = ({ children, value }) => {
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
};
```

- 여기서 `children`은 컴포넌트 사이에 들어가는 모든 내용

🔹 6단계: 커스텀 훅을 사용하는 컴포넌트 <br />

```jsx
//사용할 때
import { useUser } from "../contexts/UserContext";

function Profile() {
  const user = useUser(); // 간단! 에러 처리도 자동

  return <div>안녕하세요, {user.name}님</div>;
}
```

단, App.js에서 이렇게 설정되어 있어야함.

```jsx
function App() {
  return (
    <UserProvider value={{ name: "김철수", age: 25 }}>
      <Profile /> {/* ✅ useUser()에서 값 정상적으로 사용 가능 */}
    </UserProvider>
  );
}
```

#### 실무에서 자주 쓰이는 Context

1. Theme – 다크/라이트 모드 전역 관리

2. Language / Locale – 다국어(예: ko, en, zh) 설정

3. Auth / User – 로그인 상태나 사용자 정보

🌗 1. ThemeContext (다크/라이트 모드)

```jsx
// contexts/ThemeContext.js
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext();

export const useTheme = () => {
  const ctx = useContext(ThemeContext);
  if (!ctx)
    throw new Error("useTheme은 ThemeProvider 안에서만 사용해야 합니다");
  return ctx;
};

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState("light");
  const toggleTheme = () => setTheme((t) => (t === "light" ? "dark" : "light"));

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};
```

```jsx
// App.js
<ThemeProvider>
  <YourApp />
</ThemeProvider>
```

```jsx
// Header.js
const { theme, toggleTheme } = useTheme();
```

🌍 2. LocaleContext (언어 설정)

```jsx
// contexts/LocaleContext.js
import { createContext, useContext, useState } from "react";

const LocaleContext = createContext();

export const useLocale = () => {
  const ctx = useContext(LocaleContext);
  if (!ctx)
    throw new Error("useLocale은 LocaleProvider 안에서만 사용해야 합니다");
  return ctx;
};

export const LocaleProvider = ({ children }) => {
  const [locale, setLocale] = useState("ko");
  const switchLocale = (lang) => setLocale(lang);

  return (
    <LocaleContext.Provider value={{ locale, switchLocale }}>
      {children}
    </LocaleContext.Provider>
  );
};
```

```jsx
// App.js
<LocaleProvider>
  <YourApp />
</LocaleProvider>
```

```jsx
// Footer.js
const { locale, switchLocale } = useLocale();
```

🔐 3. AuthContext (로그인 정보, 인증 상태)

```jsx
// contexts/AuthContext.js
import { createContext, useContext, useState } from "react";

const AuthContext = createContext();

export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth는 AuthProvider 안에서만 사용하세요");
  return ctx;
};

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  const isAuthenticated = !!user;

  return (
    <AuthContext.Provider value={{ user, isAuthenticated, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};
```

```jsx
// App.js
<AuthProvider>
  <YourApp />
</AuthProvider>
```

```jsx
// Navigation.js
const { user, isAuthenticated, logout } = useAuth();
```

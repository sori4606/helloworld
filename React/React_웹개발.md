## React 웹 개발 시작하기

### 리액트 시작하기

1. Node.js 설치
2. Vite 또는 CRA (Create React App 사용)

ex.

```
// Vite
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev
```

```
//CRA
npx create-react-app my-react-app
cd my-react-app
npm start
```

🔧 CRA (Create React App)

- 옛날부터 쓰던 전통적인 방법
- 리액트 공식 팀에서 만든 초기 세팅 툴
- 웹팩(Webpack) 기반

- 장점:

  - 셋업이 쉬움 (npx create-react-app)

  - 리액트 초보에게 익숙한 환경 제공

- 단점:

  - 빌드 느림

  - 커스터마이징 불편 (webpack eject 하지 않으면 내부 설정 못 건드림)

  - 꽤 무거움

⚡ Vite

- 차세대 프론트엔드 빌드 툴
- 빠른 dev server (에디터에서 저장하면 바로 브라우저에 반영)
- Rollup 기반 (CRA는 Webpack 기반)

- 장점:

  - 매우빠름

  - 설정이 유연함

  - 최근엔 리액트 프로젝트의 대세

- 단점:

  - 아주 초기엔 CRA보다 정보가 적었지만, 요즘은 정보 넘침

### `Webpack` ? `Rollup` ?

Webpack

- CRA에서 사용하는 번들러 <br />
  (💥 번들러란?
  👉 우리가 개발하는 소스코드는 여러 .js, .css, .svg, .ts, .jsx 등으로 나뉘어져 있는데,
  브라우저는 이런 파일들을 효율적으로 다루기 힘들기 때문에,
  "번들러"가 이 모든 파일을 하나(or 몇 개)의 파일로 묶어줌. (= "번들" 생성))

- 가장 오래되고 널리 쓰임
- 처음 빌드 속도가 느림

Rollup

- Vite에서 사용하는 빌드 기반 도구
- 빌드 결과가 작고 빠름

### JSX

: 자바스크립트의 확장 문법. HTML과 비슷하고, 훨씬 더 편리하게 화면에 나타낼 코드를 작성할 수 있다. <br />

특징 <br />

### 1. 속성명은 CamelCase로 작성

ex) onClick, onBlur, onFocus ... <br />

```jsx
import ReactDOM from 'react-dom';

ReactDOM.render(
  <button onClick= ... >클릭!</button>,
  document.getElementById('root')
);
```

단, 비표준 속성을 사용할 때는 HTML과 같이 그대로. (data-status ..) <br />
[비표준 속성 정리 보러가기](../JavaScript/인터렉티브자바스크립트.md#비표준속성)

```jsx
import ReactDOM from "react-dom";

ReactDOM.render(
  <div>
    상태 변경:
    <button className="btn" data-status="대기중">
      대기중
    </button>
    <button className="btn" data-status="진행중">
      진행중
    </button>
    <button className="btn" data-status="완료">
      완료
    </button>
  </div>,
  document.getElementById("root")
);
```

### 2. 자바스크립트 예약어와 같은 속성명은 사용할 수 없음

ex) for = htmlFor, class -> className <br />

```jsx
// 특정 input 요소와 연결시킬 때 htmlFor 사용

ReactDOM.render(
  <form>
    <label htmlFor="name">이름</label>
    <input id="name" className="name-input" type="text" />
  </form>,
  document.getElementById("root")
);
```

### 3. 반드시 하나의 요소로 감싸야한다

`오류발생 ex.`

```jsx
import ReactDOM from 'react-dom';

ReactDOM.render(
  <p>안녕</p>
  <p>리액트!</p>,
  document.getElementById('root')
);

```

`해결 1` <br />

부모 태그를 만들어 하나의 요소로 만들어주기

```jsx
import ReactDOM from "react-dom";

ReactDOM.render(
  <div>
    <p>안녕</p>
    <p>리액트!</p>
  </div>,
  document.getElementById("root")
);
```

`해결 2` <br />

꼭 필요하지 않은 부모 태그가 작성될 수 있으므로, Fragment로 감싸서 여러 요소 작성하기 <br />
또는 빈 태그로 감싸도 됨.

```jsx
import { Fragment } from "react";
import ReactDOM from "react-dom";

ReactDOM.render(
  <Fragment>
    <p>안녕</p>
    <p>리액트!</p>
  </Fragment>,
  document.getElementById("root")
);
```

```jsx
import ReactDOM from "react-dom";

ReactDOM.render(
  <>
    <p>안녕</p>
    <p>리액트!</p>
  </>,
  document.getElementById("root")
);
```

### 4. 중괄호로 자바스크립트 표현식 넣기.

ex.

```jsx
import ReactDOM from "react-dom";

const product = "맥북";

ReactDOM.render(
  <h1>나만의 {product} 주문하기</h1>,
  document.getElementById("root")
);
```

ex2) 조건부 렌더링

```jsx
function App() {
  const isLoggedIn = true;

  return (
    <div>
      {isLoggedIn ? <p>환영합니다!</p> : <p>로그인 해주세요</p>}
      {isLoggedIn && <button>로그아웃</button>}
    </div>
  );
}
```

ex3) 배열 렌더링

```jsx
function App() {
  const fruits = ["사과", "바나나", "오렌지"];

  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  );
}
```

ex4) 함수 호출

```jsx
function App() {
  const getCurrentTime = () => {
    return new Date().toLocaleTimeString();
  };

  return (
    <div>
      <p>현재 시간: {getCurrentTime()}</p>
    </div>
  );
}
```

ex5) 이벤트 핸들러

```jsx
function App() {
  const handleClick = () => {
    alert("버튼 클릭!");
  };

  return <button onClick={handleClick}>클릭하세요</button>;
}
```

### 리액트 엘리먼트

: JSX 문법으로 작성한 태그는 자바스크립트 객체로 변환되며, 이를 리액트 엘리먼트라고 한다. <br />
이 엘리먼트를 `ReactDOM.render()` 또는 최신 버전에선 `createRoot().render()`의 인자로 전달하면, 리액트가 이를 해석해 실제 DOM 요소로 만들어 브라우저에 렌더링한다.

```jsx
const element = <h1>Hello, React!</h1>;
```

이런 JSX 문법이 실제로는 아래와 같은 객체로 변환됨.

```js
const element = {
  type: "h1",
  props: {
    children: "Hello, React!",
  },
};
```

### 리액트 컴포넌트의 이름은 반드시 첫 글자를 대문자로 해야한다. 소문자로 작성하면 오류남

왜 ? 소문자로 시작하면 JSX에서 일반 HTML 태그로 오해하여 오류가 발생함.

```jsx
function app() {
  return <h1>Hello</h1>;
}

export default function Main() {
  return <app />; // ❌ 오류 발생! 'app' is not defined in JSX
}
```

```jsx
function App() {
  return <h1>Hello</h1>;
}

export default function Main() {
  return <App />; // ✅ 정상 작동
}
```

### Props

: 리액트에서 컴포넌트에 지정한 속성들. 컴포넌트에 데이터를 전달하는 방법 <br />
**컴포넌트의 설정값은 부모가 제어한다!**

특징

- 읽기 전용(Read-only) : 컴포넌트 내부에서 props를 직접 수정할 수 없음
- 단방향 데이터 흐름 : 부모 -> 자식 방향으로만 데이터가 흐름
- 불변성 : props는 변경되지 않는 값임

ex.

```jsx
// 부모 컴포넌트
function App() {
  return (
    <div>
      <Greeting name="김철수" age={25} isStudent={true} />
      <Greeting name="이영희" age={30} isStudent={false} />
    </div>
  );
}
```

```jsx
// 자식 컴포넌트 - 함수형 컴포넌트
function Greeting(props) {
  return (
    <div>
      <h2>안녕하세요, {props.name}님!</h2>
      <p>나이: {props.age}세</p>
      <p>학생 여부: {props.isStudent ? "학생" : "직장인"}</p>
    </div>
  );
}
```

### `구조 분해 할당 사용하기`

[구조 분해 할당 정리 보러가기](../JavaScript//모던자바스크립트.md#구조-분해-할당-객체-배열)
<br />
간단 예시.

객체 구조 분해 (이름 우선)

```jsx
const person = { name: "철수", age: 25 };
const { name, age } = person;

//변수 이름 바꾸고 싶으면
const { name: userName, age: userAge } = person;
```

배열 구조 분해 (순서 우선)

```jsx
const colors = ["빨강", "파랑", "노랑"];
const [red, , yellow] = colors; // 파랑은 건너뛰기
console.log(red); // '빨강'
console.log(yellow); // '노랑'
```

위의 예시

```jsx
function Greeting({ name, age, isStudent }) {
  return (
    <div>
      <h2>안녕하세요, {name}님!</h2>
      <p>나이: {age}세</p>
      <p>학생 여부: {isStudent ? "학생" : "직장인"}</p>
    </div>
  );
}
```

ex2)

```jsx
function ComponentExample({
  title,
  score,
  isActive,
  items,
  user,
  onClick,
  customElement,
}) {
  return (
    <div>
      <p>제목: {title}</p>
      <p>점수: {score}</p>
      <p>활성화: {isActive ? "Yes" : "No"}</p>
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
      <p>
        사용자: {user.name} ({user.email})
      </p>
      <button onClick={onClick}>클릭하세요</button>
      <div>{customElement}</div>
    </div>
  );
}

// 사용 예시
function App() {
  const handleClick = () => alert("버튼이 클릭되었습니다!");

  return (
    <ComponentExample
      title="제목입니다" //문자열은 {} 없이 " "로 작성 가능
      score={95}
      isActive={true}
      items={["사과", "바나나", "오렌지"]}
      user={{ name: "김철수", email: "kim@example.com" }}
      onClick={handleClick}
      customElement={<strong>강조된 텍스트</strong>}
    />
  );
}
```

### children (특별한 props)

: 컴포넌트 태그 사이에 포함된 모든 내용을 의미하며, 컴포넌트를 더 **유연하고 재사용 가능하게 만드는 강력한 도구** <br />
**틀은 컴포넌트가 잡고, 내용은 부모가 채운다!**

특징

- 컴포넌트의 여는 태그와 닫는 태그 사이의 모든 내용
- 텍스트, JSX 요소, 다른 컴포넌트 등이 될 수 있음
- `props.children` 또는 **구조 분해로 `children`** 직접 받을 수 있음
- 여러 개의 children이 있을 경우 배열로 전달됨

#### 💡 왜 유용한가?

- **컴포넌트 내부에 다양한 콘텐츠를 주입**할 수 있음. **레이아웃은 재사용하고 내용만 바꿀 수 있음**
- 즉, 부모가 "무엇을 보여줄지" 정하고, 자식 컴포넌트는 **"어떻게 보여줄지"만 신경 씀**

ex.

```jsx
// 부모 컴포넌트
function App() {
  return (
    <div>
      <Card>
        <h2>카드 제목</h2>
        <p>카드 내용입니다.</p>
        <button>액션 버튼</button>
      </Card>
    </div>
  );
}
```

```jsx
// Card 컴포넌트
//  "박스 모양, 패딩, 그림자, 간격, 레이아웃" 같은 스타일과 구조를 담당
function Card({ children }) {
  return (
    <div className="card">
      <div className="card-content">{children}</div>
    </div>
  );
}
```

다양한 활용

```jsx
<Card>
  {/* Card는 디자인 컨테이너 역할*/}
  <h2>회원 정보</h2>
  <UserAvatar />
  <UserBio />
  <button>팔로우</button>
</Card>
```

예시로 `props` VS `children`

```jsx
<Card color="blue">
  <h2>제목</h2>
</Card>
```

- `color="blue` -> **props** <br />
  -> 레이아웃, 스타일, 동작을 제어하기 위한 데이터

- `<h2>제목</h2>` -> **children** <br />
  -> 화면에 들어갈 실제 내용

### React.cloneElement()

: 이미 존재하는 리액트 엘리먼트를 복제하면서
새로운 props를 주입하거나,
기존 props를 오버라이드(덮어쓰기) 하거나,
children을 바꿀 수 있게 해주는 함수 <br />
=> React.cloneElement는 기존 엘리먼트를 복제하면서 props나 children을 커스터마이징할 수 있게 해주는 고급 유틸리티 함수다.

#### `문법 구조`

```js
React.cloneElement(
  element,        // 복제할 React 엘리먼트
  props?,         // 덮어씌울 or 추가할 props 객체
  children?       // 새로 줄 자식 요소들 (기존 children 덮어씀)
)
```

ex1) children으로 받은 컴포넌트에 props를 주입하고 싶을 때

```jsx
function Wrapper({ children }) {
  return React.cloneElement(children, { className: "fancy" });
}

function App() {
  return (
    <Wrapper>
      <button>Click me</button> {/* -> className='fancy'가 주입됨 */}
    </Wrapper>
  );
}
```

ex2) 특정 조건에서 props 일부만 변경하고 싶을 때

```jsx
function ThemeWrapper({ children }) {
  return React.cloneElement(children, {
    style: { color: "white", backgroundColor: "black" },
  });
}
```

ex3) children이 여러 개인 경우 map으로 처리 가능

```jsx
function FormGroup({ children }) {
  return (
    <div>
      {React.Children.map(children, (child) =>
        React.cloneElement(child, { disabled: true })
      )}
    </div>
  );
}

// 사용
<FormGroup>
  <input type="text" />
  <input type="password" />
</FormGroup>;
```

### React에서 객체를 업데이트 하는 방법

```jsx
const user = {
  name: "김철수",
  age: 25,
  email: "kim@example.com",
};

// ❌ 잘못된 방법 - 원본 객체 직접 수정
user.name = "이영희"; // 원본 객체가 변경됨 (mutation)

// ✅ 올바른 방법 - 새 객체 생성
const updatedUser = { ...user, name: "이영희" };
```

### 참조형 State를 다룰 때 주의사항

자바스크립트에서 변수는 기본형과 참조형으로 나뉨. [자세한 설명 보러가기](../JavaScript/모던자바스크립트.md#자바스크립트-자료형의-2가지-분류)

```jsx
const [gameHistory, setGameHistory] = useState([]);

const handleRollClick = () => {
  const nextNum = random(6);
  gameHistory.push(nextNum);
  setGameHistory(gameHistory); // state가 제대로 변경되지 않는다!
};
```

이렇게 push 메소드를 이용해서 배열의 값을 변경한 뒤, 변경된 배열을 setter 함수로 state를 변경하려고 하면 동작하지 않음. <br />
왜 ? -> state는 배열의 값을 가지고 있는게 아니라, 배열의 주솟값을 참조하고 있어서, push 메소드로 배열 안에 요소를 변경했다고 하더라도 결과적으로는 배열의 주솟값은 변경된 것이 아니기 때문. (리액트 입장에서는 state가 참조하는 주솟값이 여전히 똑같기 때문에 바뀌지 않았다고 판단. 그래서 리렌더링을 안함) <br />

#### `그래서 ?`

참조형 state를 활용할 때는 반드시 새로운 참조형 값을 만들어서 변경해야함 -> spread (...) 문법을 활용

- `[...]`을 쓰면 기존 배열을 복사하고, 새로운 값을 추가해서 새로운 배열 객체를 생성함
- 즉, 이전 참조값과 완전히 다른 새 참조값

```jsx
const [gameHistory, setGameHistory] = useState([]);

const handleRollClick = () => {
  const nextNum = random(6);
  setGameHistory([...gameHistory, nextNum]); // state가 제대로 변경된다!
};
```

배열 업데이트시.

```jsx
setArray(prev => [...prev, newItem]);       // 추가
setArray(prev => prev.filter(i => i !== x)); // 삭제
setArray(prev => prev.map(...))              // 수정
```

-> 항상 "새 배열"을 만들어야 리렌더링 됨!

✍️ 정리 <br />

참조형 state는 값이 아닌 "참조"를 기준으로 변경 여부를 판단하므로,
push처럼 기존 배열을 변경하지 말고, spread처럼 새 객체를 만들어서 setState 해야 한다.

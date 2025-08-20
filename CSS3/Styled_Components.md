## Styled Components

### Styled Components 란?

Styled Components는 “컴포넌트 단위로 스타일을 선언”하는 CSS-in-JS 라이브러리이다. <br />
JS/TS 파일 안에서 styled.button 같은 팩토리 함수로 스타일이 입혀진 React 컴포넌트를 만들고, 해당 컴포넌트가 고유한 className을 가진 CSS를 자동 생성·주입해준다 !

팩토리 함수 : **"특정 객체 (또는 컴포넌트) 를 만들어서 반환해주는 함수"**

보통 JS에서 객체를 만들 때 `new` 키워드로 클래스를 쓰는데,

```jsx
class User {
  constructor(name) {
    this.name = name;
  }
}
const u = new User("민수");
```

팩토리 함수는 `new` 없이 함수가 객체를 반환하는 패턴을 말함.

```jsx
function createUser(name) {
  return { name };
}
const u = createUser("민수"); // { name: "민수" }
```

여기서는, CSS를 받아서 스타일이 입혀진 React 컴포넌트를 반환해서 팩토리 함수라고 칭함.

#### `특징`

1. CSS-in-JS

   - CSS를 JavaScript 파일 내에서 작성
   - 컴포넌트와 스타일이 같은 파일에 위치하여 관리 용이

2. 동적 스타일링

   - Props를 통해 동적으로 스타일 변경 가능
   - JavaScript의 모든 기능을 스타일링에 활용

3. 스코프 격리

   - 각 컴포넌트의 스타일이 자동으로 격리되어 CSS 충돌 방지 (자동으로 고유 클래스명 생성됨)

4. 재사용되는 코드 유지보수 가능.

#### `설치 방법`

```bash
# npm
npm install styled-components

# yarn
yarn add styled-components
```

### `기본 사용법`

### 기본 컴포넌트 생성

```jsx
import styled from "styled-components";

// 기본 스타일드 컴포넌트 생성
const Button = styled.button`
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px 20px;
  cursor: pointer;

  &:hover {
    background-color: #0056b3;
  }
`;

function App() {
  return (
    <div>
      <h1>Styled Components 예시</h1>
      <Button>클릭하세요</Button>
    </div>
  );
}
```

`styled` 뒤에 태그(혹은 컴포넌트)를 쓰고, 백틱 안에 CSS 속성을 적으면 새로운 컴포넌트를 만듦.

### 기존 컴포넌트 확장

```jsx
// 기존 Button 컴포넌트를 확장
const PrimaryButton = styled(Button)`
  background-color: #28a745;

  &:hover {
    background-color: #1e7e34;
  }
`;

const SecondaryButton = styled(Button)`
  background-color: #6c757d;
  color: white;

  &:hover {
    background-color: #545b62;
  }
`;
```

-> Nesting 문법 지원. ( CSS 안에서 조건에 따른 스타일을 계층적으로 작성할 수 있다)

### 내부 특정 컴포넌트만 선택해서 스타일링하기

```jsx
import styled from "styled-components";
import nailImg from "./nail.png";

const Icon = styled.img`
  width: 16px;
  height: 16px;
`;

const StyledButton = styled.button`
  background-color: #6750a4;
  border: none;
  color: #ffffff;
  padding: 16px;

  & ${Icon} {
    margin-right: 4px;
  }

  &:hover,
  &:active {
    background-color: #463770;
  }
`;

function Button({ children, ...buttonProps }) {
  return (
    <StyledButton {...buttonProps}>
      <Icon src={nailImg} alt="nail icon" />
      {children}
    </StyledButton>
  );
}

export default Button;
```

쓰는법은

```jsx
& ${자식컴포넌트이름} {
    /* 스타일 */
}
```

위에 예시에서는 StyledButton 안에서 Icon을 쓸 때만 (부모-자식 관계일때만) margin-right이 적용되고, 다른곳에서 Icon을 쓰면 margin-right이 적용안된채로 나온다.

### Props를 활용한 동적 스타일링

`<Button primary size="large" />`처럼 props를 넘기면, JS 표현식 삽입법(`${ ... }`)을 이용해서 props 값에 따라 스타일을 동적으로 적용할 수 있다.

```jsx
const DynamicButton = styled.button`
  background-color: ${(props) => (props.primary ? "#007bff" : "#6c757d")};
  color: white;
  border: none;
  border-radius: 4px;
  padding: ${(props) => (props.size === "large" ? "15px 30px" : "10px 20px")};
  font-size: ${(props) => (props.size === "large" ? "18px" : "14px")};
  cursor: pointer;

  &:hover {
    opacity: 0.8;
  }
`;

// 사용 예시
<DynamicButton primary size="large">
  버튼
</DynamicButton>;
```

여기서, props는 객체이기 때문에, 순서 상관없다.

이것도 됨.

```jsx
<DynamicButton size="large" primary>
  버튼
</DynamicButton>
```

-> 이처럼 템플릿 리터럴 안에서 달러와 중괄호 (`${...}`)를 사용해서 JavaSccript 코드를 집어넣는 것을 표현식 삽입법이라고 부름.

`동적 스타일링 팁` : 기본값 설정

```jsx
const SIZES = {
  large: 24,
  medium: 20,
  small: 16
};

const Button = styled.button`
  ...
  font-size: ${(props) => SIZES[props.size]}px;
`;

//또는 (구조분해)

font-size: ${({size}) => SIZES[size]}px;
```

여기서, Styled Components에서는 undefined 값을 빈 문자열로 처리하기 떄문에, <br />
font-size: px; 같은 잘못된 CSS 코드가 되므로, 기본값을 정해주는게 좋다.

null 병합연산자 사용하기

```jsx
font-size: ${({ size }) => SIZES[size] ?? SIZES['medium']}px;
```

### 조건부 스타일링

- 논리 연산자 사용하기

```jsx
const ConditionalButton = styled.button`
  background-color: #f8f9fa;
  border: 2px solid #dee2e6;
  border-radius: 4px;
  padding: 10px 20px;
  cursor: pointer;

  ${(props) =>
    props.disabled &&
    `
    opacity: 0.5;
    cursor: not-allowed;
    background-color: #e9ecef;
  `}

  ${(props) =>
    props.variant === "danger" &&
    `
    background-color: #dc3545;
    color: white;
    border-color: #dc3545;
  `}
`;
```

- 삼항 연산자 사용하기

```jsx
border-radius: ${({ round }) => round ? `9999px` : `3px`};
```

### 글로벌 스타일 지정하기

```jsx
import { createGlobalStyle } from "styled-components";

const GlobalStyle = createGlobalStyle`
  * {
    box-sizing: border-box;
  }

  body {
    font-family: 'Noto Sans KR', sans-serif;
  }
`;

function App() {
  return (
    <>
      <GlobalStyle />
      <div>글로벌 스타일</div>
    </>
  );
}

export default App;
```

-> 이러면 Styled Components가 내부적으로 처리해서 `<head>` 태그 안에 우리가 작성한 CSS 코드를 넣어줌.

### styled(컴포넌트) + className 사용하기

#### HTML 태그 직접 사용 vs 커스텀 컴포넌트 사용

`HTML 태그 사용`

```jsx
// HTML 태그 직접 사용
const StyledButton = styled.button`
  background-color: blue;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
`;

// 사용:
<StyledButton>클릭하세요</StyledButton>;

// 실제 렌더링 결과:
// <button class="sc-abc123">클릭하세요</button>
```

`커스텀 컴포넌트 사용`

잘못된 ex.

```jsx
// ❌ className을 받지 않는 컴포넌트
function MyButton({ children }) {
  return <button>{children}</button>;
}

const StyledMyButton = styled(MyButton)`
  background-color: blue;
  padding: 10px 20px;
`;

// 이렇게 하면 스타일이 적용되지 않음!
<StyledMyButton>클릭하세요</StyledMyButton>;

// 실제 렌더링: <button>클릭하세요</button> (스타일 없음)
```

왜 잘못되었을까 ?

- `styled(MyButton)`은 `className="sc-123"`을 MyButton에게 넘김

- 그런데 `MyButton`이 이걸 무시해버림 → 최종 `<button>`에는 class 속성이 없음

- 따라서 CSS 매칭 실패 → 스타일 미적용

한줄 요약하자면, Styled Components는 내부적으로 className을 통해 스타일을 적용한다. 그런데 커스텀 컴포넌트가 그 className을 DOM까지 전달하지 않으면, CSS가 붙을 대상이 사라져서 스타일이 적용되지 않는다.

옯바른 ex.

```jsx
// ✅ className을 받아서 적용하는 컴포넌트
function MyButton({ className, children }) {
  return <button className={className}>{children}</button>;
}

const StyledMyButton = styled(MyButton)`
  background-color: blue;
  padding: 10px 20px;
`;

// 이제 스타일이 적용됨!
<StyledMyButton>클릭하세요</StyledMyButton>;

// 실제 렌더링: <button class="sc-abc123">클릭하세요</button>
```

**언제 어떤 방식 ?**

#### styled.태그 사용하는 경우 :

- 단순한 스타일링
- 특별한 로직이 없을 때
- 빠르고 간단하게 구현하고 싶을 때

ex. HTML 태그를 직접 사용하는 경우

```jsx
// 단순한 스타일링만 필요할 때
const Title = styled.h1`
  font-size: 2rem;
  color: #333;
`;

const Card = styled.div`
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
`;

const Button = styled.button`
  background: blue;
  color: white;
  border: none;
  padding: 10px 20px;
`;
```

#### styled(컴포넌트) + className 사용하는 경우 :

- 복잡한 로직이 있는 컴포넌트
- 이미 만들어진 컴포넌트에 스타일만 추가하고 싶을 때
- 여러 HTML 요소로 구성된 복잡한 구조
- 상태나 이벤트 처리가 필요한 경우

ex. 복잡한 로직이 있는 컴포넌트

```jsx
// 복잡한 로직이 있는 컴포넌트
function SearchInput({ className, onSearch, placeholder }) {
  const [value, setValue] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();
    onSearch(value);
  };

  return (
    <form className={className} onSubmit={handleSubmit}>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder={placeholder}
      />
      <button type="submit">검색</button>
    </form>
  );
}

const StyledSearchInput = styled(SearchInput)`
  display: flex;
  gap: 10px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 8px;
`;
```

```jsx
// ✅ 복잡한 로직이 있는 컴포넌트2
function ToggleButton({ className, isOn, onToggle, children }) {
  return (
    <button className={className} onClick={onToggle} aria-pressed={isOn}>
      {isOn ? "🟢" : "🔴"} {children}
    </button>
  );
}

const StyledToggleButton = styled(ToggleButton)`
  background: ${(props) => (props.isOn ? "#28a745" : "#dc3545")};
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
`;
```

### 테마 활용 (ThemeProvider)

```jsx
import { ThemeProvider } from "styled-components";

// 테마 정의
const theme = {
  colors: {
    primary: "#007bff",
    secondary: "#6c757d",
    success: "#28a745",
    danger: "#dc3545",
  },
  spacing: {
    small: "8px",
    medium: "16px",
    large: "24px",
  },
};

function App() {
  return (
      {/*여기서 테마를 제공하고 (ThemeProvider에서) */}
    <ThemeProvider theme={theme}>
      <div>
        <h1>앱</h1>
        <MyButton /> {/* 이 컴포넌트가 테마 사용 가능 */}
        <MyCard /> {/* 이 컴포넌트도 테마 사용 가능 */}
      </div>
    </ThemeProvider>
  );
}
```

하위 컴포넌트에서 테마를 사용할 때, <br />

방법 1 : Styled Components로 사용

```jsx
// MyButton.js
import styled from "styled-components";

const StyledButton = styled.button`
  background-color: ${(props) =>
    props.theme.colors.primary}; // 자동으로 테마 접근!
  padding: ${(props) => props.theme.spacing.medium};
  border: none;
  color: white;
`;

function MyButton() {
  return <StyledButton>버튼</StyledButton>;
}

export default MyButton;
```

방법 2 : useContext로 사용

```jsx
// MyCard.js
import { useContext } from "react";
import { ThemeContext } from "styled-components";

function MyCard() {
  const theme = useContext(ThemeContext); // 테마 가져오기

  return (
    <div
      style={{
        backgroundColor: theme.colors.success, // 테마 사용!
        padding: theme.spacing.medium,
      }}
    >
      카드 내용
    </div>
  );
}

export default MyCard;
```

### 태그의 역할을 바꾸고 싶을 때 as 키워드

예를들어, Button 컴포넌트가 `<button>`태그로 만들어져 있을 때,

```jsx
const Button = styled.button`
  /* ... */
`;
```

as로 태그 이름을 내려주면 해당하는 태그로 사용할 수 있음.

```jsx
<Button href="https://example.com" as="a">
  LinkButton
</Button>
```

이렇게하면, 태그가 `<a>` 로 바뀜

### 원치 않는 props 가 전달될 때

ex.

```jsx
import styled from "styled-components";

function Link({ className, children, ...props }) {
  return (
    <a {...props} className={className}>
      {children}
    </a>
  );
}

const StyledLink = styled(Link)`
  text-decoration: ${({ underline }) => (underline ? `underline` : `none`)};
`;

function App() {
  return (
    <StyledLink underline={false} href="https://codeit.kr">
      Codeit으로 가기
    </StyledLink>
  );
}

export default App;
```

위 코드를 실행하면, HTML 태그에 underline 이라는 속성을 지정했는데, 그 속성의 값이 문자열이 아니라서 생긴 오류가 남. <br />
`<a>` 태그에는 underline 이라는 속성이 없음. -> Spread 하는 과정에서 의도하지 않은 underline Prop까지 내려간게 원인. <br />

팁+)
JSX에서 `...props`는 객체의 모든 속성을 한 번에 컴포넌트나 태그에 전달하는 문법이야

#### `해결1` : Spread를 그대로 쓰되, 필터링

```jsx
function Link({ className, children, underline, ...props }) {
  return (
    <a {...props} className={className}>
      {children}
    </a>
  );
}
```

`underline`을 구조분해로 미리 빼내고 DOM에는 전달하지 않기.

#### `해결2` : Transient Prop을 만들면 된다. 앞에다 $ 기호를 붙여주면 된다.

Styled Components는 $로 시작하는 prop은 DOM에 전달하지 않음.

```jsx
import styled from "styled-components";

function Link({ className, children, ...props }) {
  return (
    <a {...props} className={className}>
      {children}
    </a>
  );
}

const StyledLink = styled(Link)`
  text-decoration: ${({ $underline }) => ($underline ? `underline` : `none`)};
`;

function App() {
  return (
    <StyledLink $underline={false} href="https://codeit.kr">
      Codeit으로 가기
    </StyledLink>
  );
}

export default App;
```

- $underline Prop은 StyledLink 안에서만 사용되고, Link로 전달되지는 않음.
- 스타일 전용 props는 무조건 $ 붙이는 습관을 들여라 !

### 애니메이션

```jsx
import styled, { keyframes } from "styled-components";

// 키프레임 애니메이션 정의
const fadeIn = keyframes`
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
`;

const AnimatedCard = styled.div`
  background-color: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  animation: ${fadeIn} 0.5s ease-in-out;
`;
```

### 미디어 쿼리

```jsx
const ResponsiveContainer = styled.div`
  padding: 16px;

  @media (max-width: 768px) {
    padding: 8px;
  }

  @media (min-width: 1024px) {
    padding: 32px;
    max-width: 1200px;
    margin: 0 auto;
  }
`;
```

### 장단점 비교

#### 장점

- 컴포넌트 기반: 스타일과 로직이 함께 관리되어 유지보수 용이
- 동적 스타일링: JavaScript의 모든 기능을 활용한 유연한 스타일링
- 자동 스코프: CSS 클래스명 충돌 방지
- 서버사이드 렌더링: SSR 지원
- 테마 시스템: 일관된 디자인 시스템 구축 가능

#### 단점

- 번들 크기: 추가 라이브러리로 인한 번들 크기 증가
- 런타임 오버헤드: 스타일 생성이 런타임에 발생
- 디버깅: 생성된 클래스명으로 인한 디버깅 어려움
- 학습 곡선: CSS-in-JS에 대한 새로운 개념 학습 필요

## 리액트로 웹사이트 만들기

#### `링크`

리액트 라우터에서는 a태그 대신에 Link 태그를 사용한다. to라는 prop으로 경로 지정

ex.

```jsx
<Link to="/posts">블로그</Link>
```

### 리액트 라우터

: 경로에 따라 페이지를 나누도록 해주는 라이브러리

#### `라우터`

#### - 사용법

최상위 컴포넌트에서 `<BrowserRouter>`로 감싸주고,
Routes컴포넌트 안에다가 Route 컴포넌트를 배치해서 페이지를 나눠줌. <br />

```jsx
<BrowserRouter>
    <Routes>
        <Route path = '/' element = {<HomePage />} />
        <Route path='/about' element={<AboutPage />} />
        <Route path='/contact' element={<ContactPage />} />
    <Routes>
</BrowserRouter>
```

- `path`: URL 경로를 지정 (예: '/', '/about', '/contact')
- `element`: 해당 경로에 매칭될 때 보여줄 리액트 컴포넌트를 JSX 형태로 전달
- `element`는 반드시 JSX 컴포넌트 형태로 작성해야 함 (`<HomePage />`, `<div>내용</div>` 등)

#### - 하위 페이지 나누기

: Route 컴포넌트 안에다가 Route 컴포넌트 배치 <br />
이때 하위 페이지 최상위 경로는 path prop이 아니라 index prop을 사용한다.

ex.

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="posts" element={<PostLayout />}>
          <Route index element={<PostListPage />} />
          <Route path=":postId" element={<PostPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

-> posts 경로의 기본 하위 페이지로 PostListPage를 보여주고 싶을 때 index를 사용. <br />
즉, `index`는 해당 부모 경로에 기본으로 보여줄 컴포넌트를 의미하며, `path`를 따로 적지 않고 `element`만 지정함.

```jsx
// PostLayout.js
import { Outlet } from "react-router-dom";

function PostLayout() {
  return (
    <div>
      {/* 공통으로 보여줄 내용 */}
      <header>
        <h1>블로그 포스트</h1>
        <nav>
          <Link to="/posts">전체 목록</Link>
          <Link to="/posts/1">첫 번째 포스트</Link>
        </nav>
      </header>

      {/* 자식 Route 컴포넌트들이 여기에 렌더링됨 */}
      <main>
        <Outlet />
      </main>

      {/* 공통 푸터 */}
      <footer>
        <p>© 2024 My Blog</p>
      </footer>
    </div>
  );
}
```

#### `Outlet`이 왜 필요한가?

하위 페이지(중첩 라우팅)를 사용할 때, 부모 컴포넌트에서 자식 컴포넌트를 어디에 렌더링할지 지정해야 함. 이때 `Outlet`이 그 위치를 표시하는 역할을 한다.

#### `실제 렌더링 결과`

`/post` 경로 접속 시 :

```jsx
<div>
  <header>...</header>
  <main>
    <PostListPage /> {/* Outlet 위치에 렌더링 */}
  </main>
  <footer>...</footer>
</div>
```

`/posts/123` 경로 접속 시 :

```jsx
<div>
  <header>...</header>
  <main>
    <PostPage /> {/* Outlet 위치에 렌더링 */}
  </main>
  <footer>...</footer>
</div>
```

#### 동적인 경로 다루기

: useParams 훅 (경로에서 사용하는 동적인 값을 파라미터라고 부르고, 이런 파라미터를 모아 놓은 걸 파람즈(params)라고 함.) <br />
path에 콜론(:)을 넣어서 사용함.

ex.

```jsx
<Routes>
  <Route path="/">
    <HomePage />
  </Route>
  <Route path="posts" element={<PostLayout />}>
    <Route index element={<PostListPage />} />
    <Route path=":postId" element={<PostPage />} />
  </Route>
</Routes>
```

경로 파라미터를 쓸 때는,

```jsx
function PostPage() {
  const { postId } = useParams(); // URL에서 postId 값을 추출

  console.log("현재 postId:", postId);
  // URL이 /posts/123이면 → postId = "123"
  // URL이 /posts/hello면 → postId = "hello"

  return (
    <div>
      <h1>포스트 ID: {postId}</h1>
      <p>이 포스트의 상세 내용을 보여줍니다.</p>
    </div>
  );
}
```

#### useSearchParams

`searchParams` : 현재 URL의 쿼리 파라미터를 나타내는 URLSearchParams 객체 <br />
`setSearchParams` : 쿼리 파라미터를 수정하는 함수 <br />

#### - 쿼리 값 읽기

```jsx
const [searchParams, setSearchParams] = useSearchParams();

const id = searchParams.get("id");
```

-> URL이 ...?id=123이면, id = '123'이 된다. 값이 없으면 null

#### - 쿼리 값 설정하기

`모든 쿼리 교체` :

```jsx
setSearchParams({ id: "456", filter: "popular" });
```

위 코드는 URL을 ...?id=456&filter=popular로 바꿈. <br />
**주의**: 기존에 있던 다른 쿼리 파라미터들은 모두 사라짐!

`기존 쿼리를 유지하면서 수정` :

```jsx
setSearchParams((prev) => {
  const next = new URLSearchParams(prev); // 현재 쿼리의 복사본 생성
  next.set("filter", "popular"); // filter만 추가/수정
  return next;
});
```

#### useLocation 훅

: 현재 페이지의 경로 정보를 가져오는 훅

```jsx
// useLocation - 현재 경로 정보 가져오기
const location = useLocation();
console.log(location.pathname); // 현재 경로 (예: "/posts/123")
console.log(location.search); // 쿼리 스트링 (예: "?filter=popular")
console.log(location.state); // 페이지 이동 시 전달된 상태 데이터
```

### 페이지 이동 : Navigate 컴포넌트

: Navigate 컴포넌트의 `to prop`으로 지정한 경로로 이동함

ex.

```jsx
import { Navigate } from "react-router-dom";

function PostPage() {
  // ...

  const post = getPost(postId);

  // post가 없는 경우 /posts 페이지로 이동
  if (!post) {
    return <Navigate to="/posts" />;
  }

  // ...
}
```

#### useNavigate Hook

: navigate 함수를 가져오면 이 함수를 통해 페이지 이동 가능. <br />

ex.

```jsx
const navigate = useNavigate();

const handleClick = () => {
  // ... 어떤 작업을 한 다음에 페이지를 이동
  navigate("/wishlist");
};
```

#### `navigate 함수 옵션`

```jsx
// 기본 이동
navigate("/wishlist");

// 히스토리 스택에서 교체 (뒤로가기 방지)
navigate("/login", { replace: true });

// 상태와 함께 이동 (프로필 페이지에서 "어디서 왔는지" 알아야 할 때)
navigate("/profile", { state: { from: "dashboard" } });

// 상대 경로로 이동
navigate("../parent-route");

// 뒤로가기/앞으로가기
navigate(-1); // 뒤로가기
navigate(1); // 앞으로가기
```

#### `Link` VS `Navigate` VS `useNavigate`

| 방법          | 언제 사용              | 사용 특징                              |
| ------------- | ---------------------- | -------------------------------------- |
| `Link`        | 유저가 클릭할 때       | 사용자 상호작용 기반 페이지 이동       |
| `Navigate`    | 조건 만족 시 자동 이동 | 컴포넌트 렌더링 시점에 리다이렉트      |
| `useNavigate` | 코드 로직 끝나고 이동  | JavaScript 로직으로 동적으로 이동 제어 |

`Link` 사용 예시 :

- 사용자가 클릭해서 페이지를 이동하도록 할 때 사용
- 네비게이션 메뉴, 버튼, 링크 등

`Navigate` 사용 예시 :

- 조건 만족 시 자동 이동 (리다이렉트용)
- 특정 경로에서 렌더링 시점에 다른 페이지로 이동시키고 싶을 떄 사용
- 로그인하지 않은 사용자가 회원 전용 페이지 접근 시 로그인 페이지로 리다이렉트
- 삭제된 상품 페이지 접근 시 상품 목록으로 리다이렉트

`useNavigate` 사용 예시 :

- 코드 로직 끝나고 이동 (JS로 이동 제어)
- 특정한 코드의 실행이 끝나고나서 페이지를 이동시키고 싶을 때 사용하면 됨.
- 쇼핑몰에서 장바구니에 담기를 눌렀을 때 리퀘스트를 보내고 장바구니 페이지로 이동시키는 경우
- 쇼핑몰에서 결제하기 버튼을 누르고 나서 모든 결제가 완료된 후에 페이지를 이동시키는 경우
- 리다이렉트된 로그인 페이지에서 로그인을 완료한 후에 처음 진입했던 페이지로 돌아가는 경우

#### 왜 꼭 저 상황에서는 이 방법을 써야할까 ?

#### 1. Link - 선언적(Declarative) 방식

```jsx
<Link to="/profile">프로필</Link>
```

- HTML의 <a> 태그와 유사한 역할
- 접근성(스크린 리더, 키보드 탐색) 자동 지원
- SEO에 유리 (크롤러가 링크를 인식)
- 우클릭으로 새 탭에서 열기 가능

#### 2. Navigate - 조건부 렌더링

```jsx
if (!isLoggedIn) {
  return <Navigate to="/login" />;
}
```

- 렌더링 과정에서 리다이렉트
- 컴포넌트가 화면에 그려지기 전에 이동
- 조건부 로직에 최적화됨

#### 3. useNavigate - 명령적(Imperative) 방식

```jsx
const handleSubmit = async () => {
  await submitForm();
  navigate("/success"); // 비동기 작업 후 이동
};
```

- 이벤트 핸들러 안에서 사용
- 비동기 작업 완료 후 이동
- 복잡한 로직 처리 가능

### 렌더링 종류

리액트에서의 렌더링 방식은 총 세 가지로 나뉜다:  
**CSR (Client Side Rendering)**, **SSR (Server Side Rendering)**, **SSG (Static Site Generation)**

각 방식은 사용자 경험, SEO, 초기 로딩 속도, 서버 부담 등 다양한 요소에서 차이가 있다.

#### 1. 클라이언트 사이드 렌더링 (CSR)

#### 🔍 정의

- 브라우저에서 리액트 코드가 실행되어 HTML을 동적으로 생성하는 방식
- HTML은 기본적으로 비어 있고, JavaScript가 실행되면서 뷰를 그림

#### ⚙️ 동작 원리

- 브라우저는 초기 HTML 로드 후, 자바스크립트 번들(JS)을 다운로드
- 그 후 `ReactDOM.render()`로 뷰를 생성

#### ✅ 장점

- 페이지 간 전환이 매우 빠름
- 사용자 상호작용이 부드럽고 동적 처리에 최적화

#### ⚠️ 단점

- 초기 로딩 속도가 느림 (JS 다운로드 → 실행 → 렌더)
- SEO에 불리 (검색 엔진이 HTML 내용을 파악하기 어려움)

#### 2. 서버사이드 렌더링 (SSR)

#### 🔍 정의

- 서버에서 리액트 코드를 실행하여 **완성된 HTML**을 만들어 클라이언트에 전송

#### ⚙️ 동작 원리

- 클라이언트 요청 → 서버에서 HTML 생성 → 응답
- 이후 클라이언트에서 **하이드레이션(Hydration)** 과정을 통해 React와 연결

#### 💧 하이드레이션(Hydration)이란?

- 서버에서 전달된 정적인 HTML에 리액트의 동적인 기능(이벤트, 상태 등)을 "물 붓듯" 다시 활성화하는 과정
- ex) HTML은 보이지만, 클릭/입력은 안 됨 → JS가 연결되며 인터랙션이 가능해짐

#### ✅ 장점

- 초기 화면 표시 속도가 빠름 (FCP 개선)
- SEO에 유리 (검색 엔진이 완성된 HTML을 읽을 수 있음)

#### ⚠️ 단점

- 서버 부하 증가 (모든 요청에 대해 서버가 HTML 생성)
- 하이드레이션 지연으로 인해 초기 인터랙션이 늦어질 수 있음

#### 3. 정적 사이트 생성 (SSG)

#### 🔍 정의

- **빌드 시점**에 HTML을 미리 생성해두고, 이를 서버나 CDN에 배포하는 방식

#### ⚙️ 동작 원리

- 페이지 요청이 오기 전에 HTML을 만들어두고, 클라이언트는 이를 빠르게 로드
- JS를 통해 동적 데이터도 fetch 가능 (블로그 글, 문서, 뉴스 등 변하지 않는 페이지)

#### ✅ 장점

- 가장 빠른 페이지 로딩 속도 (CDN 캐싱 활용 가능)
- 서버 리소스 사용 거의 없음 (정적 파일만 서빙)
- 높은 SEO 친화성

`CDN 캐싱 활용 가능` ?

: SSG는 HTML을 미리 만들어두기 때문에, 전 세계에 있는 CDN 서버에 그 파일을 올려놓고, 사용자에게 가까운 곳에서 빠르게 제공할 수 있다. <br />
그니까 한국에서 접속한 사용자는 한국에 위치한 CDN 서버에서 바로 받고, 미국 사용자는 미국에 있는 CDN 서버에서 받음.

#### ⚠️ 단점

- **동적 컨텐츠 처리**가 제한적 (사용자별로 바뀌는 콘텐츠, 로그인 상태, 실시간 알림)
- 컨텐츠 업데이트 시 다시 전체 빌드 필요 (빌드 시간 ↑)

#### 📊 성능/SEO 관점 비교

| 지표             | CSR     | SSR     | SSG          |
| ---------------- | ------- | ------- | ------------ |
| 초기 로딩 속도   | ❌ 느림 | ✅ 빠름 | ✅ 매우 빠름 |
| 페이지 전환 속도 | ✅ 빠름 | ⚠️ 보통 | ⚠️ 보통      |
| SEO              | ❌ 불리 | ✅ 유리 | ✅ 유리      |
| 서버 부하        | ✅ 없음 | ❌ 큼   | ✅ 없음      |
| 구축 난이도      | ✅ 쉬움 | ⚠️ 보통 | ⚠️ 보통      |

#### ⏱ 성능 지표별 비교

| 성능 지표        | 설명                                                        | 빠른 순서       |
| ---------------- | ----------------------------------------------------------- | --------------- |
| **FCP**          | First Contentful Paint (최초 콘텐츠가 화면에 나타나는 시간) | SSG > SSR > CSR |
| **LCP**          | Largest Contentful Paint (가장 큰 요소가 로드되는 시간)     | SSG > SSR > CSR |
| **TTI** (선택적) | Time to Interactive (사용자가 조작 가능한 시점)             | SSG > SSR > CSR |

#### 🧠 SEO 최적화 관점

| 렌더링 방식 | 검색 엔진 인식       | SEO 친화도 |
| ----------- | -------------------- | ---------- |
| CSR         | JS 실행 전엔 빈 HTML | ❌ 낮음    |
| SSR         | 완성된 HTML 제공     | ✅ 높음    |
| SSG         | 정적 HTML 제공       | ✅ 높음    |

> 최근 구글은 자바스크립트를 어느 정도 실행하지만,  
> **예측 가능한 SEO를 위해선 SSR 또는 SSG가 더 안정적**

#### 📝 결론

- 퍼포먼스 중심 → **SSG**
- 검색 노출 중심 + 실시간성 → **SSR**
- 내부 관리자 툴, 대시보드, 로그인 기반 SPA → **CSR**

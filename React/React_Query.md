## React Query

### TanStack Query (React Query)

: 서버 상태 데이터만을 집중적으로 관리함. <br />
웹 사이트에서 사용하는 데이터는 크게 2가지. <br /> 1. 클라이언트 상태 데이터 2. 서버 상태 데이터

클라이언트 상태 데이터는 웹사이트의 어떤 메뉴가 열렸는지 닫혔는지, 사용자가 어떤 버튼을 눌렀는지 아닌지와 같은 UI 상태 값이나, 유저가 입력 폼에 입력한 데이터 등 서버와는 상관없이 웹 브라우저 안에서만 사용하는 데이터이다. <br />
서버 상태 데이터는 서버에서 가져오는 데이터. 예를 들어, 네이버에 접속했을 때 우리가 보는 뉴스기사, 각종 글 등을 예시로 들 수 있다.

근데, 기존 Redux로 서버 데이터를 관리할 떄의 문제점 ( 데이터 하나 가져오는데 수십줄의 코드 필요, 로딩/에러 상태 수동 관리, 데이터 동기화 어려움 등)과, Context API의 한계 (불필요한 리렌더링, 복잡한 로직 등) 로 인하여 리액트 쿼리가 등장하게 되었다.

#### 핵심 기능 예시로 먼저 살펴보기

1. 캐싱 (Caching)

```jsx
// React Query는 자동으로 캐싱 처리
const { data } = useQuery({
  queryKey: ["posts"],
  queryFn: getPosts,
  staleTime: 5 * 60 * 1000, // 5분간 fresh 상태 유지
  cacheTime: 10 * 60 * 1000, // 10분간 캐시 보관
});

// 같은 queryKey로 다른 컴포넌트에서 호출해도
// 캐시된 데이터를 즉시 반환하고, 백그라운드에서 업데이트
```

#### `캐시` ? `캐싱` ?

`캐시` : 데이터를 미리 복사해 놓는 임시 장소. 보통 저장 공간의 크기는 작지만, 데이터를 가져오는 속도는 아주 빠름. <br />
`캐시`를 사용하는 것을 `'캐싱'`이라고 칭함 <br />
React Query의 캐시는 기본적으로 브라우저 메모리(RAM) 에 저장되고, 새로고침 시 사라진다.

- React 앱이 돌아가는 동안, QueryClient 인스턴스 안의 캐시 객체에 데이터가 들어간다.

- 브라우저 탭을 닫거나 새로고침하면 메모리는 날아가니까, 기본적으로 캐시도 다 지워진다.

- 단, 원한다면 persist 기능을 써서 localStorage / IndexedDB 등에 저장해 “페이지 새로고침 이후에도 캐시 복원”이 가능하다. <br />
  (예: `@tanstack/react-query-persist-client`)

  `persist` 기능 : React Query의 캐시 데이터를 브라우저의 영구 저장소(localStorage, sessionStorage, IndexedDB 등)에 저장해서 페이지 새로고침이나 브라우저 재시작 후에도 캐시를 복원할 수 있게 해주는 기능

2. 백그라운드 업데이트

```jsx
const { data } = useQuery({
  queryKey: ["posts"],
  queryFn: getPosts,
  refetchOnWindowFocus: true, // 창 포커스 시 자동 업데이트 (기본값 true)
  refetchOnReconnect: true, // 네트워크 재연결 시 자동 업데이트 (기본값 true)
  refetchInterval: 30000, // 30초마다 자동 업데이트, 실시간성이 중요한 데이터 (기본값 false)
});
```

- 사용자가 다른 탭에 갔다가 돌아오면 자동으로 최신 데이터 확인
- 네트워크가 끊어졌다가 다시 연결되면 자동으로 데이터 동기화
- 설정한 간격마다 자동으로 데이터 업데이트
- 기본값이 대부분의 상황에 적합해서, 특별한 요구사항이 없다면 별도 설정 없이 사용함

3. 자동 재시도

```jsx
const { data } = useQuery({
  queryKey: ["posts"],
  queryFn: getPosts,
  retry: 3, // 실패 시 3번까지 재시도
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  // 지수 백오프: 1초 → 2초 → 4초 → 8초 (최대 30초)
});
```

- 네트워크 일시적 오류에 대한 자동 복구
- 개발자가 재시도 로직을 직접 구현할 필요 없음
- 지능적인 재시도 간격 조절

4. 낙관적 업데이트

```jsx
const mutation = useMutation({
  mutationFn: updatePost,
  onMutate: async (newPost) => {
    // 진행 중인 쿼리 취소
    await queryClient.cancelQueries({ queryKey: ["posts"] });

    // 이전 데이터 백업
    const previousPosts = queryClient.getQueryData(["posts"]);

    // 낙관적으로 UI 업데이트 (서버 응답 전에)
    queryClient.setQueryData(["posts"], (old) =>
      old.map((post) => (post.id === newPost.id ? newPost : post))
    );

    return { previousPosts };
  },
  onError: (err, newPost, context) => {
    // 에러 발생 시 이전 데이터로 롤백
    queryClient.setQueryData(["posts"], context.previousPosts);
  },
});
```

- 버튼 클릭 즉시 UI가 업데이트됨 (서버 응답 대기 X)
- 실패 시 자동으로 이전 상태로 롤백
- 빠른 반응성으로 더 나은 UX 제공

### `useQuery()`

React Query의 가장 기본적인 훅으로, 서버에서 데이터를 가져올 때 사용한다.

#### 기본 사용법

useQuery는 두 개의 필수 매개변수를 받는다:

- queryKey: 쿼리를 고유하게 식별하는 키
- queryFn: 실제 데이터를 가져오는 비동기 함수

#### 예시

```jsx
// api.js에서,
const BASE_URL = "https://learn.codeit.kr/api/codestudit";

export async function getPosts() {
  const response = await fetch(`${BASE_URL}/posts`);
  return await response.json();
}
```

```jsx
// Homepage.js에서,
import { useQuery } from "@tanstack/react-query";
import { getPosts } from "./api";

function HomePage() {
  const result = useQuery({ queryKey: ["posts"], queryFn: getPosts });
  console.log(result);

  return <div>홈페이지</div>;
}

export default HomePage;
```

#### `useQuery()` 리턴 값

1. data : 성공적으로 fetch된 응답 데이터
2. isLoading : 첫 번째 fetch가 진행 중일 때 true
3. isFetching : 쿼리 요청이 진행 중일 때 true, 최초 로딩뿐만 아니라 백그라운드 리페치(윈도우 포커스, 네트워크 재연결, 수동 refetch) 시에도 true가 되며, 기존 데이터가 유지된 채로 업데이트 중임을 구분할 수 있다.
4. isError : 에러 발생 시 true
5. error : 에러 객체, 예) Axios 에러 객체 등
6. isSuccess : 요청이 성공적으로 완료되었을 때 true
7. status : 현재 쿼리의 진행 상태 ( `loading` , `error` , `success` )
8. refetch : 데이터를 수동으로 재요청할 수 있는 함수.
9. dataUpdatedAt : 현재의 데이터를 받아온 시간.

### `Query Status` VS `Fetch Status`

#### 1. Query Status : 실제로 받아 온 data값이 있는지 없는지를 나타내는 상태 값

세 가지 상태 값을 가짐. `pending`, `success`, `error`

1. pending : 아직 데이터를 받아오지 못했을 때를 의미 - isPending과 매칭
2. error : 데이터를 받아오는 중에 에러가 발생했음 - isError와 매칭
3. success : 데이터를 성공적으로 받아옴 - isSuccess와 매칭

매칭값을 활용해서 (isPending, isError, isSuccess) 현재 쿼리의 상태가 어떤지 확인할 수 있음.

#### 2. Fetch Status : QueryFn() 함수가 현재 실행되는 중인지 아닌지를 나타냄.

세 가지 상태 값을 가짐. `fetching`, `paused`, `idle`

1. fetching : 쿼리 함수가 실행되는 중임
2. paused : 쿼리 함수가 시작은 했는데 실제로 실행되고 있지는 않음 ( 대표적으로 네트워크가 오프라인이 된 경우)
3. idle : 쿼리 함수가 어떠한 작업도 하고 있지 않음.

### `React Query의 캐시`

우리가 설정한 queryKey로 캐싱을 할 수 있음. 위에 `useQuery()` 예제에서 ['posts'] 라는 쿼리 키로 데이터를 받아왔는데 이 동작을 자세히 살펴보면, <br />
먼저 전달받은 쿼리 키로 캐시에 저장된 데이터가 있는지 확인함.
-> 데이터가 없으면 쿼리 함수를 실행해서 데이터를 백엔드로부터 받아오고, 캐시에 저장. <br />
-> 데이터가 있으면 캐시에 저장되어 있는 데이터를 리턴함. 만약 데이터가

1. fresh 상태

   - 방금 패치된 따끈따끈한 데이터.

   - staleTime 동안은 "신선"하다고 보고, 추가 요청 안 하고, 캐시 데이터만 반환.

2. stale 상태

   - staleTime이 지나면 "구식(stale)"으로 표시됨.

   - 컴포넌트가 다시 마운트되거나, 윈도우 포커스가 돌아오거나, 네트워크가 다시 연결될 때 자동으로 refetch 시도.

3. inactive 상태

   - 쿼리를 사용하던 컴포넌트가 언마운트되면, 해당 쿼리는 inactive 상태가 됨.

   - inactive라고 해서 바로 삭제되는 건 아니고, `gcTime`(v5) 동안 메모리에 보존됨.

   - 이 기간 안에 같은 queryKey를 가진 `useQuery`가 다시 마운트되면 → 기존 캐시 데이터 재사용 + 필요 시 refetch

   - `gcTime`이 지나면 캐시에서 완전히 제거됨. <br />
     ( inactive 상태의 데이터는 가비지 컬렉션 타임이 지나면 캐시에서 삭제. 기본적으로 5분으로 설정되어있고, 값 변경 가능.)

간단 요약하자면,

- fresh: 데이터가 최신(staleTime 내) → 캐시 데이터만 반환.

- stale: 데이터가 구식(staleTime 지남) → 캐시 데이터를 우선 보여주고, 백그라운드에서 refetch.

- inactive: 해당 쿼리를 구독하는 컴포넌트가 없는 상태. 데이터는 여전히 메모리에 있고, gcTime/cacheTime 이내에 다시 구독되면 캐시 데이터 재사용. 만약 이 시간이 지나면 캐시에서 제거됨.

**Stale Time의 기본값은 0 이므로, 적절하게 변경해야함.**

ex. 1분으로 변경하기.

```jsx
function HomePage() {
  const result = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    staleTime: 60 * 1000,
    gcTime: 60 * 1000 * 10, //데이터가 메모리에 남아있는 시간
  });
  console.log(result);

  return <div>홈페이지</div>;
}
```

### 쿼리키를 배열로 설정하는 이유

: 배열로 지정하면 계층적 구조를 만들 수 있어, 같은 자원(Resource) 안에서 다양한 변형 데이터를 깔끔하게 구분할 수 있다.

예를 들면,

- `['posts']` → 전체 피드

- `['posts', username]` → 특정 유저 피드

- `['posts', username, { status: 'private' }]` → 특정 유저의 private 피드

👉 이렇게 배열을 쓰면 부분 무효화( `invalidateQueries(['posts'])` ), 파라미터 전달(queryFn에서 queryKey 분해), 디버깅 가독성까지 훨씬 좋아진다.

### 자주 쓰는 옵션과 리턴 값

- query status가 pending 상태일 때, isPending = true 인 상황에서는 로딩 중이라는 메시지 보여준다.

- isError = true 인 상황에서는 적절한 에러 미시지 보여주는 식으로 에러를 처리한다.

ex.

```jsx
function HomePage() {
  const {
    data: postsData,
    isPending,
    isError,
  } = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    retry: 0,
  });

  if (isPending) return "로딩 중입니다..."; // 또는 로딩중 컴포넌트 보여주기

  if (isError) return "에러가 발생했습니다."; // 또는 에러 컴포넌트 보여주기

  const posts = postsData?.results ?? [];

  return (
    <div>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>
            {post.user.name}: {post.content}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### `useMutation()`

: 데이터를 생성, 수정, 삭제할 때 사용하는 훅. `useQuery()`와 달리 컴포넌트 마운트 시 자동 실행되지 않고, 사용자가 직접 `.mutate()` 메소드를 호출해야 실행된다.

ex.

```jsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

// API 함수
async function createPost(newPost) {
  const response = await fetch("/api/posts", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(newPost),
  });
  return response.json();
}

function CreatePostForm() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: createPost,
    onSuccess: () => {
      // 성공 시 posts 쿼리를 무효화하여 최신 데이터로 업데이트
      queryClient.invalidateQueries({ queryKey: ["posts"] });
      // 이제 캐시의 posts 데이터가 "stale(오래된)" 상태로 표시됨
    },
  });

  const handleSubmit = (formData) => {
    // 사용자가 버튼 클릭 등의 액션을 할 때 수동으로 실행
    mutation.mutate({
      title: formData.title,
      content: formData.content,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* 폼 내용 */}
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? "저장 중..." : "저장"}
      </button>
    </form>
  );
}
```

#### `QueryClient()`

: QueryClient는 React Query의 핵심 인스턴스로, 모든 쿼리와 캐시를 관리하는 중앙 관리자. <br />
**캐시 저장소 관리자**라고 생각 !

도서관 비유로 이해하면,

```
// 도서관(QueryClient)에 책(데이터)들이 저장되어 있음
캐시 저장소: {
  ['posts']: [게시글1, 게시글2, 게시글3],
  ['users']: [유저1, 유저2],
  ['profile', userId]: { name: '홍길동', age: 25 }
}
```

실제 사용하는 이유

```jsx
// 상황: 게시글을 새로 작성했다고 가정

function CreatePost() {
  // 1. queryClient를 가져옴 (도서관 사서에게 접근)
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: createPost,
    onSuccess: () => {
      // 2. 사서에게 말함: "posts 관련 책들은 이제 오래됐어, 새로 주문해줘"
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    },
  });
}

function PostList() {
  // 3. 이 컴포넌트가 posts 데이터 요청
  const { data } = useQuery({ queryKey: ["posts"], queryFn: getPosts });
  // 4. QueryClient가 "아 posts가 invalidate됐네? 서버에서 새로 가져와야겠다"
  // 5. 자동으로 최신 데이터 가져와서 화면 업데이트
}
```

#### `QueryClient 설정`

```jsx
// App.js - 앱 전체에 도서관(QueryClient) 설치
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5분간 데이터를 신선하다고 간주
      cacheTime: 10 * 60 * 1000, // 10분간 캐시에 보관
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* 이제 모든 하위 컴포넌트에서 useQueryClient()로 접근 가능 */}
      <HomePage />
      <CreatePost />
    </QueryClientProvider>
  );
}
```

QueryClient의 역할:

- 쿼리 캐시 관리 (어떤 데이터가 저장되어 있는지)
- 쿼리 무효화 및 재요청 제어
- 전역적으로 쿼리 상태 관리

#### `invalidateQueries()`

invalidate의 의미

invalidate = "무효화하다", "효력을 잃게 하다"

React Query에서는 "캐시된 데이터를 오래된 것으로 표시하고, 다음에 해당 데이터가 필요할 때 서버에서 새로 가져오도록 하는 것"을 의미함.

왜 필요할까 ?

```jsx
// 상황: 사용자가 새 게시글을 작성했다고 가정

// 1. 게시글 목록을 미리 가져온 상태
const { data: posts } = useQuery({ queryKey: ["posts"], queryFn: getPosts });
// 현재 캐시: [게시글1, 게시글2, 게시글3]

// 2. 새 게시글 작성
const mutation = useMutation({
  mutationFn: createPost,
  onSuccess: () => {
    // 3. 이때 invalidateQueries를 안 하면?
    // 캐시는 여전히 [게시글1, 게시글2, 게시글3]
    // 새로 작성한 게시글4가 목록에 안 보임!

    // 4. invalidateQueries를 하면?
    queryClient.invalidateQueries({ queryKey: ["posts"] });
    // 캐시를 "오래된 것"으로 표시 → 자동으로 서버에서 최신 데이터 가져옴
    // 결과: [게시글1, 게시글2, 게시글3, 게시글4] ✅
  },
});
```

상세한 사용법

```jsx
import { useQueryClient } from "@tanstack/react-query";

function SomeComponent() {
  const queryClient = useQueryClient();

  // 1. 모든 쿼리 무효화
  queryClient.invalidateQueries();

  // 2. 특정 queryKey의 쿼리만 무효화
  queryClient.invalidateQueries({ queryKey: ["posts"] });

  // 3. 여러 쿼리 동시 무효화
  queryClient.invalidateQueries({ queryKey: ["posts"] });
  queryClient.invalidateQueries({ queryKey: ["users"] });

  // 4. 더 구체적인 조건으로 무효화
  queryClient.invalidateQueries({
    queryKey: ["posts"],
    exact: true, // 정확히 ['posts']만 무효화 (하위 키 제외)
  });

  // 5. 여러 관련 쿼리를 한 번에 무효화
  queryClient.invalidateQueries({
    predicate: (query) => query.queryKey[0] === "posts",
    // 'posts'로 시작하는 모든 쿼리 무효화
  });
}
```

`predicate` 옵션 : 이 조건에 맞는 쿼리들만 골라서 무효화하고 싶을 때 사용하는 더 세밀한 필터링 방법.

언제 호출 ? : 뮤테이션 성공 시점

```jsx
const mutation = useMutation({
  mutationFn: async (newPost) => {
    const response = await fetch("/api/posts", {
      method: "POST",
      body: JSON.stringify(newPost),
    });

    if (!response.ok) {
      throw new Error("게시글 생성 실패");
    }

    return response.json(); // 여기서 성공적으로 반환되면
  },
  onSuccess: (data) => {
    // 이 시점이 "뮤테이션 성공 시점"
    // 서버에서 정상적으로 응답이 왔고, 에러가 없는 상태
    console.log("서버에 데이터가 성공적으로 저장됨:", data);

    // 이때 invalidateQueries를 호출해서 다른 쿼리들을 최신으로 업데이트
    queryClient.invalidateQueries({ queryKey: ["posts"] });
  },
  onError: (error) => {
    // 서버 에러가 발생한 경우는 여기서 처리
    console.error("저장 실패:", error);
    // invalidateQueries 호출하지 않음 (실패했으니까)
  },
});
```

#### `mutate()` 함수의 콜백 옵션

onSuccess, onError, onSettled와 같은 옵션은 `useMutation()`에서도 사용할 수 있고, `mutate()` 함수에서도 사용할 수 있다. 이때, `useMutation()`에 등록한 콜백 함수들이 먼저 실행되고, 그다음에 `mutate()`에 등록한 콜백 함수들이 실행됨.

여기서 주의할 점은, `useMutation()`에 등록된 콜백 함수들은 컴포넌트가 언마운트되더라도 실행이 되지만, `mutate()`의 콜백 함수들은 뮤테이션이 끝나기 전에 언마운트되면 실행이 안됨. 그래서, `query invalidation`과 같이 뮤테이션 과정에서 꼭 필요한 로직은 `useMutation()`을 통해 등록하고, 그 외에 다른 페이지 리다이렉트 또는 결과를 토스트로 띄워주는 것들은 `mutate()`를 통해 등록.

여기서 onSettled 란 ? : 뮤테이션이 성공하든 실패하든 상관없이 무조건 실행되는 콜백

ex. 컴포넌트 언마운트 시 차이점

```jsx
function CreatePostPage() {
  const navigate = useNavigate();

  const mutation = useMutation({
    mutationFn: createPost,
    onSuccess: () => {
      // ✅ 컴포넌트가 언마운트되어도 실행됨 (안전함)
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    },
  });

  const handleSubmit = (formData) => {
    mutation.mutate(formData, {
      onSuccess: () => {
        // ⚠️ 컴포넌트가 언마운트되면 실행 안 될 수 있음
        navigate("/posts");
        toast.success("게시글이 생성되었습니다!");
      },
    });
  };

  return <form onSubmit={handleSubmit}>{/* 폼 내용 */}</form>;
}
```

ex2. 권장 사용법

```jsx
const mutation = useMutation({
  mutationFn: createPost,
  // 🔥 중요한 시스템 로직은 여기에
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["posts"] });
  },
  onError: (error) => {
    // 에러 로깅 등 중요한 작업
    console.error("게시글 생성 실패:", error);
  },
});

const handleSubmit = (formData) => {
  mutation.mutate(formData, {
    // 🎨 UI 관련 로직은 여기에
    onSuccess: () => {
      navigate("/posts");
      toast.success("게시글이 생성되었습니다!");
    },
    onError: () => {
      toast.error("게시글 생성에 실패했습니다.");
    },
    onSettled: () => {
      setIsSubmitting(false); // 로딩 상태 해제
    },
  });
};
```

### `Dependant Query`

: 어떤 특정 값이나 조건이 충족된 이후에 실행되는 쿼리.

첫 번째 쿼리의 결과가 나와야만 두 번째 쿼리를 실행할 수 있는 상황에서 사용함.

#### 왜 필요한가 ?

문제상황

ex.

```jsx
// ❌ 잘못된 예시 - userId가 없는데 쿼리 실행
const { data: user } = useQuery({
  queryKey: ["user", email],
  queryFn: getUserByEmail,
});

const { data: projects } = useQuery({
  queryKey: ["projects", user?.id],
  queryFn: () => getProjectsByUser(user?.id), // user?.id가 undefined일 수 있음!
});

// 결과: user 데이터가 로드되기 전에 getProjectsByUser(undefined) 호출 → 에러!
```

해결방법

ex. enabled 사용

```jsx
const { data: user } = useQuery({
  queryKey: ["user", email],
  queryFn: getUserByEmail,
});

const userId = user?.id;

const { data: projects } = useQuery({
  queryKey: ["projects", userId],
  queryFn: getProjectsByUser,
  enabled: !!userId,
});
```

### 페이지네이션에서 사용되는 `placeholderData`, `prefetchQuery`

페이지네이션 구현할 때, page와 limit을 넘겨주는 API로 설계되어있을거임.

ex.

```jsx
// api.js에서

export async function getPosts(page = 0, limit = 10) {
  const response = await fetch(`${BASE_URL}/posts?page=${page}&limit=${limit}`);
  return await response.json();
}
```

```jsx
// Homepage.js에서

import { useState } from "react";
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { getPosts, uploadPost, getUserInfo } from "./api";

const PAGE_LIMIT = 3;

function HomePage() {
  // ...
  const [page, setPage] = useState(0);
  const {
    data: postsData,
    isPending,
    isError,
  } = useQuery({
    queryKey: ["posts", page],
    queryFn: () => getPosts(page, PAGE_LIMIT),
  });

  // ...

  const posts = postsData?.results ?? [];

  if (isPending) {
    return <div>로딩 중...</div>;
  }

  if (isError) {
    return <div>에러가 발생했습니다.</div>;
  }

  return (
    <>
      <div>
        {currentUsername ? (
          loginMessage
        ) : (
          <button onClick={handleLoginButtonClick}>codeit으로 로그인</button>
        )}
        <form onSubmit={handleSubmit}>
          <textarea
            name="content"
            value={content}
            onChange={handleInputChange}
          />
          <button disabled={!content} type="submit">
            업로드
          </button>
        </form>
      </div>
      <div>
        <ul>
          {posts.map((post) => (
            <li key={post.id}>
              {post.user.name}: {post.content}
            </li>
          ))}
        </ul>
        <div>
          <button
            disabled={page === 0}
            onClick={() => setPage((old) => Math.max(old - 1, 0))}
          >
            &lt;
          </button>
          <button
            disabled={!postsData?.hasMore}
            onClick={() => setPage((old) => old + 1)}
          >
            &gt;
          </button>
        </div>
      </div>
    </>
  );
}

export default HomePage;
```

=> 근데 이렇게하면, 다음 페이지로 넘어갈 때마다 로딩 메시지가 뜸. 왜냐면 새로운 페이지에 해당하는 쿼리를 보낼 때마다 완전히 새로운 쿼리로 인식을 하기 때문. (queryKey가 ["posts", 0] -> ["posts", 1] 이런식으로 변경되기 때문. ) <br />
이럴 때 사용하는게 `placeholderData`

`placeholderData` 옵션에 `keepPreviousData` 혹은 `(prevData) => prevData`를 넣어주면 페이지가 바뀌더라도 매번 `pending` 상태가 되지 않고, 이전의 데이터를 유지해서 보여주다가 새로운 데이터 fetch가 완료되면 자연스럽게 새로운 데이터로 바꿔서 보여줌.

ex) 현재 보이는 데이터가 이전 데이터, 즉 placeholderData면 다음 페이지 버튼 비활성화

```jsx
const {
  data: postsData,
  isPending,
  isError,
  isPlaceholderData,
} = useQuery({
  queryKey: ['posts', page],
  queryFn: () => getPosts(page, PAGE_LIMIT),
  placeholderData: keepPreviousData,
});

...

return (
  ...
    <div>
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{`${post.user.name}: ${post.content}`}</li>
      ))}
    </ul>
    <div>
      <button
        disabled={page === 0}
        onClick={() => setPage((old) => Math.max(old - 1, 0))}
      >
        &lt;
      </button>
      <button
        disabled={isPlaceholderData || !postsData?.hasMore}
        onClick={() => setPage((old) => old + 1)}
      >
        &gt;
      </button>
    </div>
  </div>
);
```

#### `prefetchQuery`

- 다음 페이지 데이터를 미리 백그라운드에서 가져옴
- 사용자가 다음 페이지로 넘어갈 때 즉시 표시 가능

prefetch 예시)

```jsx
useEffect(() => {
  if (!isPlaceholderData && postsData?.hasMore) {
    // 현재 표시 중인 데이터가 실제 데이터이고 + 다음 페이지가 있으면
    queryClient.prefetchQuery({
      queryKey: ["posts", page + 1], // 다음 페이지 데이터를
      queryFn: () => getPosts(page + 1, PAGE_LIMIT), // 미리 가져와서 캐시에 저장
    });
  }
}, [isPlaceholderData, postsData, queryClient, page]);
```

### Infinite Query : `useInfiniteQuery()`

기존의 `useQuery()`를 `useInfiniteQuery()`로 바꾸고, `initialPageParam`과 `getNextPageParam` 옵션을 설정해줘야함.

- `initialPageParam`: 첫 페이지 파라미터(오프셋 0, 혹은 null 커서 등)

- `getNextPageParam`(lastPage, allPages, lastPageParam, allPageParams): 마지막 응답을 보고 다음 `pageParam` 반환. 반환값이 `undefined`면 `hasNextPage=false`.

ex.

```jsx
import { useInfiniteQuery } from "@tanstack/react-query";
// import 나머지들 ~~

const {
  data: postsData,
  isPending,
  isError,
  hasNextPage,
  fetchNextPage,
  isFetchingNextPage,
} = useInfiniteQuery({
  queryKey: ["posts"],
  queryFn: ({ pageParam }) => getPosts(pageParam, PAGE_LIMIT),
  initialPageParam: 0,
  getNextPageParam: (lastPage, allPages, lastPageParam, allPageParams) =>
    lastPage.hasMore ? lastPageParam + 1 : undefined, // 다음 페이지가 있으면 다음 페이지 번호를, 없으면 undefined를 반환
});

// ...

return (
  <div>
    {postsData?.pages.map((page, pageIndex) => (
      <div key={pageIndex}>
        {page.data.map((post) => (
          <div key={post.id}>{post.content}</div>
        ))}
      </div>
    ))}

    {isPending && <div>로딩 중...</div>}
    {isFetchingNextPage && <div>추가 데이터 로딩 중...</div>}

    <button onClick={fetchNextPage}>더 불러오기</button>
  </div>
);
```

- `lastPage`: 가장 최근에 가져온 페이지 데이터
- `allPages`: 지금까지 가져온 모든 페이지 데이터 배열
- `lastPageParam`: 가장 최근 페이지 요청에 사용된 파라미터
- `allPageParams`: 모든 페이지 파라미터 배열

### 낙관적 업데이트

서버 응답을 기다리지 않고 UI를 먼저 업데이트하여 사용자 경험을 향상시키는 패턴.

**동작 과정:**

1. 사용자 액션 발생 (예: 좋아요 버튼 클릭)
2. UI를 즉시 업데이트 (낙관적으로 성공할 것이라 가정)
3. 백그라운드에서 서버 요청 실행
4. 성공: 서버 데이터로 동기화
5. 실패: 이전 상태로 롤백

**장점:** 빠른 사용자 피드백, 네트워크 지연 감춤 <br />
**단점:** 서버 오류 시 롤백 처리 필요

**언제 사용하면 좋은가** ? : 네트워크가 느린 환경, 실시간성이 중요한 기능(좋아요, 팔로우 등)

**주의사항**: 서버 검증이 중요한 기능에서는 신중하게 사용

**에러 처리**: toast 메시지나 사용자 알림 방법

ex.

```jsx
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import {
  getLikeCountByPostId,
  getLikeStatusByUsername,
  likePost,
  unlikePost,
} from "./api";

function Post({ post, currentUsername }) {
  const queryClient = useQueryClient();

  const { data: likeCount } = useQuery({
    queryKey: ["likeCount", post.id],
    queryFn: () => getLikeCountByPostId(post.id),
  });

  const { data: isPostLikedByCurrentUser } = useQuery({
    queryKey: ["likeStatus", post.id, currentUsername],
    queryFn: () => getLikeStatusByUsername(post.id, currentUsername),
    enabled: !!currentUsername,
  });

  const likesMutation = useMutation({
    mutationFn: async ({ postId, username, userAction }) => {
      if (userAction === "LIKE_POST") {
        await likePost(postId, username);
      } else {
        await unlikePost(postId, username);
      }
    },
    onMutate: async ({ postId, username, userAction }) => {
      await queryClient.cancelQueries({
        queryKey: ["likeStatus", postId, username],
      });
      await queryClient.cancelQueries({ queryKey: ["likeCount", postId] });

      const prevLikeStatus = queryClient.getQueryData([
        "likeStatus",
        postId,
        username,
      ]);
      const prevLikeCount = queryClient.getQueryData(["likeCount", postId]);

      queryClient.setQueryData(
        ["likeStatus", postId, username],
        userAction === "LIKE_POST"
      );
      queryClient.setQueryData(["likeCount", postId], (prev) =>
        userAction === "LIKE_POST"
          ? (prev || 0) + 1
          : Math.max((prev || 0) - 1, 0)
      );

      return { prevLikeStatus, prevLikeCount };
    },
    onError: (err, { postId, username }, context) => {
      queryClient.setQueryData(
        ["likeStatus", postId, username],
        context.prevLikeStatus
      );
      queryClient.setQueryData(["likeCount", postId], context.prevLikeCount);
    },
    onSettled: (data, err, { postId, username }) => {
      queryClient.invalidateQueries({
        queryKey: ["likeStatus", postId, username],
      });
      queryClient.invalidateQueries({
        queryKey: ["likeCount", postId],
      });
    },
  });

  const handleLikeButtonClick = (userAction) => {
    console.log("@@@here", currentUsername);
    if (!currentUsername) return;
    likesMutation.mutate({
      postId: post.id,
      username: currentUsername,
      userAction,
    });
  };

  return (
    <li>
      <div>
        {post.user.name}: {post.content}
      </div>
      <button
        onClick={() =>
          handleLikeButtonClick(
            isPostLikedByCurrentUser ? "UNLIKE_POST" : "LIKE_POST"
          )
        }
      >
        {isPostLikedByCurrentUser ? "♥️ " : "♡ "}
        {`좋아요 ${likeCount ?? 0}개`}
      </button>
    </li>
  );
}

export default Post;
```

해설

`코드 역할`

① useQueryClient() 전역 캐시 매니저 핸들. 이걸로 캐시 읽기·쓰기·무효화·취소 전부 수행 <br />
② likeCount 쿼리 ['likeCount', post.id] 키 기준으로 개수 가져와 캐싱 <br />
③ likeStatus 쿼리 ['likeStatus', post.id, currentUsername] 키(=유저별)로 내가 눌렀는지 조회 <br />
enabled: !!currentUsername → 로그인 안 했으면 호출 안 함 <br />
④ mutationFn LIKE_POST ↔ UNLIKE_POST 구분해 실제 서버 요청 수행 <br />
⑤ onMutate 낙관적 단계 <br />

1. 🔕 cancelQueries – 같은 키의 refetch 일시중지
2. 📸 getQueryData – 이전 캐시 스냅샷 확보
3. ✍️ setQueryData – UI를 즉시 바꿈(개수 ±1, 상태 T/F)
4. return { prev… } – 롤백용 데이터 전달

⑥ onError 서버 오류 시 스냅샷으로 되돌리기 (setQueryData 두 번) <br />
⑦ onSettled 성공·실패 모두 무효화(invalidateQueries) → 서버 정본으로 동기화 <br />
⑧ handleLikeButtonClick 버튼 클릭→ mutate({...}) 호출. <br/>userAction을 변수로 넘겨서 위 콜백들이 동일하게 사용 <br />

### Next.js에서 React Query 사용하기

1. App 컴포넌트에서 초기설정

```jsx
// _app.tsx 에서

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

export default function App({ Component, pageProps }) {
  const [queryClient] = React.useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            // 보통 SSR에서는 staleTime을 0 이상으로 해줌으로써
            // 클라이언트 사이드에서 바로 다시 데이터를 refetch 하는 것을 피한다.
            staleTime: 60 * 1000,
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      <Component {...pageProps} />
    </QueryClientProvider>
  );
}
```

1-2 App Router : `app/providers.tsx` (Client 컴포넌트) 로 분리해 재사용.

```jsx
// app/providers.tsx (client)
"use client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { useState } from "react";

export default function ReactQueryProvider({
  children,
}: {
  children: React.ReactNode,
}) {
  const [client] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60_000, // SSR 후 재요청 방지
            refetchOnWindowFocus: false, // UX 필요에 따라
          },
        },
      })
  );
  return (
    <QueryClientProvider client={client}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

```jsx
// app/layout.tsx (server)
import ReactQueryProvider from "./providers";
export default function RootLayout({
  children,
}: {
  children: React.ReactNode,
}) {
  return (
    <html lang="ko">
      <body>
        <ReactQueryProvider>{children}</ReactQueryProvider>
      </body>
    </html>
  );
}
```

### 2. 2가지 방법으로 prefetch 구현하기.

### 2-1. initialData 사용

```jsx
export async function getServerSideProps() {
  const posts = await getPosts();
  return { props: { posts } };
}

function Posts(props) {
  const { data } = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    initialData: props.posts,
  });

  // ...
}
```

장점

1. `prefetching` 단계에서는 리액트 쿼리를 전혀 사용하지 않아도 됨.

```jsx
// getServerSideProps에서는 일반적인 데이터 fetching
export async function getServerSideProps() {
  // 그냥 fetch나 axios 사용하면 됨
  const response = await fetch("https://api.example.com/posts");
  const posts = await response.json();

  return { props: { posts } };
}
```

단점

1. `getStaticProps()`, `getServerSideProps()`는 `pages` 폴더 안에서만 동작하기 때문에 `useQuery()`를 사용하려는 컴포넌트까지 `prefetch`한 데이터를 `props drilling`으로 내려줘야만 한다.

```jsx
// pages/posts.tsx
export async function getServerSideProps() {
  const posts = await getPosts();
  return { props: { posts } };
}

function PostsPage({ posts }) {
  return (
    <div>
      <Header />
      <PostList posts={posts} /> {/* props로 전달 */}
    </div>
  );
}

// components/PostList.tsx
function PostList({ posts }) {
  return (
    <div>
      <PostFilter />
      <PostItems posts={posts} /> {/* 또 props로 전달 */}
    </div>
  );
}

// components/PostItems.tsx
function PostItems({ posts }) {
  const { data } = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    initialData: posts, // 여기서 finally 사용
  });

  return (
    <div>
      {data.map((post) => (
        <PostItem key={post.id} post={post} />
      ))}
    </div>
  );
}
```

2. 쿼리의 `useQuery()`를 여러 군데서 사용한다면, 모든 `useQuery()`에 똑같은 `initialData`를 설정해줘야함.

```jsx
// components/PostList.tsx
function PostList({ posts }) {
  const { data } = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    initialData: posts, // 같은 initialData
  });
  // ...
}

// components/PostSidebar.tsx
function PostSidebar({ posts }) {
  const { data } = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    initialData: posts, // 또 같은 initialData
  });
  // ...
}

// components/PostHeader.tsx
function PostHeader({ posts }) {
  const { data } = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    initialData: posts, // 또또 같은 initialData
  });
  // ...
}
```

3. `initialData`는 기존 캐시를 덮어쓰지 않음. 따라서 이미 오래된 데이터가 캐시에 있어도, 서버에서 가져온 최신 데이터로 업데이트되지 않을 수 있다.

```jsx
// 시나리오: 사용자가 페이지를 여러 번 방문하는 상황

// 1단계: 첫 방문 (10:00)
function PostsPage({ posts }) {
  // posts: 서버에서 가져온 최신 데이터 (10:00)
  const { data } = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    initialData: posts,
  });
  // 캐시에 10:00 데이터 저장됨
}

// 2단계: 다른 페이지로 이동 후 다시 돌아옴 (10:30)
// 이때 서버에서는 10:30 최신 데이터를 가져왔지만...

function PostsPage({ posts }) {
  // posts: 서버의 최신 데이터 (10:30)
  const { data } = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    initialData: posts, // 10:30 최신 데이터
  });

  // 하지만 실제로 보여지는 data는 캐시에 있던 10:00 데이터!
  // initialData는 캐시가 비어있을 때만 사용되기 때문
}
```

+) initialData가 있으면 쿼리는 즉시 status: 'success'가 되고, staleTime이 지나기 전엔 새로고침(refetchOnMount)이 안 일어남.

```jsx
function PostsPage({ posts }) {
  const { data, status, isStale } = useQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
    initialData: posts,
    staleTime: 5 * 60 * 1000, // 5분
  });

  console.log(status); // 즉시 'success'
  console.log(isStale); // false (staleTime 때문에)

  // staleTime이 지나기 전까지는 백그라운드에서
  // 새로운 데이터를 가져오지 않음
  // 서버 데이터가 업데이트되어도 모름!
}
```

### 2-2. Hydration API 사용하기

: 서버에서 데이터를 미리 가져와서 React Query 캐시에 저장한 뒤, 그 캐시 상태를 클라이언트로 전달하여 "수화(hydration)"하는 방식

- Dehydrate (탈수): 물을 빼서 말리는 것 → 데이터를 직렬화해서 전송 가능한 형태로 만드는 것

```jsx
// 서버에서
const qc = new QueryClient();
await qc.prefetchQuery({ queryKey: ["posts"], queryFn: getPosts });

// 이 시점의 QueryClient 내부 상태:
// qc.cache = {
//   "posts": {
//     data: [{ id: 1, title: "Post 1" }, { id: 2, title: "Post 2" }],
//     status: "success",
//     dataUpdatedAt: 1640995200000,
//     // ... 기타 메타데이터
//   }
// }

// dehydrate = "물을 빼서 말리기" = 직렬화
const dehydratedState = dehydrate(qc);

// 결과물 (JSON 형태):
// {
//   mutations: [],
//   queries: [
//     {
//       queryKey: ["posts"],
//       queryHash: '"posts"',
//       state: {
//         data: [{ id: 1, title: "Post 1" }, { id: 2, title: "Post 2" }],
//         dataUpdatedAt: 1640995200000,
//         status: "success"
//       }
//     }
//   ]
// }
```

- Hydrate (수화): 물을 다시 넣어서 원래 상태로 되돌리는 것 → 직렬화된 데이터를 다시 활성 상태로 복원하는 것

```jsx
// 클라이언트에서
<HydrationBoundary state={dehydratedState}>
  <Posts />
</HydrationBoundary>

// HydrationBoundary가 하는 일:
// 1. dehydratedState (JSON)를 받음
// 2. 클라이언트의 QueryClient 캐시에 복원
// 3. 마치 클라이언트에서 직접 쿼리를 실행한 것처럼 상태 재구성
```

ex.

```jsx
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
  useQuery,
} from "@tanstack/react-query";

export async function getStaticProps() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery({
    queryKey: ["posts"],
    queryFn: getPosts,
  });

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
}

function Posts() {
  const { data } = useQuery({ queryKey: ["posts"], queryFn: getPosts });

  // 이 쿼리는 서버에서 prefetch하지 않는 데이터.
  // prefetch하는 데이터와 아닌 데이터를 자유롭게 섞어서 활용할 수 있다.
  const { data: commentsData } = useQuery({
    queryKey: ["posts-comments"],
    queryFn: getComments,
  });

  // ...
}

export default function PostsRoute({ dehydratedState }) {
  return (
    <HydrationBoundary state={dehydratedState}>
      <Posts />
    </HydrationBoundary>
  );
}
```

=> prefetch한 결괏값이 담긴 queryClient를 dehydrate해서 클라이언트로 보내 주기.
이렇게 하면 초기 설정 코드가 늘어나지만 initialData를 이용하면서 발생하는 여러 단점을 모두 해결할 수 있음.

나열해보자면,

1. Props drilling 불필요 - 컴포넌트 어디서든 useQuery로 접근 가능
2. 캐시 일관성 보장 - 서버와 클라이언트 상태 완벽 동기화
3. 선택적 prefetch 가능 - 필요한 쿼리만 미리 로드
4. 기존 캐시와 자연스럽게 통합

#### `두 가지 실무 옵션`

1. 어떤 쿼리만 탈수할지 선택

```tsx
const state = dehydrate(queryClient, {
  shouldDehydrateQuery: (q) => q.state.status === "success", // 성공만 포함
});
```

2. 무한 쿼리 사전 패치
   무한 스크롤이면 `prefetchInfiniteQuery`를 사용:

```tsx
await queryClient.prefetchInfiniteQuery({
  queryKey: ["posts"],
  queryFn: ({ pageParam = 0 }) => getPosts(pageParam, 20),
  initialPageParam: 0,
});
```

#### Pages Router 예시

```tsx
// pages/posts.tsx
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
  useQuery,
} from "@tanstack/react-query";
import { getPosts, getComments } from "@/lib/api";

export async function getServerSideProps() {
  const qc = new QueryClient(); // 서버용 QueryClient 생성
  await qc.prefetchQuery({ queryKey: ["posts"], queryFn: getPosts }); // 서버에서 미리 데이터를 가져와 캐시에 저장
  return { props: { dehydratedState: dehydrate(qc) } }; // 캐시 상태를 직렬화해서 클라이언트로 전달
}

function Posts() {
  const { data } = useQuery({ queryKey: ["posts"], queryFn: getPosts });
  const { data: comments } = useQuery({
    queryKey: ["posts-comments"],
    queryFn: getComments,
  });
  return /* ... */;
}

export default function PostsRoute({ dehydratedState }) {
  return (
    <HydrationBoundary state={dehydratedState}>
      <Posts />
    </HydrationBoundary>
  );
}
```

사용

```jsx
function Posts() {
  // 이미 캐시에 데이터가 있어서 즉시 반환됨
  const { data } = useQuery({ queryKey: ["posts"], queryFn: getPosts });

  // 이 쿼리는 prefetch 안했으므로 클라이언트에서 새로 요청
  const { data: comments } = useQuery({
    queryKey: ["posts-comments"],
    queryFn: getComments,
  });

  return /* ... */;
}
```

#### App Router 예시

- 서버 컴포넌트에서 미리 패치 → 상태만 클라이언트로 전달 → Client Provider + HydrationBoundary로 감싸기.

```tsx
// app/posts/page.tsx (server)
import { dehydrate, QueryClient } from "@tanstack/react-query";
import ReactQueryProvider from "../providers";
import Posts from "./posts-client"; // client 컴포넌트

export default async function Page() {
  const qc = new QueryClient();
  await qc.prefetchQuery({ queryKey: ["posts"], queryFn: getPosts }); // 서버에서 데이터 prefetch

  const dehydratedState = dehydrate(qc, {
    shouldDehydrateQuery: (q) => q.state.status === "success",
  }); // 성공한 쿼리만 dehydrate (옵션)

  return (
    <ReactQueryProvider>
      {/* HydrationBoundary는 client에서 사용 */}
      {/* 간단히 props로 넘겨서 내부에서 감싸도 OK */}
      <Posts dehydratedState={dehydratedState} />
    </ReactQueryProvider>
  );
}
```

```tsx
// app/posts/posts-client.tsx (client)
"use client";
import { HydrationBoundary, useQuery } from "@tanstack/react-query";

export default function PostsClient({
  dehydratedState,
}: {
  dehydratedState: unknown;
}) {
  return (
    <HydrationBoundary state={dehydratedState}>
      <PostsInner />
    </HydrationBoundary>
  );
}

function PostsInner() {
  const { data } = useQuery({ queryKey: ["posts"], queryFn: getPosts });
  return /* ... */;
}
```

**언제 어떤 방법을 사용할까?**

**initialData 사용:**

- 간단한 페이지 수준 데이터
- 한 번만 사용되는 데이터
- 빠른 프로토타이핑

**Hydration API 사용:**

- 복잡한 애플리케이션
- 여러 컴포넌트에서 같은 데이터 사용
- 실시간 업데이트가 중요한 경우

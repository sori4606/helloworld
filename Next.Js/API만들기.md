## Next.js API 만들기

### 리퀘스트 핸들러 함수

: Next.js에서 API 라우트를 만들 때 사용하는 함수 <br />
클라이언트(브라우저나 앱 등)에서 `/api/short-links`와 같은 경로로 HTTP 요청을 보내면, 이 요청을 처리하는 함수가 필요한데, 이걸 리퀘스트 핸들러 함수라고 함 <br />
즉, 특정 URL과 HTTP 메서드로 들어온 요청을 받아서, 무엇을 할지 결정하고 응답을 돌려주는 함수.

#### 어디에 두면 동작 ?

- Pages Router : `/pages/api/**.js` -> 자동으로 `/api/**` 경로에 매핑
- App Router : `/app/api/**/route.ts` -> 파일 경로가 그대로 API URL(`/api/**`)이 되고, 핸들러는 Node의 `req/res`가 아니라 브라우저 표준 Fetch API의 `Request/Response`로 다룬다.

#### 파일 -> URL 매핑 예시.

| 파일 경로 (App Router)             | 실제 API URL                                    |
| ---------------------------------- | ----------------------------------------------- |
| `app/api/ping/route.ts`            | `/api/ping`                                     |
| `app/api/users/route.ts`           | `/api/users`                                    |
| `app/api/users/[id]/route.ts`      | `/api/users/:id` (예: `/api/users/42`)          |
| `app/api/posts/[...slug]/route.ts` | `/api/posts/**` (캐치올: `/api/posts/a/b/c`)    |
| `app/api/(group)/reports/route.ts` | `/api/reports` _(괄호 폴더는 URL에 포함 안 됨)_ |

`캐치올` : 경로의 남은 부분을 전부 한 번에 받는 동적 세그먼트 <br />

파일 위치

```
/pages/api/short-links.js
OR
/pages/api/short-links/index.js
```

ex.

```js
export default async function handler(req, res) {
  // req : 들어온 요청 (request 객체)
  // res : 응답을 보낼 때 사용하는 (response 객체)

  // 예시: GET 요청 처리
  if (req.method === "GET") {
    res.status(200).json({ message: "GET 요청 처리 완료" });
  }

  // 예시: POST 요청 처리
  else if (req.method === "POST") {
    const data = req.body;
    res.status(201).json({ message: "POST 요청 처리 완료", data });
  }

  // 그 외 메서드는 허용 안 함
  else {
    res.status(405).json({ message: "허용되지 않은 메서드입니다." });
  }
}
```

#### `리퀘스트 객체`

| 속성    | 타입   | 설명                                            |
| ------- | ------ | ----------------------------------------------- |
| method  | string | HTTP 메소드 ('GET', 'POST', 'PUT', 'DELETE' 등) |
| query   | object | 쿼리 스트링과 동적 라우트 파라미터              |
| body    | any    | 요청 본문 데이터                                |
| cookies | object | 쿠키 값들이 키/밸류로 저장된 객체               |
| headers | object | HTTP 헤더 정보                                  |
| url     | string | 요청 URL                                        |

`ex`

```js
// 요청: GET /api/users/123?page=1&limit=10
// 쿠키: token=abc123
// 헤더: Authorization: Bearer xyz789

export default function handler(req, res) {
  console.log("Method:", req.method); // 'GET'
  console.log("Query:", req.query); // { id: '123', page: '1', limit: '10' }
  console.log("Cookies:", req.cookies); // { token: 'abc123' }
  console.log("Headers:", req.headers); // { authorization: 'Bearer xyz789', ... }
  console.log("URL:", req.url); // '/api/users/123?page=1&limit=10'

  // POST 요청시
  console.log("Body:", req.body); // 클라이언트에서 보낸 데이터
}
```

#### `리스폰스 객체`

| 메서드                 | 설명                  | 예시                                       |
| ---------------------- | --------------------- | ------------------------------------------ |
| status(code)           | HTTP 상태 코드 설정   | res.status(200)                            |
| json(data)             | JSON 응답 전송        | res.json({ success: true })                |
| send(data)             | 텍스트/HTML 응답 전송 | res.send(`<h1>Hello</h1>`)                 |
| redirect(url)          | 리다이렉트            | res.redirect('/login')                     |
| setHeader(name, value) | 응답 헤더 설정        | res.setHeader('Cache-Control', 'no-cache') |
| end()                  | 응답 종료             | res.end()                                  |

`ex`

```js
export default function handler(req, res) {
  try {
    if (req.method === "GET") {
      // 성공
      res.status(200).json({ message: "조회 성공" });
    }

    if (req.method === "POST") {
      // 생성 성공
      res.status(201).json({ message: "생성 성공" });
    }

    if (req.method === "PUT") {
      // 업데이트 성공 (데이터 반환)
      res.status(200).json({ message: "업데이트 성공" });
      // 또는 업데이트 성공 (데이터 반환 없음)
      res.status(204).end();
    }
  } catch (error) {
    // 서버 에러
    res.status(500).json({ error: "서버 에러가 발생했습니다." });
  }
}
```

### 동적 라우팅

#### `파일 구조`

```
/pages/api/
  ├── users/
  │   ├── index.js          // /api/users
  │   ├── [id].js           // /api/users/123
  │   └── [id]/posts.js     // /api/users/123/posts
  ├── posts/
  │   └── [...slug].js      // /api/posts/2023/12/25 (catch-all)
  └── [[...params]].js      // optional catch-all
```

`ex`

```js
// /pages/api/users/[id].js
export default function handler(req, res) {
  const { id } = req.query;  // URL에서 id 파라미터 추출

  if (req.method === 'GET') {
    // GET /api/users/123
    res.status(200).json({ userId: id, name: 'John Doe' });
  }

  if (req.method === 'DELETE') {
    // DELETE /api/users/123
    res.status(200).json({ message: `User ${id} deleted` });
  }
}

// /pages/api/posts/[...slug].js
export default function handler(req, res) {
  const { slug } = req.query;
  // /api/posts/2023/12/25 -> slug = ['2023', '12', '25']

  res.status(200).json({ path: slug });
}
```

### 에러 처리

#### 기본 에러 처리

`ex`

```js
export default async function handler(req, res) {
  try {
    // 메서드 체크
    if (!["GET", "POST"].includes(req.method)) {
      return res.status(405).json({ error: "허용되지 않은 메서드" });
    }

    // 인증 체크
    const token = req.headers.authorization;
    if (!token) {
      return res.status(401).json({ error: "인증 토큰이 필요합니다" });
    }

    // 데이터 검증
    if (req.method === "POST") {
      const { name, email } = req.body;
      if (!name || !email) {
        return res.status(400).json({
          error: "필수 필드가 누락되었습니다",
          required: ["name", "email"],
        });
      }
    }

    // 비즈니스 로직 실행
    const result = await processData(req.body);
    res.status(200).json(result);
  } catch (error) {
    console.error("API Error:", error);

    // 알려진 에러 타입별 처리
    if (error.name === "ValidationError") {
      return res.status(400).json({ error: "데이터 검증 실패" });
    }

    if (error.name === "UnauthorizedError") {
      return res.status(401).json({ error: "권한이 없습니다" });
    }

    // 기본 서버 에러
    res.status(500).json({ error: "서버 에러가 발생했습니다" });
  }
}
```

#### 미들웨어 패턴

`미들웨어` : 서버에서 요청(Request) 과 응답(Response) 사이에 끼어,
공통 로직을 처리하는 중간 처리기 역할을 하는 함수. 실제 비즈니스 로직이 실행되기 전에 공통적으로 처리해야 할 작업들을 담당함.

#### 1. 공통 로직 분리

: 먼저 재사용할 미들웨어 함수들을 별도 파일로 분리한다.

```js
// /lib/middleware.js
export function cors(req, res, next) {
  // 모든 도메인에서 접근 허용 (프로덕션에서는 특정 도메인으로 제한을 권장한다)
  res.setHeader("Access-Control-Allow-Origin", "*");

  // 허용할 HTTP 메서드 지정
  res.setHeader("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE");

  // 허용할 헤더 지정
  res.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");

  if (req.method === "OPTIONS") {
    res.status(200).end();
    return;
  }

  next(); // 다음 미들웨어 또는 핸들러로 진행
}

// 인증 미들웨어 - JWT 토큰 검증
export function auth(req, res, next) {
  const token = req.headers.authorization;

  if (!token) {
    return res.status(401).json({ error: "토큰이 필요합니다" });
  }

  // 토큰 검증 로직
  try {
    const user = verifyToken(token); // JWT 토큰 검증 함수
    req.user = user; // 검증된 사용자 정보를 request 객체에 추가
    next(); // 인증 성공 시 다음 단계로
  } catch (error) {
    return res.status(401).json({ error: "유효하지 않은 토큰" });
  }
}

// 미들웨어 실행 헬퍼
// Next.js에서 미들웨어를 Promise 형태로 실행
export function runMiddleware(req, res, fn) {
  return new Promise((resolve, reject) => {
    fn(req, res, (result) => {
      // 이 콜백 함수가 next()입니다
      if (result instanceof Error) {
        return reject(result);
      }
      return resolve(result); // ← next() 호출되면 여기가 실행됨
    });
  });
}
```

#### 2. 미들웨어 사용

```js
// /pages/api/protected-route.js
import { cors, auth, runMiddleware } from "../../lib/middleware";

export default async function handler(req, res) {
  try {
    // 1. CORS 미들웨어 실행 (교차 출처 요청 허용)
    await runMiddleware(req, res, cors);

    // 2. 인증 미들웨어 실행 (JWT 토큰 검증)
    await runMiddleware(req, res, auth);

    // 3. 미들웨어 실행 완료 후, req.user에 인증된 사용자 정보가 있음
    res.status(200).json({
      message: "인증된 사용자만 볼 수 있습니다",
      user: req.user, // 인증 미들웨어에서 설정한 사용자 정보
    });
  } catch (error) {
    // 미들웨어에서 에러가 발생한 경우
    console.error("Middleware Error:", error);
    res.status(500).json({ error: "서버 에러가 발생했습니다" });
  }
}
```

- `next()` 는 어디로 넘어가는가 ? : 다음에 실행될 함수로 넘어감.
  1. `cors` 미들웨어에서 `next()` 호출 → `runMiddleware` Promise가 resolve됨 (= `await runMiddleware(...)`가 완료되어서 다음 줄로 넘어감)
  2. `auth` 미들웨어에서 `next()` 호출 → `runMiddleware` Promise가 resolve됨

`next()`가 없으면 ?

- `cors()`에서 `next()` 안 쓰면 auth 미들웨어까지 못감.
- 토큰 검증 자체가 실행되지 않고
- 클라이언트는 응답을 받지 못해 타임아웃이 발생함.

그래서, 미들웨어에서는 반드시 `next()`를 호출하거나, 아니면 직접 `res.json()` 등으로 응답을 완료해야 한다 !

`실행 과정 분석`

```js
export default async function handler(req, res) {
  // 1단계
  await runMiddleware(req, res, cors);
  console.log("CORS 완료!"); // ← 이게 언제 실행될까?

  // 2단계
  await runMiddleware(req, res, auth);
  console.log("Auth 완료!"); // ← 이게 언제 실행될까?

  // 3단계
  res.status(200).json({ message: "성공!" });
}
```

```js
console.log("시작"); // 1번

await runMiddleware(req, res, cors); // 2번 - 여기서 대기
// cors 함수 실행 → next() 호출 → Promise resolve
// ↑ resolve되면 await 완료!

console.log("CORS 완료!"); // 3번

await runMiddleware(req, res, auth); // 4번 - 여기서 대기
// auth 함수 실행 → next() 호출 → Promise resolve
// ↑ resolve되면 await 완료!

console.log("Auth 완료!"); // 5번

res.json({ message: "성공!" }); // 6번
```

#### `실전 예시`

**완전한 CRUD API**

```js
// /pages/api/posts/[id].js
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export default async function handler(req, res) {
  const { id } = req.query;

  try {
    switch (req.method) {
      case "GET":
        // 게시글 조회
        const post = await prisma.post.findUnique({
          where: { id: parseInt(id) },
        });

        if (!post) {
          return res.status(404).json({ error: "게시글을 찾을 수 없습니다" });
        }

        res.status(200).json(post);
        break;

      case "PUT":
        // 게시글 수정
        const { title, content } = req.body;

        const updatedPost = await prisma.post.update({
          where: { id: parseInt(id) },
          data: { title, content },
        });

        res.status(200).json(updatedPost);
        break;

      case "DELETE":
        // 게시글 삭제
        await prisma.post.delete({
          where: { id: parseInt(id) },
        });

        res.status(204).end();
        break;

      default:
        res.status(405).json({ error: "허용되지 않은 메서드" });
    }
  } catch (error) {
    console.error("Database error:", error);

    if (error.code === "P2025") {
      // P2025 = Prisma에서 "레코드를 찾을 수 없음"을 의미하는 에러 코드
      return res.status(404).json({ error: "게시글을 찾을 수 없습니다" });
    }

    res.status(500).json({ error: "서버 에러가 발생했습니다" });
  }
}
```

**파일 업로드 API**

```js
// /pages/api/upload.js
import multer from "multer";
import { promisify } from "util";

const upload = multer({
  dest: "./public/uploads/", // 파일 저장 위치
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB = 5 × 1024KB × 1024B
  fileFilter: (req, file, cb) => {
    // 업로드 파일 검증
    if (file.mimetype.startsWith("image/")) {
      // image/jpeg, image/png 등
      cb(null, true); // 허용
    } else {
      cb(new Error("이미지 파일만 업로드 가능합니다")); // 거부
    }
  },
});

// Multer는 콜백 기반이므로 Promise로 변환
const uploadSingle = promisify(upload.single("image"));

export default async function handler(req, res) {
  if (req.method !== "POST") {
    return res.status(405).json({ error: "허용되지 않은 메서드" });
  }

  try {
    // 이제 async/await 사용 가능
    await uploadSingle(req, res);

    if (!req.file) {
      return res.status(400).json({ error: "파일이 업로드되지 않았습니다" });
    }

    const fileUrl = `/uploads/${req.file.filename}`; // 웹에서 접근 가능한 URL
    res.status(200).json({
      message: "파일 업로드 성공",
      url: fileUrl,
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
}

export const config = {
  api: {
    bodyParser: false, // multer가 body를 파싱하므로 Next.js의 기본 body 파싱 끄기
  },
};
```

클라이언트에서 CRUD API 호출 <br />

```js
// 게시글 조회
const post = await fetch("/api/posts/123").then((res) => res.json());

// 게시글 수정
await fetch("/api/posts/123", {
  method: "PUT",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ title: "새 제목", content: "새 내용" }),
});

// 게시글 삭제
await fetch("/api/posts/123", { method: "DELETE" });
```

클라이언트에서 파일 업로드 <br />

```js
const formData = new FormData();
formData.append("image", fileInput.files[0]); // HTML input[type="file"]의 파일

const response = await fetch("/api/upload", {
  method: "POST",
  body: formData, // JSON이 아닌 FormData!
});

const result = await response.json();
console.log(result.url); // "/uploads/abc123.jpg" 같은 파일 URL
```

#### `Prisma` ?

: 데이터베이스를 쉽게 다룰 수 있게 도와주는 도구. SQL을 직접 작성하지 않고도 Javascript/Typescript로 데이터베이스 작업을 할 수 있음.

기존 SQL 방식 :

```sql
-- 게시글 조회
SELECT * FROM posts WHERE id = 123;

-- 게시글 생성
INSERT INTO posts (title, content) VALUES ('제목', '내용');

-- 게시글 수정
UPDATE posts SET title = '새 제목', content = '새 내용' WHERE id = 123;

-- 게시글 삭제
DELETE FROM posts WHERE id = 123;
```

Prisma 방식 :

```js
// 게시글 조회
await prisma.post.findUnique({ where: { id: 123 } });

// 게시글 생성
await prisma.post.create({ data: { title: "제목", content: "내용" } });

// 게시글 수정
await prisma.post.update({
  where: { id: 123 },
  data: { title: "새 제목", content: "새 내용" },
});

// 게시글 삭제
await prisma.post.delete({ where: { id: 123 } });
```

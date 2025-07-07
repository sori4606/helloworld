# CSS 핵심개념

### id값에 CSS값 넣기 -> '#'을 붙혀서 값을 넣는다.

ex.

```html
<h3>우도</h3>
<h3 id="hallasan">한라산 국립공원</h3>
<h3>성산 일출봉</h3>
<h3>군산 오름</h3>
```

```css
#hallasan {
  color: #f56513;
}
```

### 클래스에는 ? -> '.'을 붙혀서 넣는다.

ex.

```html
<h3 class="place">우도</h3>
<h3 class="place" id="hallasan">한라산 국립공원</h3>
<h3 class="place">성산 일출봉</h3>
<h3 class="place">군산 오름</h3>
```

```css
.place {
  font-size: 16px;
  font-weight: 400;
}
```

### line-height는 글줄 간의 높이(줄 간격)를 조절하는 속성.

font-size의 상대적인 값으로 측정된다.

ex.

```css
p {
  font-size: 16px;
  line-height: 1.5;
}
```

이 경우 line-height는 (줄높이) 16px \* 1.5 = 24px이 된다.

실무에서는 보통 단위 없는 수치 (1.5, 1.6 등)를 가장 많이 쓴다. -> **상속에 강하고 유지보수가 쉬움**

#### line-height와 박스 높이 계산

ex.

```css
span {
  display: inline-block;
  font-size: 16px;
  line-height: 1.5;
}
```

-> 이럴 때 inline-block 요소의 전체 높이는 24px.
즉, line-height는 **단순히 줄 간격뿐만 아니라 박스의 높이 결정에도 영향**을 줌.

### 백그라운드 이미지 넣기

- **배경 이미지** : background-image : url('flowers.png');
- **배경의 위치** : background-position : center (기본값 : left top, right, bottom, left, top 등)
- **배경의 반복** : background-repeat : no-repeat (기본값 : repeat)
- **배경의 크기** : background-size : cover (비율 유지하면서 꽉차게, 이미지 잘릴 수 있음.) <br />
  (contain : 비율 유지하면서 최대한 크게, 이미지 잘리지 않음.)

+) 추가

배경이미지는 여러개 넣을 수 있음. <br />
ex.

```
background-image:
url('a.png');
url('b.png');
url('c.png');
```

-> a.png 아래에 b.png가 깔리고, 맨 밑에는 c.png가 깔린다.

### 그 외 나머지 CSS 속성들

- **그라디언트** : background-image:
  linear-gradient(45deg, rgba(0, 0, 0, 0.8), rgba(0, 0, 0, 0.2));
  -> 각도, 시작색상, 종료색상 순 (기본방향 각도는 180도 [위에서 아래] )

- **불투명도** : opacity

  ```css
  opacity: 0; /* 투명 */
  opacity: 0.6;
  opacity: 1; /* 불투명 */
  ```

### 박스 모델

- 바깥에서부터 차례로 바깥 여백인 마진, 테두리인 보더, 테두리와 실제 내용 사이의 여백인 패딩, 실제 내용이 들어가는 콘텐트 박스.

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5632&directory=margin-padding-summary-00.png&name=margin-padding-summary-00.png'>

- border 속성
  : 주로 굵기, 테두리 종류, 색상 순서로 씀.

  ```css
  border: 2px solid #ededed;
  ```

- box-sizing 속성
  : 기본적으로 요소에 크기를 지정하면 그 크기는 박스 모델에서 콘텐트 영역에 대한 크기. 예를들어,

  ```css
  #box {
    margin: 20px;
    padding: 30px;
    width: 100px;
  }
  ```

  -> 요소의 넓이는 100+30+30 = 160

  만약, 이런게 아니라 더 직관적으로 크기를 지정하고 싶다면, box-sizing 속성을 바꿔주면 됨. 기본값 : content-box 대신, border-box

  ```css
  #box {
    margin: 20px;
    padding: 30px;
    width: 100px;
    box-sizing: border-box;
  }
  ```

- **자동으로 채우기** : auto

  ```css
  margin: 16px auto;
  width: 520px; //반드시 너비가 정해져 있어야 자동으로 채울 수 있음.
  ```

### 마진 상쇄

발생상황

1.  인접한 블록 요소들 사이 : 위쪽 요소의 하단 마진과 아래쪽 요소의 상단 마진이 겹칠 때

    ```html
    <div id="a">a</div>
    <div id="b">b</div>
    ```

    ```css
    #a {
      margin: 30px;
    }

    #b {
      margin: 20px;
    }
    ```

    <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5632&directory=margin-padding-summary-01a.png&name=margin-padding-summary-01a.png'>

    -> 둘 사이의 마진은 30px이 된다.

2.  부모-자식 관계 : 부모 요소에 패딩, 보더, 인라인 콘텐트가 없는 경우, 자식의 상단 마진이 부모의 상단 마진과 결합됨

    ```html
    <div id="a">a</div>
    <div id="b">
      <div id="c">c</div>
      b
    </div>
    ```

    ```css
    #a {
      margin: 30px;
    }

    #b {
      margin: 20px;
    }

    #c {
      margin: 40px;
    }
    ```

    <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5632&directory=margin-padding-summary-01.png&name=margin-padding-summary-01.png'>

    -> b,c는 부모자식 관계이고, 위쪽 마진은 40px이 됨. 이 마진값을 이웃한 #a와 겹치면, #a와 #b 사이의 마진은 40px임

3.  빈 블록 : 높이가 0인 요소의 상하 마진이 서로 상쇄됨

    ```css
    div {
      margin-top: 20px;
      margin-bottom: 20px;
      height: 0;
    }
    ```

    -> 요소안에 콘텐츠가 없고, height도 0이면, margin-top과 margin-bottom이 서로 상쇄되어 하나만 적용. 즉, 총 여백은 20px (둘중 더 큰값 적용)

방지하는 방법

1. 부모에 padding 또는 border 추가 (부모에 저게 있으면 경계가 있다고 생각한다)
2. 부모에 display : flex 또는 grid 설정 (플렉스 컨테이너는 마진 상쇄 무시)
3. 요소에 position : absolute 적용 (포지셔닝 요소는 마진 상쇄에서 제외됨)

### display 속성

1. block
   : 위에서 아래로 차례대로 배치되고, 크기를 지정할 수 있다
   기본적으로 display 값이 block 인 태그들

   1. &lt;h1&gt;, &lt;h2&gt;, …, &lt;h6&gt;
   2. &lt;p&gt;
   3. &lt;div&gt;

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5637&directory=display-summary-001.png&name=display-summary-001.png'>

2. inline
   : 글을 쓰는 방향으로 줄이 바뀌면서 배치된다 블록과 달리 크기를 지정할 수 없음 (즉, width나 height를 줄 수 없다)
   기본적으로 display 값이 inline 인 태그들

   1. &lt;a&gt;
   2. &lt;br&gt;
   3. &lt;img&gt;
   4. &lt;span&gt;

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5637&directory=display-summary-002.png&name=display-summary-002.png'>

   **참고로** 이미지같이 외부 데이터를 보여 주는 태그는 인라인이지만 크기를 지정할 수 있다. (&lt;img&gt;, &lt;canvas&gt;, &lt;video&gt;, &lt;iframe&gt; 등)

3. inline-block
   : 인라인처럼 배치되고, 블록처럼 크기를 갖을 수 있다

    <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5637&directory=display-summary-003.png&name=display-summary-003.png'>

### float 속성

: 삽화처럼 넣을 때, 그 주변으로는 인라인 요소처럼 배치

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5637&directory=display-summary-004.png&name=display-summary-004.png'>

### css 선택자

: css 규칙에서 맨 앞에 적어 주는 걸 CSS 선택자라고 부름

```
선택자 {
   선언;
   선언;
   선언;
}
```

#### 선택자 목록

: 콤마(,)로 선택자를 연결하면 여러 선택자에 같은 규칙을 적용할 수 있음

```
선택자1,
선택자2 {
    ...
}
```

#### 선택자 붙여 쓰기

: 여러 조건을 동시에 만족하는 요소를 선택하고 싶을 때

```html
<h2 id="mongolia" class="large title">몽골 대자연으로 떠나는 여행</h2>
```

1. 아이디 + 클래스

```
#mongolia.title
```

2. 클래스 + 클래스

```
.large.title
```

3. 태그 + 아이디 + 클래스

```
h2#mongolia.large.title
```

#### 자손 결합자

: 스페이스(띄어쓰기)로 선택자를 이어줌

```html
<div class="article">
  <p>
    이번에 리뷰해 볼 차량은 테슬라 모델 W 입니다.
    <img src="tesla-w-2025.png" />
  </p>
  ...
</div>
```

```css
.article img {
  width: 100%;
}
```

### 가상 클래스

: 실제 class가 없어도 특정 상태를 표현할 수 있게 해주는 선택자

1. :hover
   : 마우스를 요소에 올렸을 때

2. :active
   : 클릭한 순간

3. :focus
   : 포커스가 갔을 때

4. :nth-child(n)
   : n번째 자식 요소

5. :not(...)
   : 해당 조건이 아닌 요소 선택

### 가상 요소

: DOM에 존재하지 않지만, CSS로 요소의 앞이나 뒤에 콘텐츠를 추가하는 가상의 요소

```css
h1::before {
  content: \"🔥 \";
}
```

-> 실제 HTML에 없음에도 불구하고 h1앞에 불 이모지가 생김

### 캐스케이드

: 우선순위가 높은 규칙일수록 우선적으로 적용하는 속성

1. 코드가 더 아래에 있을 수록 우선순위 높음
2. 브라우저에서 기본으로 제공하는 스타일시트 < 우리가 작성한 코드 (h2[브라우저 기본] < h2[우리가 작성])
3. 명시도 계산 (아이디, 클래스, 태그 순)
4. 상속되는 속성들이 정해져 있음 (color, font-family, font-size, font-weight, line-height, text-align 등 ...) -> 텍스트 관련은 기본적으로 **자식도 똑같이 써라**가 기본정책이고, 레이아웃이나 박스 스타일은 **자식이 알아서 정하라**는 방식

   +) 상속안되는 속성도 상속되게 ? = inherit 키워드 사용

   ```css
   div {
     background-color: inherit; /* 부모의 배경색을 따라가게 강제 */
   }
   ```

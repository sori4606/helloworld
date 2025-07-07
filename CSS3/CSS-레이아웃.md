# CSS 레이아웃

### z-index & 쌓임 맥락 (Stacking Context)

---

- z-index는 단순히 숫자 크기로만 결정되는게 아니고, 누가 누구 위에 쌓이는지를 결정하는 룰이 따로 있음.
- 하나의 독립된 위계 공간에서 z-index가 작동하고, 다른 맥락과는 독립적으로 작동함.

1-1 기본 쌓임 순서

1. 루트 요소 <html>. (가장 아래)
2. position 있고 z-index 음수 (숫자 큰 순 -> 동순위는 DOM 순) (DOM 순 ? -> 나중에 등장한 요소가 위에 쌓인다는 말)
3. position 없음 (DOM 순).
4. position 있고 z-index: auto (DOM 순)
5. position 있고 z-index 양수 (숫자 큰 순 -> 동순위는 DOM 순)

1-2 새 쌓임 맥락을 만드는 트리거 ( 이 트리거를 만나면 부모와 독립된 3D 공간이 만들어져 형제 요소의 z-index 영향을 받지 않음.)

1. opacity가 1보다 작은 요소.
2. mix-blend-mode가 normal이 아닌 요소
3. 고정 레이아웃 ( position : fixed (항상), position : sticky (고정 상태일 때) )

```css
.card {
  opacity: 0.9; /* 여기서 새 스택 생성 */
  z-index: 50; /* .card 내부에서만 50 취급 */
}
```

### position 속성

---

1. static(기본) : 왼쪽에서 오른쪽, 위에서 아래로 쌓임. top/right/... z-index 무효
2. relative : 원래 위치를 기준으로 배치
3. absolute : 가장 가까운 포지셔닝이 된 조상요소를 기준으로 배치
4. fixed : 브라우저 화면을 기준으로 배치
5. sticky : static처럼 배치되어 있다가 정해진 위치에 브라우저가 스크롤 되면 그때부터 fixed처럼 고정

### flex 속성

---

#### flex shorthand 정리 (grow / shrink / basis 순)

```css
flex: 1 1 auto; // 콘텐츠 크기만큼 차지. 상황에 따라 늘어나고 줄어듬
flex: 1; // flex: 1 1 0% 와 같음. 남는 공간 균등 분할되며 콘텐츠 크기 무시
flex: auto; // flex: 1 1 auto 와 같음. 상황에 따라 늘고 줄어듬
flex: 0 auto; // flex: 0 1 auto 와 같음. 줄어들지만 늘어나지 않음
```

1. flex-grow (요소 늘려서 채우기)

   - 남는 공간을 어떻게 분배할지 결정하는 "성장"비율
   - 기본값 : 0 (남는 공간을 차지하지 않음)
   - 양의 정수로 설정하면 flex 컨테이너 내의 남는 공간을 해당 비율로 분배 (ex. A의 flex-grow : 1, B의 flex-grow : 2라면, 남는 공간을 1:2 비율로 나눠가진다.)

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flex-grow-00.png&name=flex-grow-00.png'>

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flex-grow-01.png&name=flex-grow-01.png'>

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-017.png&name=flexbox-summary-017.png'>

2. flex-basis (width)

   - flex 아이템의 초기 크기를 설정
   - 기본값 : auto
   - width처럼 px, %, em 등의 단위로 설정 가능
   - 0으로 설정하면 콘텐츠 크기를 무시하고 flex-grow 값에 따라 공간 분배

   ```css
   .item {
     flex: 1 1 200px; /* 200px을 출발점으로 grow/shrink */
   }
   ```

3. flex-shrink

   - 공간이 부족할 때 얼마나 줄어들지 결정하는 "수축"비율
   - 기본값: 1 (공간이 부족하면 flex-basis를 기준으로 비율에 맞게 줄어듬)
   - 0으로 설정하면 줄어들지 않음
   - 값이 클수록 다른 항목보다 더 많이 줄어듬

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-018.png&name=flexbox-summary-018.png'>

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-019.png&name=flexbox-summary-019.png'>

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-020.png&name=flexbox-summary-020.png'>

4. flex-wrap

   - 줄바꿈 여부
   - 기본값 : nowrap (강제로 한줄에 모두 배치)
   - wrap, wrap-reverse 등

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-011.png&name=flexbox-summary-011.png'>

5. flex-direction

   - 주축 방향 설정
   - 기본값 : row
   - column 등

6. flex-flow

   - flex direction + flex wrap의 shorthand
   - row wrap 등

7. justify-content

   - 주축 정렬
   - 기본값 : flex-start
   - center, space-between 등

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-003.png&name=flexbox-summary-003.png'>

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-005.png&name=flexbox-summary-005.png'>

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-006.png&name=flexbox-summary-006.png'>

8. align-items

   - 교차축 개별 아이템 정렬
   - 기본값 : stretch (늘려서 배치하기)
   - center, flex-end 등

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-007.png&name=flexbox-summary-007.png'>

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-008.png&name=flexbox-summary-008.png'>

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-009.png&name=flexbox-summary-009.png'>

   <img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-010.png&name=flexbox-summary-010.png'>

9. align-content

   - 여러 행(열) 묶음의 교차축 정렬 (한 줄로만 이루어진 플렉스 컨테이너에는 아무 효과 없음)
   - 전체 content 들의 위치정렬로 이해하면 될 듯
   - space-around, center 등

10. gap

- 행·열 간격
- ex. gap: 20px 10px; (행, 열) , (세로간격, 가로간격 순)

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-012.png&name=flexbox-summary-012.png'>

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5780&directory=flexbox-summary-013.png&name=flexbox-summary-013.png'>

ex.한 눈에 보는 패턴 예시

```css
.container {
  display: flex;
  flex-flow: row wrap; /* 가로 흐름 + 넘치면 다음 줄 */
  justify-content: center; /* 가로 가운데 */
  align-items: center; /* 세로 가운데 */
  gap: 24px;
}
```

### 이건 언제 ?

- 가운데 정렬하고 싶을 때,

```css
display: flex;
justify-content: center; // 주 축(가로)에서 아이템들을 가운데 정렬
align-items: center; // 교차 축(세로)에서 아이템들을 가운데 정렬
```

=> 전체 container에 적용하면, 컨테이너 안의 모든 flex 아이템들이 컨테이너 내에서 가운데에 정렬됨. 직접 item에 적용하면, 아이템 자체가 flex 컨테이너가 되어, 아이템 내부의 텍스트나 다른 요소들이 아이템 안에서 가운데 정렬된다. ( 기준을 잘 잡고 flex 설정하자 ! )

### 인라인 안에서 플렉스 박스 ? (인라인 안에서 세로정렬 하고 싶을 때)

#### display : inline-flex;

ex.

```html
<p>
  코딩, 쉬워질 때도 됐다.
  <a class="new-window-link" href="https://codeit.kr">
    코드잇
    <img
      class="icon"
      src="new-window-link.svg"
      alt="새 창 열기"
      width="13"
      height="13"
    />
  </a>
  에서 지금 바로 시작해보세요.
</p>
```

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5781&directory=inline-flex-0.png&name=inline-flex-0.png'>

여기서, 새 창 열기 아이콘을 글 가운데에다 정렬하려고 .new-window-link 클래스에다가 display: flex 넣고, 간격을 넣어주면,

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5781&directory=inline-flex-1.png&name=inline-flex-1.png'>

이런식으로 줄이 넘어가는 걸 볼 수 있다.
왜 ? -> a 태그의 inline 이라는 기본값을 display : flex로 바꿔 줬기 때문. display: flex를 하면 그 안에서는 플렉스박스 규칙에 따라 배치되고, 그 바깥에는 display: block처럼 위에서 아래로 배치된다.

**그래서 이럴때는 inline-flex를 사용하자!**

사용한 모오습

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5781&directory=inline-flex-2.png&name=inline-flex-2.png'>

### Grid (display : grid, 2차원 레이아웃)

- display:grid;
- grid-template-columns: 100px 200px 100px;
- grid-template-columns: 1fr 1fr 1fr; (비율 1:1:1) **또는** grid-template-columns: repeat(3, 1fr);
- grid-template-columns: repeat(3, minmax(200px, 1fr)); => 최소너비 200px
- grid-template-rows: 150px 200px;
- gap : 20px 10px; (세로 : 20px, 가로 : 10px)

#### 원하는 위치에 요소 배치하기

ex. 3\*2 상황에서 (세로선은 1, 2, 3, 4 / 가로선은 1, 2, 3)
grid-column : 2/4; = 2/-1 = 2/span 2; (2칸차지)
grid-row : 1/3 = 1/-1; = 1/span 2;

#### ex

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5797&directory=grid-summary-001.png&name=grid-summary-001.png'>

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5797&directory=grid-summary-002.png&name=grid-summary-002.png'>

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5797&directory=grid-summary-003.png&name=grid-summary-003.png'>

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5797&directory=grid-summary-004.png&name=grid-summary-004.png'>

<img src = 'https://bakey-api.codeit.kr/api/files/resource?root=static&seqId=5797&directory=grid-summary-005.png&name=grid-summary-005.png'>

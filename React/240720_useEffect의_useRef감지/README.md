# 240720_useEffect의 useRef 인식

### 문제 상황

- 게시글상세페이지(FilmDetailPage)에서 yum버튼(좋아요버튼)을 클릭했을 때만 하단 게시글 내용 컨테이너가 제자리로 위치하는 버그 발생

### 문제 해결 과정

1. 문제 인식
   - useEffect로 지정한 초기위치 490이 먼저 적용되고 이후 resizeObserver를 통해 갱신되도록 로직을 구성하였으나, 초기값에서 변하지 않는 상황이라고 생각하고 접근 : 0으로 바꿔서 확인
   - 이 과정에서 초기에 useRef가 null이라 resizeObserver가 관측하지 않고, 그에 따라 사이즈가 변경되지 않는 상황 발견 : console.log로 확인
   - useEffect 에서 의존성 배열에 useRef, ref.current 값의 변화를 감지하지 못함을 확인, 그에 관련된 검색을 해보니 이미 많은 해결방법이 나와있음

### 결론

1. @types의 의미

### 알게 된 것

1. @types
2. dependencies

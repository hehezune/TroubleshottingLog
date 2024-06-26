# 240404_Swal2

### 문제 상황

- Modal 이 열려 있는 상태에서 Swal 의 ConfirmButton 이 클릭될 경우, Modal 또한 같이 close 되는 문제
- 원인 : Swal에서 발생한 click 이벤트가 버블링되어 document에서 감지, 기존 조건문인 `if( modalRef.current &&
    !modalRef.current.contains(e.target as Node))` 에 의해 Modal이 종료

### 문제 해결 과정

1. addEventListenr를 통해 ConfirmButtond의 이벤트 전파를 막기

   - 다음과 같은 코드로 시도해보았으나 querySelector로 제어하는 element는 addEventListner의 자료형 문제가 발생하여 원하는 대로 다룰 수 없었음

   ```javascript
     function test(event: MouseEvent) {
       console.log('확인');
       event.stopPropagation();
     }
     useEffect(() => {
       document.querySelector('swal2-confirm')
     ?.addEventListener('click', test);
   ```

2. Swal의 공식 문서를 통해 ConfirmButton의 이벤트 제어를 할 수 있는지 알아보기
   - 조절 불가능, 관련 속성 찾을 수 없었음
3. Modal 의 조건문을 변경하기 : Modal 내부 컴포넌트에 속하지 않으면서, Modal 외부 회색 영역에 포함되는가?
   - ```javascript
     if (modalRef.current && !modalRef.current.contains(e.target as Node) &&
     modal2Ref.current?.contains(e.target as Node)
     )
     ```
   - 해결

### 결론

- Swal 에서 이벤트 자체를 컨트롤 할 순 없다.
- React 내부 문법에서 querySelector를 운용하는 것에 애로사항이 있다. (document 객체의 addEventListner)
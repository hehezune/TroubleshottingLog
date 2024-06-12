# 240601_accessToken 갱신2

### 문제 상황

- 이전까지의 해결 상황 : 액세스 토큰 기한이 만료된 이후에 리프레시 토큰을 통한 액세스 토큰 갱신 문제 해결
- 유저 인증을 담당하는 AuthRoute에서 accessToken 만료를 확인하고, refreshToken을 통해 갱신을 시킴
- 그러나 refreshToken을 통한 갱신이 끊임없이 일어나면서 무한리렌더링 현상이 발생함

### 문제 해결 과정

1. 액세스토큰 갱신이 제대로 이루어지는가?

   - CheckAuthRoute의 로직 상, accessToken을 통한 권한 인증이 실패했을 경우 refreshToken 갱신 절차가 이루어진다. 그런데 refreshToken 갱신이 계속해서 이루어진다는 것은 accesssToken 갱신이 정상적으로 작동하지 않았다는 의미일 것이다.
   - 이를 확인하기 위해 refreshToken 갱신 결과, jotai에 갱신되는 accessToken을 확인해보았으나 다 정상이었다.
   - accessToken은 정상적으로 갱신되나, 그 이후 accessToken을 포함한 요청에서는 과거에 사용하는 accessToken을 사용하는 것을 파악하였다
   - 로컬스토리지 내역을 삭제하고 다시 테스트해본 결과 이번엔 요청 자체가 전송되지 않았다.
   - 위의 사실들을 들어 봤을때, 기존의 로컬스토리지에 토큰을 저장하고 그걸 꺼내쓰는 방식이 남아 있어 과거의 토큰이 요청 시 덮어씌워지는 것이 문제였다.
   - 일단 CheckAuthRoute의 로직이 완성된 후에, 기존 로직들을 제거하자는 생각으로 놔두고 있었는데 이 판단으로 인해 오히려 CheckAuthRoute의 완성에 방해가 되고 있었다.
   - localStorage를 이용하는 로직을 전부 주석처리 하니 accessToken 갱신이 정상적으로 이루어졌다.

2. useEffect의 적용

   - 1번의 accessToken이 과거 데이터로 덮어씌워지는 문제를 해결했음에도 불구하고, 무한리렌더링은 해결되지 않았다.
   - 첫 렌더링 후 한번만 accesToken을 체크하기에 useEffect의 필요성을 못느껴 useEffect를 사용하지 않은 코드를 작성했는데, 결국 외부 API와 연동해야 하는 것이므로 여기서 문제가 발생하는것이 아닐까 싶어 useEffect를 적용했다.

3. react-query의 캐싱 문제

   - useEffect를 적용했음에도 불구하고 여전히 accessToken 갱신 이후 새로운 요청이 발생하지 않았다.
   - accessToken을 이용하는 userProfile 요청이 초기에 한번만 일어나는 것을 보고, react-query의 캐싱이 적용되어 새로운 요청이 일어나지 않는 것으로 보였다.
   - useQuery는 queryKey의 변화가 일어났을때 새로운 요청을 보내는데, queryKey에는 memberId만 감지하고 있기에 새로고침을 하더라도 memberId의 변화는 없는 것이다. 그렇기에 갱신된 이후 다시 인증을 검증하는 절차가 발동되지 않는다. (refresh를 통한 갱신 이후, 새로 발급받은 accessToken에 대한 인증 절차를 가져야 하는데 queryKey가 같기 때문에 인증을 요구하는 요청이 가지 않음)
   - 처음에는 staleTime을 0으로 만들어 바로 유효하지 않은 데이터를 만드는 방식을 시도했지만, 애초에 queryKey의 변화가 없기때문에 해결하지 못했다.
   - 이전 240517에서 사용한 queryClient를 이용해 쿼리데이터를 제거하는 방식을 통해 정상작동함을 확인할 수 있었다.

### 결론

1. useEffect와 react-query에 대해 더 정확하게 이해하는 것이 필요하다.

   1. useEffect : React Hook that lets you synchronize a component with an external system.

      - 위의 문구는 react 공식문서 내용인데, synchronize에 대해서 다른사람에게 설명할 수 있을 정도의 이해해야할 거 같다.

   2. react-query : useQuery에서의 data가 어떤 경우에는 알아서 변화를 감지해주고, 어떤 경우에는 감지못하는 경우가 있는데 명확히 어떤 경우에 이루어지는지 파악이 필요하다.

2. 문제 해결 과정의 1번에서, 새로운 방식을 도입하는 과정에서 기존 방식의 코드가 남아있어 방해하는 것을 경험했다. 어차피 기존 방식 코드는 깃에 남아있으므로, 새로운 방식을 도입할 경우에는 깔끔하게 기존 방식을 다 지우는게 더 빠른 문제 해결로 이어질 수 있다는 것을 느꼈다.

### 질문

1. useEffect를 적용하지 않았을 경우, 왜 refreshToken 재요청만 계속 일어났을까? (accessToken을 이용하는 userProfile 요청은 왜 한번만 일어나는가)

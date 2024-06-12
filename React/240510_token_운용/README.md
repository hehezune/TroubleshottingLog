# 240510_token 운용

### 문제 상황

1. accessToken 가져오기

   - 백엔드에서 보내주는 accessToken은 header의 Authorization에 담겨 있다.
   - axios에서 해당 부분을 가져오려면 headers.getAuthorization()을 통해 가져올 수 있다.
   - 하지만 실제로 headers.getAuthorization()을 적용했을 때, 에러가 뜨는 상황을 겪었다.

2. refreshToken 가져오기

   - refreshToken은 백엔드에서 set-cookie에 담겨져서 전달된다.
   - cookie를 JS에서 꺼내는 법과, 다시 요청을 보낼 때 담는 방법에 대해 무지하여 어려움을 겪었다.

### 문제 해결 과정

1. accessToken 가져오기

   - headers.getAuthorization() 적용시 에러문구
   - getAuthorization이 무엇인지(타입) 결정되지 않아 발생하는 typescript 오류였음
   - 이를 할당하기 위해 getAuthorization이 함수인지 조건문을 통해 판별한 수 사용하면 사용할 수 있었음

2. refreshToken 가져오기

   - 백엔드에서 refreshToken을 HttpOnly 속성으로 보냈다. 이는 프론트에서 JS를 통해 해당 토큰을 직접 조작할 수 없음을 의미한다.
   - 당시엔 이를 모르고 cookie를 가져오려고 document.cookie에서 값을 읽어내려 했지만, 어떤 값도 읽어낼 수 없었다.
   - refreshToken을 보내기 위해선 읽어서 가져와야 했고, 결국 HttpOnly를 true로 보내는 것이 잘못되었다고 생각하여 HttpOnly를 false로 변경하여 cookie를 가져왔다.
   - refreshToken을 백엔드롤 보내는 방법이 문제였다. 내가 추출해낸 cookie를 담는 방법을 찾지 못했다. (당시 검색 미스였으나 쿠키를 직접 담는 방법은 존재함. 그러나 현재 클라이언트에서 쿠키를 직접 조작하는 것 자체가 보안상의 문제가 발생할 여지가 있어 withCredentials를 사용하는 방법이 권장됨)
   - withCredentials : 서로 다른 도메인(크로스 도메인)에 요청을 보낼 때 요청에 credential 정보를 담아서 보낼 지를 결정하는 항목입니다. (1)
     - 여기서 credentials 정보란,
     - 쿠키를 첨부해서 보내는 요청
     - 헤더에 Authorization 항목이 있는 요청
   - withCredentials`은 자격 증명을 사용하여 사이트 간 액세스 제어 요청을 해야 하는지 여부를 나타냅니다 (2: axios 공식문서)
   - 즉 refreshToken을 쿠키에 첨부해서 보내야 하므로, withCredential 옵션을 사용하는데, 해당 옵션은 response에서 받은 쿠키를 사용하므로 직접 쿠키를 가져올 필요가 없고, 따라서 HttpOnly true는 올바른 사용이었다는 것

### 결론, 얻은 점

- typescript에서 타입 보는 것의 중요성, 역시 문제 해결을 위해선 에러코드 분석이 중요
- withCredentials의 의미와 기능

### 참고사이트

1. withCredentials
   - https://junglast.com/blog/http-ajax-withcredential
2. https://axios-http.com/kr/docs/req_config

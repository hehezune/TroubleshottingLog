# 240530_accessToken 갱신

### 문제 상황

- accessToken의 기한이 만료된 이후 refreshToken을 통해 갱신할 경우, 403 에러가 발생했다.
- 이 사실을 백엔드 담당자에게 전달했으나, 로직상 문제가 없다는 것을 확인받았고 postman을 통해 확인해볼 것을 요청받았다.
- postman을 통해 확인했을 때 문제가 없음을 인지했고, 결국 프론트엔드 로직에 문제가 있다는 의미인 셈

### 문제 해결 과정

- postman을 통한 결과와 프론트에서 보낸 결과가 차이가 있다는 점에서, 두개의 request와 response를 비교해보고, 서버에서의 로그를 비교해보았다.
- postman으로 발생한 서버로그에서 주목할 점은 token null이라는 로그였다.
- 프론트엔드에서 보내는 요청에는 accessToken이 들어있었다.
- 백엔드에서 판단했을 때, token 유무에 따라 Filter의 분기가 갈리고, refreshToken을 통한 갱신은 token이 없을때 일어나는 로직이라 만료된 accessToken때문에 처리가 안되던 것이었다.
- 문제로는 대부분의 API 요청에 accessToken을 직접 넣어주는 로직이었다.
- 위의 로직은 axios.default.headers를 통해 accesToken을 설정하는 방식이, 페이지가 변할 경우에 적용이 되지 않던 것으로 착오하여 모든 API에 직접 넣어줬던 로직이었다.
- axios에 accessToken을 직접 넣는 로직을 제거한 뒤 정상작동하였다.

### 결론

- axios.default.headers는 페이지가 바뀌어도 정상 작동했다. 오류 당시 다른 이유가 겹쳐서 정상작동 하지 않은 것으로 보인다.
- 백엔드 API 통신에 있어서 오류 검증을 할 때, postman을 통한 검증을 반드시 해보는 게 좋다.

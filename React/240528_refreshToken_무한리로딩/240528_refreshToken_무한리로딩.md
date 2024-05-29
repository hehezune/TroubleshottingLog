# 240517_refreshToken 무한리로딩

### 문제 상황

- accessToken 권한 확인용 AuthRoute를 통해, accessToken이 만료되었는지 확인 후 refreshToken으로 재발급하려 함
- 현재 백엔드에서 만료되었을 경우 403에러를 전달하기에, 403 에러를 확인했을 경우 refresh를 던지도록 했으나 무한으로 refreshToken 전달 및 그에 따른 Outlet 렌더링이 일어나는 상황

### 문제 해결 과정

### 결론

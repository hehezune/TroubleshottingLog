# 240517\_페이지 이동하면서 refetch 트리거

### 문제 상황

- 게시글 수정 이후 게시글 상세페이지로 navigate 시켰을 때, 게시글 상세 페이지에서 읽어오는 데이터가 react query의 캐싱되어 있는 수정 전 게시글 상세 데이터가 읽어와짐
- 이를 해결하기 위한 과정 (useEffect + refetch, queryClient 사용 등)

### 문제 해결 과정

1. refetch 사용
   - 캐싱된 데이터를 이용하지 않고, 새로 API 요청을 하는 refetch를 적용
   - props drilling이 심하지 않을 경우엔 적용할만하지만, 이 경우엔 너무 많은 경우의 수가 존재하여 부적합하다고 판단
2. removeQueries나 invalideQueries 이용
   - 전달한 queryKey에 대응되는 캐싱된 쿼리데이터를 지우는 removeQueries나 보유중인 쿼리데이터를 invalidate시키고 새로 받아오는 invalidateQueries를 사용
   - 위의 메서드들은 queryClient에 내장된 메서드들이므로, queryClient를 가져와야 함
   - 처음에 시도할 당시에는 new QueryClient() 생성자를 통해 가져온 queryClient를 사용하였기에, 해당 queryClient에는 저장된 쿼리데이터들이 없었음
   - App.tsx에서 초기설정으로서 QueryClient 생성자를 사용했었는데, 이때 생성된 queryClient가 내가 원할때 생성시켜서 만드는 것과 연결되어있는 것에 대한 의구심이 들었음
   - 결국 react query 문서를 통해 이미 정의된 queryClient를 사용하려면 useQueryClient를 사용해야 함을 깨달음
   - removeQueries를 사용했으며, 정상작동함을 확인

### 결론 및 깨달은 점

1. 사실 new QueryClient로 기존의 객체를 가져온다는것 자체가 애초에 말이 안되는건데 시도한 것이 매우 바보같은 선택이었음
2. useQueryClient로 정의된 객체를 가져오는 것에서 뭔가 디자인패턴의 일종인가 느낌
3. removeQueries는 쿼리데이터를 지우는 것에서 끝나지만, invalidateQueries는 invalidate 후 refetch까지 포함된다. 이번 경우는 removeQueries 이후 상세 페이지로 navigate하며 해당 페이지에서 자동으로 API를 호출하기에 removeQueries 사용이 가능했다.

### 궁금한 점

- 만약 invalidateQueries로 API를 갱신시켰으면, removeQueries보다 빠른 로딩 속도를 기대할 수 있는가? removeQueries는 엄밀히 따지면 상세페이지에서 API 요청을 하는 것이고, invalidateQueries는 API 요청 보내고 페이지를 이동시키는건데.. 뭔가 navigate하면서 요청이 꼬일수도? 테스트가 필요할듯

## Query 상태 변화

![image](https://user-images.githubusercontent.com/62097867/210059335-2b868518-df00-4327-8a85-900a0e00a1b7.png)

### 🔎 Stale Time vs Cache Time

`Stale Time`

- fresh한 쿼리 데이터가 stale 상태로 전환될 때까지의 시간 (default: 0)
- 언제부터 데이터를 stale하다고 취급할 것인지.

`Cache Time`

- inactive한 쿼리가 cache에서 제거될 때까지의 시간 (default: 5분)
- 메모리에 얼마나 있을건지.

### fresh

- 새로운 쿼리 인스턴스
- staleTime동안 fresh한 상태가 유지되고, staleTime이 지나면 stale 상태로 전환된다.
- staleTime이 0이면 바로 stale 상태로 전환된다.
- 쿼리가 fresh한 경우 cache로부터 데이터를 읽어오며 네트워크 요청이 발생하지 않는다.

### stale

- 인스턴스가 존재하지만 staleTime이 지난 상태로 구식 데이터임을 의미한다.
- 쿼리가 stale한 경우에도 여전히 데이터를 cache에서 가져오지만 특정 조건에서 background refetch가 발생한다.

### inactive

- 쿼리에 대한 옵저버가 없는 경우(즉 해당 쿼리를 사용하는 모든 컴포넌트들이 unmount된 상태) 즉시 inactive 상태로 전환된다.
- inactive 쿼리는 cacheTime이 지나면 GC에 의해 처리된다.

https://tkdodo.eu/blog/practical-react-query#the-defaults-explained

## Caching Examples

**query instances without cache data**

1. 새로운 쿼리 인스턴스가 mount되면 loading state를 보여주고 데이터를 가져오기 위한 네트워크 요청이 발생한다.

2. 가져온 데이터는 queryKey string을 key로 하여 캐싱된다.

**query instances with cache data**

1. actvie한 상태의 쿼리 인스턴스가 mount된다.

2. 데이터를 cache로부터 가져온다.

3. 쿼리가 stale한 경우 background refetch가 발생한다. 이때 쿼리 status가 업데이트 된다. => isLoading / isFetching 구분

**inactive queries**

1. 쿼리 인스턴스가 모두 unmount되어 active한 인스턴스가 없는 경우 cache timeout이 cacheTime으로 설정된다.

2. cache timeout되기 전에 다른 인스턴스가 mount되면 cache로부터 즉시 데이터를 가져오고 background refetch가 일어난다.

3. cacheTime 안에 새로운 인스턴스가 나타나지 않으면 캐시 데이터는 가비지 콜렉터에 의해 삭제된다.

https://tanstack.com/query/v4/docs/react/guides/caching

## 정리

| 쿼리 상태 | fetch from | background refetch |
| --------- | ---------- | ------------------ |
| fresh     | cache      | X                  |
| stale     | cache      | O                  |
| inactive  | cache      | O                  |

> **💡 structural sharing**
>
> "Query results by default are structurally shared to detect if data has actually changed and if not, the data reference remains unchanged to better help with value stabilization with regards to useMemo and useCallback."
>
> 따라서 background refetch시 가져온 데이터가 실제로 변경되었을때만 캐시가 업데이트되고 UI에 반영됨

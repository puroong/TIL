# React Query

## Overview

* API 연동 및 결과 캐싱을 간단하게 해주는 라이브러리다

* 서버 상태 관리는 클라이언트 상태 관리와는 많이 다르다
    * 서버 상태는 서버에 저장돼 있으며, 클라이언트가 통제할 수 없다
    * 비동기 처리를 요구한다
    * can potentially become out-of-date if youre not careful

* 또, 아래 항목들도 고려해야한다
    * 캐싱
    * 같은 데이터를 불러올 때 중복 요청 제거
    * updating out-of-date data in the background
    * knowing when the data is out-of-date
    * 최신 데이터를 최대한 빨린 반영하기
    * 페이지네이션, lazy loading과 같은 성능 최적화
    * 서버 상태에 대한 메모리 관리와 가비지 컬렉션

* React Query를 사용하면 아래 효과들을 얻을 수 있음
    * API 연동과 관련된 복잡한 로직 간단하게 만들 수 있음
    * 속도 향상

* 예시
```
import { QueryClient, QueryClientProvider, useQuery } from 'react-query'
 
 const queryClient = new QueryClient()
 
 export default function App() {
   return (
     <QueryClientProvider client={queryClient}>
       <Example />
     </QueryClientProvider>
   )
 }
 
 function Example() {
   const { isLoading, error, data } = useQuery('repoData', () =>
     fetch('https://api.github.com/repos/tannerlinsley/react-query').then(res =>
       res.json()
     )
   )
 
   if (isLoading) return 'Loading...'
 
   if (error) return 'An error has occurred: ' + error.message
 
   return (
     <div>
       <h1>{data.name}</h1>
       <p>{data.description}</p>
       <strong>👀 {data.subscribers_count}</strong>{' '}
       <strong>✨ {data.stargazers_count}</strong>{' '}
       <strong>🍴 {data.forks_count}</strong>
     </div>
   )
 }
```

## Important Defaults

* useQuery 또는 useInfinityQuery로 생성한 쿼리 인스턴스는 기본적으로 캐시를 오래된 데이터로 간주한다
    * `staleTime` 옵션으로 설정을 바꿀 수 있다

* 오래된 데이터는 아래의 경우일 때 자동으로 갱신된다
    * 새로운 쿼리 인스턴스가 마운트된 경우
    * 창이 refocused된 경우
    * 네트워크가 끊겼다 다시 연결된 경우
    * `refetch interval`이 옵션으로 설정된 경우

* 예상하지 못했을 때 데이터가 갱신됐다면 창이 refocusing 됐을 가능성이 크다
    * `refetchOnMount`, `refetchOnWindowFocus`, `refetchOnReconnect`, `refetchInterval` 옵션으로 위 기능을 바꿀 수 있다

* `useQuery`, `useInfiniteQuery`또는 쿼리 옵저버의 활성화된 인스턴스가 없는 쿼리 결과는 inactive로 간주되고 캐시에 저장된다
* 기본적으로 inactive한 쿼리 결과는 5분 뒤에 가비지 컬렉터가 회수한다
    * `cacheTime`으로 위 기능을 바꿀 수 있다

* 쿼리가 실패했다면 화면에 에러를 표시하기 전에 3번 더 호출된다
    *  `retry`, `retryDelay` 옵션으로 이 설정을 바꿀 수 있다

## Queries

* A query is a declarative dependency on an asynchronous of data that is tied to a unique key
* Promise와 함께 사용할 수 있다
* 데이터를 수정해야 한다면 Mutation을 사용하는 것이 좋다

* To subscribe to a query in your components or custom hooks, call the useQuery hook with at least:
    * 고유 키
    * Promise를 반환하는 함수

* 고유 키는 내부적으로 refetching, 캐싱, 어플리케이션 내에서 쿼리 결과 공유를 할 때 사용된다

```
result = useQuery('todos', fetchTodoList);
```

* result로 쿼리를 조회에 성공했는지 여부를 알 수 있다
    * isLoading / status === 'loading'
        * 데이터를 가져오고 있는중 (데이터 아직 없음)
    * isError / status === 'error'
        * 에러 발생함
        * result.error로 에러 상세 정보를 확인할 수 있음
    * isSuccess / status === 'success'
        *  데이터 가져온느데 성공함 
        * result.data로 데이터를 확인할 수 있음
    * isIdle / status === 'idle'
        * 쿼리가 disabled된 상태임

    * 모든 상태에 대해 데이터를 가져오고 있는 중이라면 result.isFetching 값이 true임

## Query Keys

* 내부적으로 React Query는 고유 키를 사용하여 캐시를 관리한다
* 직렬화만 가능하다면 어떤 값이든 고유 키로 사용할 수 있다

* 가능한 Query Key 종류
    * String only Query Keys
    * Array Keys
    * Query Keys are hashed deterministically
        * 아래 키 모두 동일한 키임
        ```
        useQuery(['todos', { status, page }], ...)
        useQuery(['todos', { page, status }], ...)
        useQuery(['todos', { page, status, other: undefined }], ...)
        ```

* 만약 쿼리 함수가 의존하는 값이 있다면 쿼리 키에 포함 시키는 것이 좋다
```
function Todos({ todoId }) {
    const result = useQuery(['todos', todoId], () => fetchTodoById(todoId))
}
```

## Query Functions

* Promise를 반환하는 어떤 함수

* 쿼리 함수 내에서 throw 된 에러는 result.error에서 확인할 수 있다

* Query Function Variable
    * 쿼리 키로 쿼리 함수의 인자를 넘겨줄 수 있다
    ```
    function Todos({ status, page }) {
        const result = useQuery(['todos', { status, page }], fetchTodoList)
    }

    // Access the key, status and page variables in your query function!
    function fetchTodoList({ queryKey }) {
        const [_key, { status, page }] = queryKey
        return new Promise()
    }
    ```

* Using a Query Object instead of parameters
```
 import { useQuery } from 'react-query'
 
 useQuery({
   queryKey: ['todo', 7],
   queryFn: fetchTodo,
   ...config,
 })
```

## Parallel Queries

* 동시에 실행된다

* Manual Parallel queries
    * useQuery를 여러번 사용하면 된다 
    (React Query의 suspense 모드에선 첫번째 쿼리가 Promise를 반환하고 다른 쿼리들이 실행되기 전에 컴포넌트를 suspend 시키기 때문에 작동하지 않는다. 이런 경우엔 useQueries를 사용해야 한다)

* Dynamic Parallel Queries with useQuries
    * 렌더링 할 때마다 실행해야 되는 쿼리 개수가 변하면 Manual Parallel Query를 사용할 수 없다
    (that would violate the rules of hooks)

    * 대신 아래와 같이 useQueries를 사용할 수 있다
    ```
    function App({ users }) {
        const userQueries = useQueries(
        users.map(user => {
            return {
                queryKey: ['user', user.id],
                queryFn: () => fetchUserById(user.id),
            }
        })
        )
    }
    ```

## Dependent Queries

* 어떤 쿼리가 다른 쿼리에 의존하도록 만드려면 enabled 옵션을 사용하면 된다
```
// Get the user
const { data: user } = useQuery(['user', email], getUserByEmail)

const userId = user?.id

// Then get the user's projects
const { isIdle, data: projects } = useQuery(
    ['projects', userId],
    getProjectsByUser,
    {
        // The query will not execute until the userId exists
        enabled: !!userId,
    }
)

// isIdle will be `true` until `enabled` is true and the query begins to fetch.
// It will then go to the `isLoading` stage and hopefully the `isSuccess` stage :)
```

## Background Fetching Indicators

* 쿼리가 refetching하고  있다는 걸 확인하고 싶을 땐 isFetching을 사용하면 된다

```
function Todos() {
    const { status, data: todos, error, isFetching } = useQuery(
        'todos',
        fetchTodos
    )

    return status === 'loading' ? (
        <span>Loading...</span>
    ) : status === 'error' ? (
        <span>Error: {error.message}</span>
    ) : (
    <>
        {isFetching ? <div>Refreshing...</div> : null}

        <div>
            {todos.map(todo => (
            <Todo todo={todo} />
            ))}
        </div>
    </>)
}
```

* Displaying Global Background Fetching Loading State

* any 쿼리가 fetching 중인지 확인하고 싶다면 useIsFetching 훅을 사용하면 된다
```
import { useIsFetching } from 'react-query'

function GlobalLoadingIndicator() {
    const isFetching = useIsFetching()

    return isFetching ? (
        <div>Queries are fetching in the background...</div>
    ) : null
}
```

## Window Focus Refetching

* If a user leaves your application and returns to stale data, React Query automatically requests fresh data for you in the background
* You can disable this globally or per-query using the refetchOnWindowFocus option:

* Disabling Globally
```
const queryClient = new QueryClient({
    defaultOptions: {
        queries: {
            refetchOnWindowFocus: false,
        }
    }
})

function App() {
    return <QueryClientProvider client={queryClient}>...</QueryClientProvider>
}
```

* Disabling Per Query

```
useQuery('todos', fetchTodos, { refetchOnWindowFocus: false })
```

* Custom Window Focus Event

* 아주 희귀한 케이스지만 window focus 이벤트 핸들러를 직접 관리해야 할 경우가 있을 수 있다
* focusManager.setEventListener로 관리할 수 있다
* focusManager.setEventListener는 이전 핸들러를 새로운 핸들러로 덮어쓴다
```
focusManager.setEventListener(handleFocus => {
   // Listen to visibillitychange and focus
   if (typeof window !== 'undefined' && window.addEventListener) {
     window.addEventListener('visibilitychange', handleFocus, false)
     window.addEventListener('focus', handleFocus, false)
   }
 
   return () => {
     // Be sure to unsubscribe if a new handler is set
     window.removeEventListener('visibilitychange', handleFocus)
     window.removeEventListener('focus', handleFocus)
   }
 })
```

* Ignoring IFrame Focus Event

* IFrame은 window focus 이벤트를 두번 발생 시키는 문제점을 가지고 있다
* focusManager.setEventListener로 이를 해결할 수 있다


* Managing focus state

```
import { focusManager } from 'react-query';

focusManager.setFocused(true);
focusManager.setFocused(undefined);
```

## Disabling/Pausing Queries

* 쿼리를 disable 하고 싶다면 enabled 옵션을 사용하면 된다
* enabled가 false라면
    * 쿼리에 대한 캐시가 있으면 success 상태가 된다
    * 캐시가 없다면 idle 상태가 된다
    * The query will not automatically fetch on mount
    * refetch 함수로 다시 쿼리를 활성화 시킬 수 있음

## Query Retries

* useQuery 쿼리가 실패하면 설정된 횟수만큼 재요청한 뒤 에러 메시지를 표시한다

* retry 옵션으로 설정할 수 있다
* retry=false면 retry를 비활성화한다
* retry=6은 6번 재시도한다
* retry=true는 무한번 재시도한다
* retry = (failureCount, error) => ...  은 쿼리가 실패했을때 전달받은 함수를 호출한다

```
import { useQuery } from 'react-query'
 
 // Make a specific query retry a certain number of times
 const result = useQuery(['todos', 1], fetchTodoListPage, {
   retry: 10, // Will retry failed requests 10 times before displaying an error
 })
```

* Retry Delay

* retryDelay 옵션으로 딜레이도 설정할 수 있다
* 기본적으로 1000ms에서 시작해서 retry할 때마다 2배씩 늘어난다 (단 30초를 넘진 않는다)
```
 // Configure for all queries
 import { QueryCache, QueryClient, QueryClientProvider } from 'react-query'
 
 const queryClient = new QueryClient({
   defaultOptions: {
     queries: {
       retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
     },
   },
 })
 
 function App() {
   return <QueryClientProvider client={queryClient}>...</QueryClientProvider>
 }
```

* 추천하진 않지만 retryDelay에 정수를 전달하면 항상 그 값만큰 딜레이된다

## Paginated/Lagged Qureies

* 쿼리 키에 page 변수를 넣음으로써 페이지네이션을 처리할 수 있다
* The UI jumps in and out of the success and loading states because each new page is treated like a brand new query

* Better Pagination with keepPreviousData

* page 값이 바뀔때마다 쿼리 키 값이 바뀌기 때문에 새로운 쿼리를 만들어낼 것이다
* keepPreviousData 옵션을 사용하면 이 문제를 해결할 수 있다
    * 쿼리키가 바뀌어도 새 데이터를 받아올 동안 이전에 받았던 데이터를 쓸 수 있다
    * 새로운 데이터를 가져오면 이전 데이터를 대체한다
    * isPreviousData로 이전 데이터인지 여부를 확인할 수 있다

```
function Todos() {
   const [page, setPage] = React.useState(0)
 
   const fetchProjects = (page = 0) => fetch('/api/projects?page=' + page).then((res) => res.json())
 
   const {
     isLoading,
     isError,
     error,
     data,
     isFetching,
     isPreviousData,
   } = useQuery(['projects', page], () => fetchProjects(page), { keepPreviousData : true })
 
   return (
     <div>
       {isLoading ? (
         <div>Loading...</div>
       ) : isError ? (
         <div>Error: {error.message}</div>
       ) : (
         <div>
           {data.projects.map(project => (
             <p key={project.id}>{project.name}</p>
           ))}
         </div>
       )}
       <span>Current Page: {page + 1}</span>
       <button
         onClick={() => setPage(old => Math.max(old - 1, 0))}
         disabled={page === 0}
       >
         Previous Page
       </button>{' '}
       <button
         onClick={() => {
           if (!isPreviousData && data.hasMore) {
             setPage(old => old + 1)
           }
         }}
         // Disable the Next Page button until we know a next page is available
         disabled={isPreviousData || !data?.hasMore}
       >
         Next Page
       </button>
       {isFetching ? <span> Loading...</span> : null}{' '}
     </div>
   )
 }
```

* Lagging Infinite Query results with keepPreviousData
    * keepPreviousData는 useInfinityQuery 훅과 사용할 수 있다

## Infinite Queries

*  인피니트 스크롤을 구현할 대 useInfinityQuery를 사용할 수 있다
*  useInfinityQuery는 useQuery가 다른 점이 몇가지 있다
    * data가 infinite query data를 가지고 있다
    * data.pages는 페이지들의 배열이다
    * data.pageParams는 데이터를 가져올 때 사용한 page 파라미터의 배열이다
    * fetchNextPage, fetchPreviousPage 함수가 제공된다
    * getNextPageParam, getPreviousPageParam 옵션이 제공된다. 더 불러올 데이터가 있는지 판단할 때 사용한다
    * getNextPageParam이 undefined가 아닌 다른 값을 반환하면 hasNextPage의 값이 true다 
    * getPreviousPageParam이 undefined가 아닌 다른 값을 반환하면 hasPrevioustPage의 값이 true다 
    * isFetchingNextPage와 isFetchingPreviousPage로 refreshing과 loading을 구분할 수 있다

* 더보기/인피니트 스크롤 구현하기
    1. useInfiniteQuery가 데이터 가져오는거 기다리기
    2. 다음 데이터를 가져오는데 필요한 정보를 getNextQueryParam이 반환하도록 하기
    3. fetchNextPage 함수 호출하기
        * pageParam을 오버라이드할 게 아니라면 절대 fetchNextPage에 인자를 넘겨선 안된다

```
import { useInfiniteQuery } from 'react-query'
 
 function Projects() {
   const fetchProjects = ({ pageParam = 0 }) =>
     fetch('/api/projects?cursor=' + pageParam)
 
   const {
     data,
     error,
     fetchNextPage,
     hasNextPage,
     isFetching,
     isFetchingNextPage,
     status,
   } = useInfiniteQuery('projects', fetchProjects, {
     getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
   })
 
   return status === 'loading' ? (
     <p>Loading...</p>
   ) : status === 'error' ? (
     <p>Error: {error.message}</p>
   ) : (
     <>
       {data.pages.map((group, i) => (
         <React.Fragment key={i}>
           {group.projects.map(project => (
             <p key={project.id}>{project.name}</p>
           ))}
         </React.Fragment>
       ))}
       <div>
         <button
           onClick={() => fetchNextPage()}
           disabled={!hasNextPage || isFetchingNextPage}
         >
           {isFetchingNextPage
             ? 'Loading more...'
             : hasNextPage
             ? 'Load More'
             : 'Nothing more to load'}
         </button>
       </div>
       <div>{isFetching && !isFetchingNextPage ? 'Fetching...' : null}</div>
     </>
   )
 }
```

* What happends when an infinite query needs to be refetched?
    * 첫 페이지부터 순서대로 refetch 된다

* What if I need to pass custom information to my query function?
    * You can pass custom variables to the fetchNextPage function which will override the default variable like so:
    ```
    function Projects() {
        const fetchProjects = ({ pageParam = 0 }) =>
            fetch('/api/projects?cursor=' + pageParam)

        const {
            status,
            data,
            isFetching,
            isFetchingNextPage,
            fetchNextPage,
            hasNextPage,
        } = useInfiniteQuery('projects', fetchProjects, {
            getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
        })

        // Pass your own page param
        const skipToCursor50 = () => fetchNextPage({ pageParam: 50 })
    }
    ```
* What if I want to implement bi-directional infinite list
    * getPreviousPageParam, fetchPreviousPage, hasPreviousPage, isFetchingPreviousPage를 사용하여 구현할 수 있다

* What if I want to show the pages in reversed order?
    * select 옵션을 사용하면 된다
    ```
    useInfiniteQuery('projects', fetchProjects, {
        select: data => ({
            pages: [...data.pages].reverse(),
            pageParams: [...data.pageParams].reverse(),
        }),
    })
    ```
* What if I want to manually update the infinite query?
    * queryClient.setQueryData를 사용하면 된다
    * 이때 반드시 page, pageParam 데이터 구조는 유지해야 한다

## Placeholder Query Data

* initialData 옵션과 비슷하지만 캐시에 저장되지 않는다

* placeholder 설정하기
    * Value
        ```
        function Todo() {
            const result = useQuery('todos', () => fetch('/todos'), {
                placeholderData: placeholderTodos,
            })
        }
        ```
    * Function  
        * 데이터를 메모이제이션 해도 되는 경우
        ```
        function Todos() {
            const placeholderData = useMemo(() => generateFakeTodos(), []);
            const result = useQuery('todos', () => fetch('/todos', { placehodlerData }))
        }
        ```
    * From Cache
        * 종종 캐시된 결과를 placeholder data로 써야 할 때가 있다
        ```
        function Todo({ blogPostId }) {
            const result = useQuery(['blogPost', blogPostId], () => fetch(`blogPosts/${blogPostId}`), {
                placeholderData: () => {
                    return queryClient.getQueryData('blogPosts)
                        ?.find(d => d.id === blogPostId)
                }
            })
        }
        ```
## Initial Query Data

* Using initialData to prepopulate a query
    * 쿼리를 실행하기 전에 이미 초기값을 가진경우가 있을 수 있다
    * initialData 옵션을 사용하면 된다
    ```
    function Todo() {
        const result = useQuery('todos', () => fetch('/todos'), {
            initialData,
        })
    }
    ```

* staleTime and initialDataUpdatedAt
    * By default, initialData is treated as totally fresh, as if it were just fetched
    * This also means that it will affect how it is interpreted by the staleTime option

    * 쿼리 옵저버에 initialData 옵션만 주고 staleTime을 주지 않으면 마운트 되자마자 바로 refetch된다
        (staleTime으 기본값이 0이기 때문이다)
    * 쿼리 옵저버에 initialData을 주고  staleTime을 1000ms로 설정하면 1000ms 이후에 refetch된다
    
    * So what if your initialData isn't totally fresh?
    * 이럴땐 initialDataUpdatedAt을 사용하면 된다
    * initialDataUpdatedAt에 initialData가 마지막으로 업데이트 된 시간의 타임스탬프를 넘길 수 있다

* Initial Data Function
    * 쿼리의 초기 데이터를 불러오는 작업이 intensive하다면 initialData에 함수를 전달해도 된다
    * 이 함수는 쿼리가 초기화 될 때만 한 번 실행된다
    ```
    function Todos() {
        const result = useQuery('todos', () => fetch('/todos'), {
            initialData: () => {
                return getExpensiveTodos();
            }
        })
    }
    ```

* Initial Data from Cache
    * 캐시를 초기 값으로 사용해야 될 때가 있을 수 있다
    ```
    function Todos({ todoId }) {
        const result = useQuery(['todos', todoId], () => fetch('/todos'), {
            initialData: () => {
                return queryClient.getQueryData('todos')?.find(d => d.id === todoId);
            }
        })
    }
    ```

* Initial Data from the cache with initialDataUpdatedAt
    * 캐시를 초기 값으로 설정하는 경우 staleTime 설정을 통해 인위적으로 refetching 하는 것보단 initialDataUpdatedAt에 dataUpdatedAt 값을 넘겨 refetch 여부를 쿼리 인스턴스가 판단하도록 두는게 좋다
    ```
    function Todos({ todoId }) {
        const result = useQuery(['todo', todoId], () => fetch('/todos/${todoId}'), {
            initialData: () => 
                queryClient.getQueryData('todos')?.find(d => d.id === todoId),
            initialDataUpdatedAt: () =>
                queryClient.getQueryState('todos')?.dataUpdatedAt
        })
    }
    ```

* Conditional Initial Data from Cache
    * 캐시가 너무 오래됐다면 초기값으로 쓰고 싶지 않을 수 있다
    * queryClient.getQueryState 메소드로 캐시가 언제 업데이트 됐는지 알 수 있다
    ```
    function Todo({ todoId }) {
        const result = useQuery(['todo', todoId], () => fetch('/todos/${todoId}'), {
            initialData: () => {
                const state = queryClient.getQueryState('todos');

                if(state && Date.now() - state.dataUpdatedAt <= 10*1000)
                    return state.data.find(d => d.id === todoId)
            }
        })
    }
    ```

## Prefetching

* 사용자가 무엇을 할 지 안다면 해당 데이터를 미리 불러오면 좋을 것이다
* queryClient.prefetchQuery 메소드를 사용하면 된다
```
const prefetchTodos = async () => {
    await queryClient.prefetchQuery('todos', fetchTodos);
}
```

* 만약 쿼리의 데이터가 캐시에 있고 갱신되지 않았으면 data will not be fetched
* staleTime이 전달됐고, 데이터가 주어진 staleTime 전에 가져온것이라면 fetch
* prefetched된 쿼리에 대한 useQuery가 없다면 cacheTime 이후에 prefetch됐던 데티어는 가비지 컬렉팅 된다

* Manually Priming a Query
    * queryClient.setQueryData로 캐시에 직접 데이터 추가할 수 있다

## Mutations

* 쿼리와 다르게 뮤테이션은 CUD를 수행한다
* React Query는 useMutation 훅을 제공한다
```
 function App() {
   const mutation = useMutation(newTodo => axios.post('/todos', newTodo))
 
   return (
     <div>
       {mutation.isLoading ? (
         'Adding todo...'
       ) : (
         <>
           {mutation.isError ? (
             <div>An error occurred: {mutation.error.message}</div>
           ) : null}
 
           {mutation.isSuccess ? <div>Todo added!</div> : null}
 
           <button
             onClick={() => {
               mutation.mutate({ id: new Date(), title: 'Do Laundry' })
             }}
           >
             Create Todo
           </button>
         </>
       )}
     </div>
   )
 }
```

* mutation과 onSuccess 옵션, query client의 invalidateQueries, setQueryData를 같이 사용하면 유용하다
```
// This will not work in React 16 and earlier
 const CreateTodo = () => {
   const mutation = useMutation(event => {
     event.preventDefault()
     return fetch('/api', new FormData(event.target))
   })
 
   return <form onSubmit={mutation.mutate}>...</form>
 }
 
 // This will work
 const CreateTodo = () => {
   const mutation = useMutation(formData => {
     return fetch('/api', formData)
   })
   const onSubmit = event => {
     event.preventDefault()
     mutation.mutate(new FormData(event.target))
   }
 
   return <form onSubmit={onSubmit}>...</form>
 }
```

* Reseting Mutation State
    * mutation의 error와 data를 초기화 시켜야하는 경우 reset 함수를 사용할 수 있다

    ```
    const CreateTodo = () => {
    const [title, setTitle] = useState('')
    const mutation = useMutation(createTodo)
    
    const onCreateTodo = e => {
        e.preventDefault()
        mutation.mutate({ title })
    }
    
    return (
        <form onSubmit={onCreateTodo}>
        {mutation.error && (
            <h5 onClick={() => mutation.reset()}>{mutation.error}</h5>
        )}
        <input
            type="text"
            value={title}
            onChange={e => setTitle(e.target.value)}
        />
        <br />
        <button type="submit">Create Todo</button>
        </form>
    )
    }
    ```

    * Mutation Side Effects
        * useMutate comes with some helper options that allow quick side effects at any stage during the mutation lifecycle
        * 위 기능은 쿼리를 갱신하고 refetching 할 때 유용하다
        ```
        useMutation(addTodo, {
            onMutate: variables => {
                // A mutation is about to happen!
            
                // Optionally return a context containing data to use when for example rolling back
                return { id: 1 }
            },
            onError: (error, variables, context) => {
                // An error happened!
                console.log(`rolling back optimistic update with id ${context.id}`)
            },
            onSuccess: (data, variables, context) => {
                // Boom baby!
            },
            onSettled: (data, error, variables, context) => {
                // Error or success... doesn't matter!
            },
        })
        ```
        
        * When returning a promise in any of the callback functions it will first be awaited before the next callback is called
        ```
        useMutation(addTodo, {
            onSuccess: async () => {
                console.log('Im first');
            },
            onSettled: async () => {
                console.log('Im second');
            }
        })
        ```
        * mutate 한 이후에도 콜백 함수를 사용할 수 있다
        ```
        useMutation(addTodo, {
            onSuccess: (data, variables, context) => {
                // I will fire first
            },
            onError: (error, variables, context) => {
                // I will fire first
            },
            onSettled: (data, error, variables, context) => {
                // I will fire first
            },
            })
            
            mutate(todo, {
            onSuccess: (data, variables, context) => {
                // I will fire second!
            },
            onError: (error, variables, context) => {
                // I will fire second!
            },
            onSettled: (data, error, variables, context) => {
                // I will fire second!
            },
        })
        ```

    * Promises
        * Use mutateAsync instead of mutate to get a promise which will resolve success or throw error
        ```
        const mutation = useMutation(addTodo)
 
        try {
            const todo = await mutation.mutateAsync(todo)
            console.log(todo)
        } catch (error) {
            console.error(error)
        } finally {
            console.log('done')
        }
        ```

    * Retry
        * 만약 네트워크가 끊겨서 mutation이 실패한거라면 다시 연결됐을 때 retry한다
        * 그 외의 이유 때문에 mutation 도중 에러가 발생하면 기본적으로 retry를 하지 않는다
        * retry 옵션을 넘기면 retry를 한다

    * Persist Mutations
        * Mutations can be persisted to storage if needed and resumed at a later point
        * This can be done with the hydration functions
        ```
        const queryClient = new QueryClient()
        
        // Define the "addTodo" mutation
        queryClient.setMutationDefaults('addTodo', {
        mutationFn: addTodo,
        onMutate: async (variables) => {
            // Cancel current queries for the todos list
            await queryClient.cancelQueries('todos')
        
            // Create optimistic todo
            const optimisticTodo = { id: uuid(), title: variables.title }
        
            // Add optimistic todo to todos list
            queryClient.setQueryData('todos', old => [...old, optimisticTodo])
        
            // Return context with the optimistic todo
            return { optimisticTodo }
        },
        onSuccess: (result, variables, context) => {
            // Replace optimistic todo in the todos list with the result
            queryClient.setQueryData('todos', old => old.map(todo => todo.id === context.optimisticTodo.id ? result : todo))
        },
        onError: (error, variables, context) => {
            // Remove optimistic todo from the todos list
            queryClient.setQueryData('todos', old => old.filter(todo => todo.id !== context.optimisticTodo.id))
        },
        retry: 3,
        })
        
        // Start mutation in some component:
        const mutation = useMutation('addTodo')
        mutation.mutate({ title: 'title' })
        
        // If the mutation has been paused because the device is for example offline,
        // Then the paused mutation can be dehydrated when the application quits:
        const state = dehydrate(queryClient)
        
        // The mutation can then be hydrated again when the application is started:
        hydrate(queryClient, state)
        
        // Resume the paused mutations:
        queryClient.resumePausedMutations()
        ```

## Query Invalidation

* queryClient.invalidateQueries lets you mark queries as stale and potentially refetch them
```
// invalidate all queries in cache
queryClient.invalidateQueries();
```

* When a query is invalidated with invalidateQueries, 2 things happen
    * It is marked as stale. This state overrides any staletime configuration used by useQuery or related hooks
    * If the query is currently being rendered, it will by refetched

* Query Matching with invalidateQueries
    * invalidateQueries, removeQueries를 사용할 대 prefix로 여러 쿼리를 지정할 수 있다
    * todos prefix로 todos로 시작하는 모든 쿼리를 invalidate할 수 있다
    ```
    import { useQuery, useQueryClient } from 'react-query'
    
    // Get QueryClient from the context
    const queryClient = useQueryClient()
    
    queryClient.invalidateQueries('todos')
    
    // Both queries below will be invalidated
    const todoListQuery = useQuery('todos', fetchTodoList)
    const todoListQuery = useQuery(['todos', { page: 1 }], fetchTodoList)
    ```

    * 더 정확하게 지정할 수도 있다
    ```
    queryClient.invalidateQueries(['todos', { type: 'done' }])
    
    // The query below will be invalidated
    const todoListQuery = useQuery(['todos', { type: 'done' }], fetchTodoList)
    
    // However, the following query below will NOT be invalidated
    const todoListQuery = useQuery('todos', fetchTodoList)
    ```

    * 만약 정확히 todos만 invalidate하고 싶다면 exact 옵션을 사용할 수 있다
    ```
    queryClient.invalidateQueries('todos', { exact })
    ```

    * 아래와 같이 predicate로 쿼리를 지정하는 방법도 있다
    ```
    queryClient.invalidateQueries({
        predicate: query =>
            query.queryKey[0] === 'todos' && query.queryKey[1]?.version >= 10,
    })
    
    // The query below will be invalidated
    const todoListQuery = useQuery(['todos', { version: 20 }], fetchTodoList)
    
    // The query below will be invalidated
    const todoListQuery = useQuery(['todos', { version: 10 }], fetchTodoList)
    
    // However, the following query below will NOT be invalidated
    const todoListQuery = useQuery(['todos', { version: 5 }], fetchTodoList)
    ```

## Invalidation from Mutations

* mutation이 성공하면 기존 데이터들을 invalidate 해야한다
* mutations의 onSuccess, queryClient.invalidateQueries를 같이 사용하면 된다 
```
import { useMutation, useQueryClient } from 'react-query'

const queryClient = useQueryClient()

// When this mutation succeeds, invalidate any queries with the `todos` or `reminders` query key
const mutation = useMutation(addTodo, {
    onSuccess: () => {
        queryClient.invalidateQueries('todos')
        queryClient.invalidateQueries('reminders')
    },
})
```

## Updates from Mutation Response

* 상태를 업데이트 하는 mutation의 경우, 일반적으로 수정된 상태를 응답으로 반환한다
* 쿼리를 refetching 하는 것보다, 응답으로 반환된 값을 사용하는게 훨씬 효율적이다
* queryClient.setQueryData를 사용하자
```
const queryClient = useQueryClient()
 
const mutation = useMutation(editTodo, {
    onSuccess: data => {
        queryClient.setQueryData(['todo', { id: 5 }], data)
    }
})

mutation.mutate({
    id: 5,
    name: 'Do the laundry',
})

// The query below will be updated with the response from the
// successful mutation
const { status, data, error } = useQuery(['todo', { id: 5 }], fetchTodoById)
```

* onSuccess 로직을 재사용하고 싶다면 아래와 같이 custom 훅을 만들어도 된다
```
const useMutateTodo = () => {
   const queryClient = useQueryClient()
 
   return useMutation(editTodo, {
     // Notice the second argument is the variables object that the `mutate` function receives
     onSuccess: (data, variables) => {
       queryClient.setQueryData(['todo', { id: variables.id }], data)
     },
   })
 }
```

## Optimistic Updates

* optimistic update 한 후 mutate가 실패한 경우 롤백해야 한다
* onMutate 콜백 함수가 onError/onSettled로 전달할 값을 반환한다 
```
const queryClient = useQueryClient()
 
 useMutation(updateTodo, {
   // When mutate is called:
   onMutate: async newTodo => {
     // Cancel any outgoing refetches (so they don't overwrite our optimistic update)
     await queryClient.cancelQueries('todos')
 
     // Snapshot the previous value
     const previousTodos = queryClient.getQueryData('todos')
 
     // Optimistically update to the new value
     queryClient.setQueryData('todos', old => [...old, newTodo])
 
     // Return a context object with the snapshotted value
     return { previousTodos }
   },
   // If the mutation fails, use the context returned from onMutate to roll back
   onError: (err, newTodo, context) => {
     queryClient.setQueryData('todos', context.previousTodos)
   },
   // Always refetch after error or success:
   onSettled: () => {
     queryClient.invalidateQueries('todos')
   },
 })
```

* Updating a single todo
```
 useMutation(updateTodo, {
   // When mutate is called:
   onMutate: async newTodo => {
     // Cancel any outgoing refetches (so they don't overwrite our optimistic update)
     await queryClient.cancelQueries(['todos', newTodo.id])
 
     // Snapshot the previous value
     const previousTodo = queryClient.getQueryData(['todos', newTodo.id])
 
     // Optimistically update to the new value
     queryClient.setQueryData(['todos', newTodo.id], newTodo)
 
     // Return a context with the previous and new todo
     return { previousTodo, newTodo }
   },
   // If the mutation fails, use the context we returned above
   onError: (err, newTodo, context) => {
     queryClient.setQueryData(
       ['todos', context.newTodo.id],
       context.previousTodo
     )
   },
   // Always refetch after error or success:
   onSettled: newTodo => {
     queryClient.invalidateQueries(['todos', newTodo.id])
   },
 })
```

## Query Cacellation

* 기본적으로 중간에 언마운트 되거나 promise가 resolve되기 전에 사용하지 않는 쿼리는 무시된다
* 왜 취소되지 않을까?
    * 대부분의 애플리케이션에선 단순히 무시하는 것만으로도 충분하다
    * 모든 쿼리가 취소 api를 제공하진 않을 수 있다
    * 만약 취소 api가 제공된다면 라이브러리(axios, fetch, etc)마다 구현이 다를 수 있다

* React Query는 CancelToken으로 쿼리를 취소할 수 있는 일반적인 방법을 제공한다
* 이 기능을 사용하기 위해선 쿼리가 반환한 promise에 cancel을 attach하면 된다

* Axios
```
 import { CancelToken } from 'axios'
 
 const query = useQuery('todos', () => {
   // Create a new CancelToken source for this request
   const source = CancelToken.source()
 
   const promise = axios.get('/todos', {
     // Pass the source token to your request
     cancelToken: source.token,
   })
 
   // Cancel the request if React Query calls the `promise.cancel` method
   promise.cancel = () => {
     source.cancel('Query was cancelled by React Query')
   }
 
   return promise
 })
```

* Fetch
```
const query = useQuery('todos', () => {
   // Create a new AbortController instance for this request
   const controller = new AbortController()
   // Get the abortController's signal
   const signal = controller.signal
 
   const promise = fetch('/todos', {
     method: 'get',
     // Pass the signal to your request
     signal,
   })
 
   // Cancel the request if React Query calls the `promise.cancel` method
   promise.cancel = () => controller.abort()
 
   return promise
 })
```

* Manual Cancellation
    * 쿼리를 직접 취소하고 싶을 수도 있다
        (api 취소 버튼, 요청이 너무 오래 걸리는 경우 등)
    * queryClient.cancelQueries(key)를 호출하면 된다
```
const [queryKey] = useState('todos')
 
 const query = useQuery(queryKey, () => {
   const controller = new AbortController()
   const signal = controller.signal
 
   const promise = fetch('/todos', {
     method: 'get',
     signal,
   })
 
   // Cancel the request if React Query calls the `promise.cancel` method
   promise.cancel = () => controller.abort()
 
   return promise
 })
 
 const queryClient = useQueryClient();
 
 return (
   <button onClick={(e) => {
     e.preventDefault();
     queryClient.cancelQueries(queryKey);
    }}>Cancel</button>
 )
```
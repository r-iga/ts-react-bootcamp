# TanStack Query の基礎

## 📖 概要

TanStack Query（旧 React Query）は、サーバーステートを管理するための強力なライブラリです。
データ取得・キャッシュ・同期・更新を自動的に処理してくれます。

## 🎯 学習目標

- TanStack Query の基本的な使い方を理解する
- useQuery でデータを取得できる
- useMutation でデータを更新できる

## 1. インストール

```bash
npm install @tanstack/react-query
```

## 2. セットアップ

### QueryClient の作成

```tsx
// main.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5分
      retry: 1,
    },
  },
});

ReactDOM.createRoot(document.getElementById('root')!).render(
  <QueryClientProvider client={queryClient}>
    <App />
  </QueryClientProvider>
);
```

## 3. useQuery - データ取得

### 基本的な使い方

```tsx
import { useQuery } from '@tanstack/react-query';

type User = {
  id: number;
  name: string;
  email: string;
};

function UserProfile({ userId }: { userId: number }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: async () => {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) {
        throw new Error('Failed to fetch user');
      }
      return response.json() as Promise<User>;
    },
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return <div>No data</div>;
  
  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  );
}
```

### queryKey の重要性

```tsx
// ✅ queryKey でキャッシュを識別
useQuery({
  queryKey: ['users'],              // すべてのユーザー
  queryFn: fetchUsers,
});

useQuery({
  queryKey: ['users', userId],      // 特定のユーザー
  queryFn: () => fetchUser(userId),
});

useQuery({
  queryKey: ['users', { page, limit }], // ページネーション
  queryFn: () => fetchUsers({ page, limit }),
});
```

## 4. useQuery のオプション

### enabled - 条件付き実行

```tsx
function UserPosts({ userId }: { userId: number | null }) {
  const { data: posts } = useQuery({
    queryKey: ['posts', userId],
    queryFn: () => fetchPosts(userId!),
    enabled: !!userId, // ✅ userId があるときだけ実行
  });
  
  return <div>{/* ... */}</div>;
}
```

### staleTime - データの新鮮度

```tsx
useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
  staleTime: 1000 * 60 * 5, // 5分間はキャッシュを使う
});
```

### refetchInterval - 自動再取得

```tsx
useQuery({
  queryKey: ['notifications'],
  queryFn: fetchNotifications,
  refetchInterval: 1000 * 30, // 30秒ごとに再取得
});
```

## 5. useMutation - データの更新

### 基本的な使い方

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

type CreateUserData = {
  name: string;
  email: string;
};

function CreateUserForm() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: async (data: CreateUserData) => {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });
      return response.json();
    },
    onSuccess: () => {
      // ✅ ユーザー一覧を再取得
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    mutation.mutate({ name, email });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="name" />
      <input name="email" />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating...' : 'Create User'}
      </button>
      {mutation.isError && <p>Error: {mutation.error.message}</p>}
      {mutation.isSuccess && <p>User created!</p>}
    </form>
  );
}
```

### Optimistic Updates - 楽観的更新

```tsx
function TodoItem({ todo }: { todo: Todo }) {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: async (id: number) => {
      await fetch(`/api/todos/${id}`, { method: 'DELETE' });
    },
    // ✅ 削除前にキャッシュを更新（即座にUIに反映）
    onMutate: async (id) => {
      await queryClient.cancelQueries({ queryKey: ['todos'] });
      
      const previousTodos = queryClient.getQueryData<Todo[]>(['todos']);
      
      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.filter(t => t.id !== id)
      );
      
      return { previousTodos };
    },
    // ✅ エラー時は元に戻す
    onError: (err, id, context) => {
      queryClient.setQueryData(['todos'], context?.previousTodos);
    },
    // ✅ 完了後は再取得
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
  
  return (
    <div>
      <span>{todo.text}</span>
      <button onClick={() => mutation.mutate(todo.id)}>Delete</button>
    </div>
  );
}
```

## 6. キャッシュの操作

### invalidateQueries - 無効化

```tsx
const queryClient = useQueryClient();

// ✅ 特定のクエリを無効化
queryClient.invalidateQueries({ queryKey: ['users'] });

// ✅ 部分一致で無効化
queryClient.invalidateQueries({ queryKey: ['users'] }); // ['users', 1], ['users', 2] も無効化
```

### setQueryData - 直接更新

```tsx
const queryClient = useQueryClient();

// ✅ キャッシュを直接更新
queryClient.setQueryData<User>(['user', userId], (old) => ({
  ...old!,
  name: 'New Name',
}));
```

### getQueryData - データ取得

```tsx
const queryClient = useQueryClient();

// ✅ キャッシュからデータを取得
const user = queryClient.getQueryData<User>(['user', userId]);
```

## 7. カスタムフックで再利用

### useUser フック

```tsx
// hooks/useUser.ts
export function useUser(userId: number) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: async () => {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) throw new Error('Failed to fetch user');
      return response.json() as Promise<User>;
    },
  });
}

// コンポーネントで使用
function UserProfile({ userId }: { userId: number }) {
  const { data: user, isLoading } = useUser(userId);
  
  if (isLoading) return <div>Loading...</div>;
  
  return <div>{user?.name}</div>;
}
```

### useTodos フック

```tsx
// hooks/useTodos.ts
export function useTodos() {
  return useQuery({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('/api/todos');
      return response.json() as Promise<Todo[]>;
    },
  });
}

export function useCreateTodo() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (text: string) => {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text }),
      });
      return response.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}

export function useDeleteTodo() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (id: number) => {
      await fetch(`/api/todos/${id}`, { method: 'DELETE' });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}

// 使用
function TodoList() {
  const { data: todos, isLoading } = useTodos();
  const createTodo = useCreateTodo();
  const deleteTodo = useDeleteTodo();
  
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <div>
      {todos?.map(todo => (
        <div key={todo.id}>
          <span>{todo.text}</span>
          <button onClick={() => deleteTodo.mutate(todo.id)}>Delete</button>
        </div>
      ))}
    </div>
  );
}
```

## 8. ページネーション

### useQuery でページネーション

```tsx
function UserList() {
  const [page, setPage] = useState(1);
  
  const { data, isLoading } = useQuery({
    queryKey: ['users', page],
    queryFn: () => fetchUsers(page),
    keepPreviousData: true, // ✅ 前のデータを保持
  });
  
  return (
    <div>
      {data?.users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
      
      <button
        onClick={() => setPage(p => p - 1)}
        disabled={page === 1}
      >
        Previous
      </button>
      
      <span>Page {page}</span>
      
      <button
        onClick={() => setPage(p => p + 1)}
        disabled={!data?.hasMore}
      >
        Next
      </button>
    </div>
  );
}
```

### useInfiniteQuery - 無限スクロール

```tsx
function InfiniteUserList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['users'],
    queryFn: ({ pageParam = 1 }) => fetchUsers(pageParam),
    getNextPageParam: (lastPage) => lastPage.nextPage,
  });
  
  return (
    <div>
      {data?.pages.map((page, i) => (
        <div key={i}>
          {page.users.map(user => (
            <div key={user.id}>{user.name}</div>
          ))}
        </div>
      ))}
      
      {hasNextPage && (
        <button onClick={() => fetchNextPage()} disabled={isFetchingNextPage}>
          {isFetchingNextPage ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
}
```

## 9. DevTools

### React Query DevTools

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

## ✅ ベストプラクティス

### 1. カスタムフックで抽象化

```tsx
// ✅ 再利用可能
export function useUser(id: number) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
  });
}
```

### 2. queryKey を統一

```tsx
// ✅ queryKeys を定義
export const queryKeys = {
  users: ['users'] as const,
  user: (id: number) => ['users', id] as const,
  posts: ['posts'] as const,
  post: (id: number) => ['posts', id] as const,
};

// 使用
useQuery({
  queryKey: queryKeys.user(userId),
  queryFn: () => fetchUser(userId),
});
```

### 3. エラーハンドリング

```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: (error) => {
        console.error('Query error:', error);
        toast.error('データの取得に失敗しました');
      },
    },
  },
});
```

## 📝 練習問題

### 問題1: 商品一覧と削除

商品一覧を表示し、削除ボタンで商品を削除できるようにしてください。

<details>
<parameter name="summary">解答例</summary>

```tsx
function ProductList() {
  const { data: products, isLoading } = useQuery({
    queryKey: ['products'],
    queryFn: async () => {
      const response = await fetch('/api/products');
      return response.json();
    },
  });
  
  const queryClient = useQueryClient();
  
  const deleteMutation = useMutation({
    mutationFn: async (id: number) => {
      await fetch(`/api/products/${id}`, { method: 'DELETE' });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
  
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <ul>
      {products?.map(product => (
        <li key={product.id}>
          {product.name}
          <button onClick={() => deleteMutation.mutate(product.id)}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```
</details>

## 🔗 次のステップ

TanStack Query を使うことで、データ取得のロジックをシンプルに保ちながら、堅牢なアプリケーションを構築できます。

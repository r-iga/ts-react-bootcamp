# Chapter 9: エコシステム

## 🎯 学習目標

- React エコシステムの主要なライブラリを理解する
- ルーティングの実装方法を習得する
- データフェッチングのベストプラクティスを学ぶ
- UI ライブラリの使い方を習得する
- 実務で使われるツールを活用できるようになる

## 📚 学習トピック

### 9.1 React エコシステム概要

**React 周辺のライブラリ分類:**
- **ルーティング:** React Router, TanStack Router
- **状態管理:** Zustand, Redux, Jotai, Recoil
- **データフェッチング:** TanStack Query, SWR, Apollo Client
- **フォーム管理:** React Hook Form, Formik
- **UI ライブラリ:** Material-UI (MUI), Chakra UI, shadcn/ui
- **スタイリング:** Tailwind CSS, Styled Components, Emotion
- **テスト:** Vitest, React Testing Library, Playwright
- **ビルドツール:** Vite, Next.js, Remix

**ライブラリの選定基準:**
- アクティブなメンテナンス
- コミュニティのサイズ
- ドキュメントの質
- TypeScript サポート
- バンドルサイズ
- プロジェクトの要件

### 9.2 React Router

**React Router とは:**
- React アプリケーションのルーティングライブラリ
- SPA (Single Page Application) のページ遷移
- クライアントサイドルーティング

**基本的なセットアップ:**
```tsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

const App = () => {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="/users/:id" element={<UserPage />} />
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </BrowserRouter>
  );
};
```

**ナビゲーション:**
```tsx
import { Link, NavLink, useNavigate } from 'react-router-dom';

// Link コンポーネント
<Link to="/about">About</Link>

// NavLink（アクティブ状態の判定）
<NavLink 
  to="/about" 
  className={({ isActive }) => isActive ? 'active' : ''}
>
  About
</NavLink>

// プログラマティックナビゲーション
const Component = () => {
  const navigate = useNavigate();
  
  const handleClick = () => {
    navigate('/about');
    // navigate(-1); // 戻る
    // navigate('/login', { replace: true }); // リダイレクト
  };
  
  return <button onClick={handleClick}>Go to About</button>;
};
```

**パラメータとクエリ:**
```tsx
import { useParams, useSearchParams } from 'react-router-dom';

// URLパラメータ: /users/:id
const UserPage = () => {
  const { id } = useParams<{ id: string }>();
  return <div>User ID: {id}</div>;
};

// クエリパラメータ: /search?q=react
const SearchPage = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get('q');
  
  return (
    <div>
      <p>Searching for: {query}</p>
      <button onClick={() => setSearchParams({ q: 'new-query' })}>
        Update Query
      </button>
    </div>
  );
};
```

**ネストルートとレイアウト:**
```tsx
import { Outlet } from 'react-router-dom';

const App = () => {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<HomePage />} />
        <Route path="about" element={<AboutPage />} />
        <Route path="users" element={<UsersLayout />}>
          <Route index element={<UsersList />} />
          <Route path=":id" element={<UserDetail />} />
        </Route>
      </Route>
    </Routes>
  );
};

const Layout = () => {
  return (
    <div>
      <header>Header</header>
      <main>
        <Outlet /> {/* 子ルートがここに表示される */}
      </main>
      <footer>Footer</footer>
    </div>
  );
};
```

**保護されたルート:**
```tsx
const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const { user } = useAuth();
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  return <>{children}</>;
};

// 使用例
<Route 
  path="/dashboard" 
  element={
    <ProtectedRoute>
      <DashboardPage />
    </ProtectedRoute>
  } 
/>
```

**Loader と Action (React Router v6.4+):**
```tsx
import { createBrowserRouter, useLoaderData } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/users/:id',
    element: <UserPage />,
    loader: async ({ params }) => {
      const response = await fetch(`/api/users/${params.id}`);
      return response.json();
    },
  },
]);

const UserPage = () => {
  const user = useLoaderData() as User;
  return <div>{user.name}</div>;
};
```

### 9.3 TanStack Query (旧 React Query)

**TanStack Query とは:**
- サーバー状態管理ライブラリ
- データフェッチング、キャッシング、同期
- 自動的な再取得、バックグラウンド更新
- 楽観的更新、ページネーション、無限スクロール

**セットアップ:**
```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
    </QueryClientProvider>
  );
};
```

**useQuery - データ取得:**
```tsx
import { useQuery } from '@tanstack/react-query';

const Users = () => {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const response = await fetch('/api/users');
      return response.json();
    },
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};
```

**useMutation - データ更新:**
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

const AddUser = () => {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: async (newUser: User) => {
      const response = await fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(newUser),
      });
      return response.json();
    },
    onSuccess: () => {
      // キャッシュを無効化して再取得
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    mutation.mutate({ name: 'New User' });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Adding...' : 'Add User'}
      </button>
      {mutation.isError && <div>Error: {mutation.error.message}</div>}
    </form>
  );
};
```

**キャッシングと再取得:**
```tsx
const { data } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
  staleTime: 5 * 60 * 1000, // 5分間は新鮮とみなす
  gcTime: 10 * 60 * 1000, // 10分後にキャッシュを削除
  refetchOnWindowFocus: true, // ウィンドウフォーカス時に再取得
});
```

**楽観的更新:**
```tsx
const mutation = useMutation({
  mutationFn: updateUser,
  onMutate: async (newUser) => {
    // 進行中のクエリをキャンセル
    await queryClient.cancelQueries({ queryKey: ['users'] });
    
    // 前の値を保存
    const previousUsers = queryClient.getQueryData(['users']);
    
    // 楽観的に更新
    queryClient.setQueryData(['users'], (old: User[]) => 
      old.map(user => user.id === newUser.id ? newUser : user)
    );
    
    return { previousUsers };
  },
  onError: (err, newUser, context) => {
    // エラー時にロールバック
    queryClient.setQueryData(['users'], context?.previousUsers);
  },
  onSettled: () => {
    // 最終的に再取得
    queryClient.invalidateQueries({ queryKey: ['users'] });
  },
});
```

### 9.4 状態管理ライブラリ

**Zustand:**
```tsx
import { create } from 'zustand';

type CounterStore = {
  count: number;
  increment: () => void;
  decrement: () => void;
};

const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

const Counter = () => {
  const { count, increment, decrement } = useCounterStore();
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
};
```

**いつ状態管理ライブラリが必要か:**
- 複数のページで共有する状態
- Context の深いネストを避けたい
- パフォーマンスが重要
- Redux のような予測可能な状態管理が必要

### 9.5 フォーム管理 - React Hook Form

**React Hook Form とは:**
- パフォーマンスに優れたフォームライブラリ
- 最小限の再レンダリング
- バリデーション機能
- TypeScript サポート

**基本的な使い方:**
```tsx
import { useForm } from 'react-hook-form';

type FormData = {
  name: string;
  email: string;
  age: number;
};

const MyForm = () => {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormData>();
  
  const onSubmit = (data: FormData) => {
    console.log(data);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input 
        {...register('name', { required: 'Name is required' })} 
        placeholder="Name"
      />
      {errors.name && <span>{errors.name.message}</span>}
      
      <input 
        {...register('email', { 
          required: 'Email is required',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: 'Invalid email address'
          }
        })} 
        placeholder="Email"
      />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input 
        type="number"
        {...register('age', { 
          min: { value: 18, message: 'Must be 18 or older' } 
        })} 
        placeholder="Age"
      />
      {errors.age && <span>{errors.age.message}</span>}
      
      <button type="submit">Submit</button>
    </form>
  );
};
```

**バリデーションスキーマ (Zod):**
```tsx
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email'),
  age: z.number().min(18, 'Must be 18 or older'),
});

type FormData = z.infer<typeof schema>;

const MyForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });
  
  // ...
};
```

### 9.6 UI ライブラリ

**Material-UI (MUI):**
```tsx
import { Button, TextField, Box } from '@mui/material';

const LoginForm = () => {
  return (
    <Box sx={{ display: 'flex', flexDirection: 'column', gap: 2 }}>
      <TextField label="Email" type="email" />
      <TextField label="Password" type="password" />
      <Button variant="contained">Login</Button>
    </Box>
  );
};
```

**shadcn/ui:**
- コピー&ペーストで使えるコンポーネント集
- Tailwind CSS ベース
- カスタマイズしやすい

**UI ライブラリの選択:**
- デザインシステムの一貫性
- カスタマイズ性
- アクセシビリティ
- バンドルサイズ
- プロジェクトの要件

### 9.7 Tailwind CSS

**Tailwind CSS とは:**
- ユーティリティファーストの CSS フレームワーク
- クラス名でスタイリング
- レスポンシブデザインが簡単
- カスタマイズ性が高い

**基本的な使い方:**
```tsx
const Card = () => {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition">
      <h2 className="text-2xl font-bold mb-4">Title</h2>
      <p className="text-gray-600">Content</p>
      <button className="mt-4 px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
        Click me
      </button>
    </div>
  );
};
```

**レスポンシブデザイン:**
```tsx
<div className="w-full md:w-1/2 lg:w-1/3">
  {/* モバイル: 100%, タブレット: 50%, デスクトップ: 33.33% */}
</div>
```

**カスタムスタイルの追加:**
```tsx
// tailwind.config.js
export default {
  theme: {
    extend: {
      colors: {
        primary: '#3490dc',
        secondary: '#ffed4e',
      },
    },
  },
};

// 使用
<div className="bg-primary text-white">Custom color</div>
```

### 9.8 テスト

**Vitest + React Testing Library:**
```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import Counter from './Counter';

describe('Counter', () => {
  it('increments count when button is clicked', () => {
    render(<Counter />);
    
    const button = screen.getByRole('button', { name: /increment/i });
    const count = screen.getByText(/count: 0/i);
    
    expect(count).toBeInTheDocument();
    
    fireEvent.click(button);
    
    expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
  });
});
```

### 9.9 その他の便利なライブラリ

**日付操作:**
- date-fns, Day.js

**バリデーション:**
- Zod, Yup

**アニメーション:**
- Framer Motion, React Spring

**アイコン:**
- React Icons, Lucide React

**ユーティリティ:**
- clsx, classnames - クラス名の結合
- immer - イミュータブル更新

## 🔗 関連リソース

- [examples/09-ecosystem/](../../examples/09-ecosystem/) - サンプルコード
- [exercises/09-ecosystem/](../../exercises/09-ecosystem/) - 演習問題

## ⏱️ 推奨学習時間

**1-2週間（15-20時間）**

## ✅ チェックリスト

- [ ] React Router でルーティングを実装できる
- [ ] TanStack Query でデータフェッチングができる
- [ ] 状態管理ライブラリの使い分けができる
- [ ] React Hook Form でフォームを実装できる
- [ ] UI ライブラリを使ってコンポーネントを構築できる
- [ ] Tailwind CSS でスタイリングができる
- [ ] 適切なライブラリを選定できる
- [ ] React Testing Library でテストを書ける

## 📝 次のステップ

[最終プロジェクト](../../projects/) - 学んだ内容を統合した実践的なアプリケーション開発

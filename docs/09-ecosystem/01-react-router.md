# React Router の基礎

## 📖 概要

React Router は、React アプリケーションでページ遷移（ルーティング）を実現するライブラリです。
SPAでありながら、URLベースのナビゲーションを提供します。

## 🎯 学習目標

- React Router の基本的な使い方を理解する
- ルート定義ができる
- ナビゲーションを実装できる

## 1. インストール

```bash
npm install react-router-dom
```

## 2. 基本的なセットアップ

### BrowserRouter

```tsx
// main.tsx
import { BrowserRouter } from 'react-router-dom';
import { App } from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

### Routes と Route

```tsx
// App.tsx
import { Routes, Route } from 'react-router-dom';
import { HomePage } from './pages/HomePage';
import { AboutPage } from './pages/AboutPage';
import { ContactPage } from './pages/ContactPage';

const App = () => {
  return (
    <Routes>
      <Route path="/" element={<HomePage />} />
      <Route path="/about" element={<AboutPage />} />
      <Route path="/contact" element={<ContactPage />} />
    </Routes>
  );
};
```

## 3. ナビゲーション

### Link コンポーネント

```tsx
import { Link } from 'react-router-dom';

const Navigation = () => {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/contact">Contact</Link>
    </nav>
  );
};
```

### NavLink - アクティブ状態

```tsx
import { NavLink } from 'react-router-dom';

const Navigation = () => {
  return (
    <nav>
      <NavLink
        to="/"
        className={({ isActive }) => isActive ? 'active' : ''}
      >
        Home
      </NavLink>
      <NavLink
        to="/about"
        style={({ isActive }) => ({
          color: isActive ? 'red' : 'black'
        })}
      >
        About
      </NavLink>
    </nav>
  );
};
```

## 4. 動的ルート

### パラメータの定義

```tsx
import { Routes, Route } from 'react-router-dom';

const App = () => {
  return (
    <Routes>
      <Route path="/users/:userId" element={<UserProfile />} />
      <Route path="/posts/:postId" element={<PostDetail />} />
      <Route path="/products/:productId/reviews/:reviewId" element={<ReviewDetail />} />
    </Routes>
  );
};
```

### useParams でパラメータを取得

```tsx
import { useParams } from 'react-router-dom';

const UserProfile = () => {
  const { userId } = useParams<{ userId: string }>();
  
  return <div>User ID: {userId}</div>;
};

const PostDetail = () => {
  const { postId } = useParams<{ postId: string }>();
  const { data: post } = useFetch<Post>(`/api/posts/${postId}`);
  
  if (!post) return <div>Loading...</div>;
  
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
};
```

## 5. ネストしたルート

### レイアウトコンポーネント

```tsx
import { Outlet } from 'react-router-dom';

const Layout = () => {
  return (
    <div>
      <header>
        <Navigation />
      </header>
      
      <main>
        <Outlet /> {/* 子ルートがここに表示される */}
      </main>
      
      <footer>
        <p>&copy; 2024 My App</p>
      </footer>
    </div>
  );
};
```

### ネストしたルート定義

```tsx
const App = () => {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<HomePage />} />
        <Route path="about" element={<AboutPage />} />
        <Route path="contact" element={<ContactPage />} />
        
        <Route path="users" element={<UsersLayout />}>
          <Route index element={<UserList />} />
          <Route path=":userId" element={<UserProfile />} />
          <Route path=":userId/edit" element={<UserEdit />} />
        </Route>
      </Route>
    </Routes>
  );
};
```

## 6. プログラムによるナビゲーション

### useNavigate

```tsx
import { useNavigate } from 'react-router-dom';

const LoginForm = () => {
  const navigate = useNavigate();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    const success = await login();
    
    if (success) {
      // ✅ ログイン成功後にダッシュボードへ
      navigate('/dashboard');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* フォームフィールド */}
      <button type="submit">Login</button>
    </form>
  );
};
```

### 履歴操作

```tsx
const Component = () => {
  const navigate = useNavigate();
  
  // 前のページに戻る
  const goBack = () => navigate(-1);
  
  // 次のページに進む
  const goForward = () => navigate(1);
  
  // 置き換え（履歴に追加しない）
  const replace = () => navigate('/home', { replace: true });
  
  // State を渡す
  const navigateWithState = () => {
    navigate('/profile', { state: { from: 'home' } });
  };
  
  return (
    <div>
      <button onClick={goBack}>Back</button>
      <button onClick={goForward}>Forward</button>
    </div>
  );
};
```

## 7. クエリパラメータ

### useSearchParams

```tsx
import { useSearchParams } from 'react-router-dom';

const ProductList = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const category = searchParams.get('category');
  const sort = searchParams.get('sort');
  const page = searchParams.get('page') || '1';
  
  const updateCategory = (newCategory: string) => {
    setSearchParams({ category: newCategory, sort, page });
  };
  
  return (
    <div>
      <p>Category: {category}</p>
      <p>Sort: {sort}</p>
      <p>Page: {page}</p>
      
      <button onClick={() => updateCategory('electronics')}>
        Electronics
      </button>
      <button onClick={() => updateCategory('books')}>
        Books
      </button>
    </div>
  );
};

// URL: /products?category=electronics&sort=price&page=1
```

## 8. 404 ページ

### Not Found ルート

```tsx
const App = () => {
  return (
    <Routes>
      <Route path="/" element={<HomePage />} />
      <Route path="/about" element={<AboutPage />} />
      
      {/* ✅ 最後に配置 */}
      <Route path="*" element={<NotFoundPage />} />
    </Routes>
  );
};

const NotFoundPage = () => {
  return (
    <div>
      <h1>404 - Page Not Found</h1>
      <Link to="/">Go to Home</Link>
    </div>
  );
};
```

## 9. Protected Routes

### 認証が必要なルート

```tsx
import { Navigate } from 'react-router-dom';

const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const { isAuthenticated } = useAuth();
  
  if (!isAuthenticated) {
    // ✅ ログインページにリダイレクト
    return <Navigate to="/login" replace />;
  }
  
  return <>{children}</>;
};

// 使用
const App = () => {
  return (
    <Routes>
      <Route path="/login" element={<LoginPage />} />
      
      <Route
        path="/dashboard"
        element={
          <ProtectedRoute>
            <DashboardPage />
          </ProtectedRoute>
        }
      />
      
      <Route
        path="/profile"
        element={
          <ProtectedRoute>
            <ProfilePage />
          </ProtectedRoute>
        }
      />
    </Routes>
  );
};
```

## 10. ローディング状態

### Suspense と Lazy Loading

```tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// ✅ コード分割
const HomePage = lazy(() => import('./pages/HomePage'));
const AboutPage = lazy(() => import('./pages/AboutPage'));
const ContactPage = lazy(() => import('./pages/ContactPage'));

const App = () => {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="/contact" element={<ContactPage />} />
      </Routes>
    </Suspense>
  );
};
```

## 11. useLocation

### 現在の location 情報を取得

```tsx
import { useLocation } from 'react-router-dom';

const Component = () => {
  const location = useLocation();
  
  console.log(location.pathname); // "/users/123"
  console.log(location.search);   // "?tab=profile"
  console.log(location.hash);     // "#section1"
  console.log(location.state);    // Navigate で渡された state
  
  return <div>Current path: {location.pathname}</div>;
};
```

### 前のページの情報を取得

```tsx
const LoginPage = () => {
  const navigate = useNavigate();
  const location = useLocation();
  
  const from = location.state?.from || '/';
  
  const handleLogin = async () => {
    await login();
    // ✅ ログイン前のページに戻る
    navigate(from, { replace: true });
  };
  
  return <button onClick={handleLogin}>Login</button>;
};
```

## ✅ ベストプラクティス

### 1. Link を使う

```tsx
// ✅ Link コンポーネント
<Link to="/about">About</Link>

// ❌ a タグ（ページ全体がリロードされる）
<a href="/about">About</a>
```

### 2. ルートを整理

```tsx
// ✅ ルートを別ファイルに
// routes.tsx
export const routes = [
  { path: '/', element: <HomePage /> },
  { path: '/about', element: <AboutPage /> },
  { path: '/contact', element: <ContactPage /> }
];

// App.tsx
import { routes } from './routes';

const App = () => {
  return (
    <Routes>
      {routes.map(route => (
        <Route key={route.path} {...route} />
      ))}
    </Routes>
  );
};
```

### 3. TypeScript で型安全に

```tsx
// ✅ params の型を指定
const { userId } = useParams<{ userId: string }>();

// ✅ navigate の型を明示
const navigate = useNavigate();
navigate('/users/123');
```

## 📝 練習問題

### 問題1: ユーザー詳細ページ

ユーザー一覧ページとユーザー詳細ページを作成してください。
一覧からクリックして詳細に移動できるようにしてください。

<details>
<summary>解答例</summary>

```tsx
// App.tsx
const App = () => {
  return (
    <Routes>
      <Route path="/users" element={<UserList />} />
      <Route path="/users/:userId" element={<UserDetail />} />
    </Routes>
  );
};

// UserList.tsx
const UserList = () => {
  const users = [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" },
    { id: 3, name: "Charlie" }
  ];
  
  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            <Link to={`/users/${user.id}`}>{user.name}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
};

// UserDetail.tsx
const UserDetail = () => {
  const { userId } = useParams<{ userId: string }>();
  
  return (
    <div>
      <h1>User Detail</h1>
      <p>User ID: {userId}</p>
      <Link to="/users">Back to list</Link>
    </div>
  );
};
```
</details>

## 🔗 次のステップ

次は [TanStack Query によるデータ取得](./02-tanstack-query.md) について学びます。

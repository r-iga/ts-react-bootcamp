# Context API による状態管理

## 📖 概要

Context API は、コンポーネントツリー全体でデータを共有するための仕組みです。
Props Drilling（プロップスのバケツリレー）を避け、効率的に状態を管理できます。

## 🎯 学習目標

- Context の作成と使用ができる
- Props Drilling を回避できる
- 適切な粒度で Context を設計できる

## 1. 基本的な使い方

### Context の作成

```tsx
import { createContext } from 'react';

type ThemeContextType = {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
};

export const ThemeContext = createContext<ThemeContextType | null>(null);
```

### Provider でデータを提供

```tsx
const App = () => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <Layout />
    </ThemeContext.Provider>
  );
};
```

### useContext でデータを取得

```tsx
import { useContext } from 'react';

const ThemeToggleButton = () => {
  const context = useContext(ThemeContext);
  
  if (!context) {
    throw new Error('ThemeContext must be used within ThemeProvider');
  }
  
  const { theme, toggleTheme } = context;
  
  return (
    <button onClick={toggleTheme}>
      Current theme: {theme}
    </button>
  );
};
```

## 2. カスタムフックでラップ

### useTheme hook

```tsx
// ThemeContext.tsx
import { createContext, useContext, useState } from 'react';

type ThemeContextType = {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
};

const ThemeContext = createContext<ThemeContextType | null>(null);

// ✅ カスタムフックでラップ
export function useTheme() {
  const context = useContext(ThemeContext);
  
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  
  return context;
}

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 使用側
const Component = () => {
  const { theme, toggleTheme } = useTheme(); // ✅ シンプル
  
  return <button onClick={toggleTheme}>{theme}</button>;
};
```

## 3. 認証 Context の実装

### AuthContext

```tsx
// AuthContext.tsx
import { createContext, useContext, useState, useEffect } from 'react';

type User = {
  id: number;
  name: string;
  email: string;
};

type AuthContextType = {
  user: User | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  loading: boolean;
};

const AuthContext = createContext<AuthContextType | null>(null);

export function useAuth() {
  const context = useContext(AuthContext);
  
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  
  return context;
}

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // ✅ 初期化：ローカルストレージからユーザー情報を復元
    const savedUser = localStorage.getItem('user');
    if (savedUser) {
      setUser(JSON.parse(savedUser));
    }
    setLoading(false);
  }, []);
  
  const login = async (email: string, password: string) => {
    setLoading(true);
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });
      
      const userData = await response.json();
      setUser(userData);
      localStorage.setItem('user', JSON.stringify(userData));
    } finally {
      setLoading(false);
    }
  };
  
  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
  };
  
  const value = {
    user,
    isAuthenticated: !!user,
    login,
    logout,
    loading
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}
```

### 使用例

```tsx
// App.tsx
const App = () => {
  return (
    <AuthProvider>
      <Router>
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
        </Routes>
      </Router>
    </AuthProvider>
  );
};

// LoginPage.tsx
const LoginPage = () => {
  const { login } = useAuth();
  const navigate = useNavigate();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    await login(email, password);
    navigate('/dashboard');
  };
  
  return <form onSubmit={handleSubmit}>{/* ... */}</form>;
};

// DashboardPage.tsx
const DashboardPage = () => {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <h1>Welcome, {user?.name}!</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
};

// ProtectedRoute.tsx
const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const { isAuthenticated, loading } = useAuth();
  
  if (loading) return <div>Loading...</div>;
  
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return <>{children}</>;
};
```

## 4. 複数の Context を組み合わせる

### Provider のネスト

```tsx
const App = () => {
  return (
    <AuthProvider>
      <ThemeProvider>
        <LanguageProvider>
          <Router>
            <Routes>{/* ... */}</Routes>
          </Router>
        </LanguageProvider>
      </ThemeProvider>
    </AuthProvider>
  );
};
```

### 複合 Provider

```tsx
// AppProviders.tsx
export const AppProviders = ({ children }: { children: React.ReactNode }) => {
  return (
    <AuthProvider>
      <ThemeProvider>
        <LanguageProvider>
          {children}
        </LanguageProvider>
      </ThemeProvider>
    </AuthProvider>
  );
};

// App.tsx
const App = () => {
  return (
    <AppProviders>
      <Router>
        <Routes>{/* ... */}</Routes>
      </Router>
    </AppProviders>
  );
};
```

## 5. パフォーマンスの最適化

### 問題：Context が変わると全て再レンダリング

```tsx
// ❌ どちらかが変わると両方のコンポーネントが再レンダリング
type AppContextType = {
  user: User | null;
  theme: 'light' | 'dark';
  setUser: (user: User) => void;
  setTheme: (theme: 'light' | 'dark') => void;
};
```

### 解決策1：Context を分ける

```tsx
// ✅ 独立した Context
<UserContext.Provider>
  <ThemeContext.Provider>
    {children}
  </ThemeContext.Provider>
</UserContext.Provider>
```

### 解決策2：値と更新を分ける

```tsx
const StateContext = createContext<State | null>(null);
const DispatchContext = createContext<Dispatch | null>(null);

const Provider = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
};
```

### 解決策3：useMemo で値をメモ化

```tsx
const ThemeProvider = ({ children }: { children: React.ReactNode }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  // ✅ 値をメモ化
  const value = useMemo(
    () => ({
      theme,
      toggleTheme: () => setTheme(prev => prev === 'light' ? 'dark' : 'light')
    }),
    [theme]
  );
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

## 6. Context + useReducer

### より複雑な状態管理

```tsx
// TodoContext.tsx
type Todo = {
  id: number;
  text: string;
  completed: boolean;
};

type State = {
  todos: Todo[];
};

type Action =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: number }
  | { type: 'DELETE_TODO'; payload: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        todos: [
          ...state.todos,
          { id: Date.now(), text: action.payload, completed: false }
        ]
      };
    
    case 'TOGGLE_TODO':
      return {
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    
    case 'DELETE_TODO':
      return {
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    
    default:
      return state;
  }
}

const TodoStateContext = createContext<State | null>(null);
const TodoDispatchContext = createContext<React.Dispatch<Action> | null>(null);

export function TodoProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, { todos: [] });
  
  return (
    <TodoStateContext.Provider value={state}>
      <TodoDispatchContext.Provider value={dispatch}>
        {children}
      </TodoDispatchContext.Provider>
    </TodoStateContext.Provider>
  );
}

export function useTodoState() {
  const context = useContext(TodoStateContext);
  if (!context) throw new Error('useTodoState must be used within TodoProvider');
  return context;
}

export function useTodoDispatch() {
  const context = useContext(TodoDispatchContext);
  if (!context) throw new Error('useTodoDispatch must be used within TodoProvider');
  return context;
}

// 使用例
const TodoList = () => {
  const { todos } = useTodoState();
  const dispatch = useTodoDispatch();
  
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}
          />
          <span>{todo.text}</span>
          <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: todo.id })}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
};

const TodoForm = () => {
  const dispatch = useTodoDispatch();
  const [text, setText] = useState('');
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    dispatch({ type: 'ADD_TODO', payload: text });
    setText('');
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">Add</button>
    </form>
  );
};
```
```

## ✅ ベストプラクティス

### 1. Context は必要なときだけ使う

```tsx
// ❌ 不要な Context
// 2-3階層のコンポーネントなら Props で十分
<UserContext.Provider>
  <Parent>
    <Child>
      <GrandChild />
    </Child>
  </Parent>
</UserContext.Provider>

// ✅ Props で渡す
<Parent user={user}>
  <Child user={user}>
    <GrandChild user={user} />
  </Child>
</Parent>
```

### 2. Context を小さく保つ

```tsx
// ❌ すべての状態を1つの Context に
<AppContext.Provider value={{ user, theme, language, settings, ... }}>

// ✅ 関心事ごとに分ける
<UserContext.Provider>
  <ThemeContext.Provider>
    <LanguageContext.Provider>
```

### 3. カスタムフックでラップ

```tsx
// ✅ カスタムフックで使いやすく
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

## 📝 練習問題

### 問題1: カート Context

ショッピングカートの Context を作成してください。
商品の追加・削除・数量変更ができるようにしてください。

<details>
<parameter name="summary">解答例</summary>

```tsx
type CartItem = {
  id: number;
  name: string;
  price: number;
  quantity: number;
};

type CartContextType = {
  items: CartItem[];
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: number) => void;
  updateQuantity: (id: number, quantity: number) => void;
  total: number;
};

const CartContext = createContext<CartContextType | null>(null);

export function useCart() {
  const context = useContext(CartContext);
  if (!context) throw new Error('useCart must be used within CartProvider');
  return context;
}

export function CartProvider({ children }: { children: React.ReactNode }) {
  const [items, setItems] = useState<CartItem[]>([]);
  
  const addItem = (newItem: Omit<CartItem, 'quantity'>) => {
    setItems(prev => {
      const existing = prev.find(item => item.id === newItem.id);
      if (existing) {
        return prev.map(item =>
          item.id === newItem.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prev, { ...newItem, quantity: 1 }];
    });
  };
  
  const removeItem = (id: number) => {
    setItems(prev => prev.filter(item => item.id !== id));
  };
  
  const updateQuantity = (id: number, quantity: number) => {
    setItems(prev =>
      prev.map(item =>
        item.id === id ? { ...item, quantity } : item
      )
    );
  };
  
  const total = items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );
  
  return (
    <CartContext.Provider value={{ items, addItem, removeItem, updateQuantity, total }}>
      {children}
    </CartContext.Provider>
  );
}
```
</details>

## 🔗 次のステップ

次は [Chapter 9: エコシステム](../09-ecosystem/) で React Router や TanStack Query について学びます。

# カスタムフックの作成

## 📖 概要

カスタムフックは、ロジックを再利用可能な関数として切り出す仕組みです。
複数のコンポーネントで同じロジックを使いたいときに便利です。

## 🎯 学習目標

- カスタムフックを作成できる
- ロジックを再利用できる
- カスタムフックのルールを理解する

## 1. カスタムフックとは

### 基本ルール

1. **名前は `use` で始める**
2. **他のフックを呼び出せる**
3. **通常の関数として扱える**

```tsx
// ✅ カスタムフック
const useCounter = () => {
  const [count, setCount] = useState(0);
  
  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(0);
  
  return { count, increment, decrement, reset };
};

// ❌ use で始まらない
const counter = () => {
  const [count, setCount] = useState(0);
  // ...
};
```

## 2. シンプルなカスタムフック

### useToggle - トグル状態の管理

```tsx
const useToggle = (initialValue: boolean = false) => {
  const [value, setValue] = useState(initialValue);
  
  const toggle = () => setValue(v => !v);
  const setTrue = () => setValue(true);
  const setFalse = () => setValue(false);
  
  return { value, toggle, setTrue, setFalse };
};

// 使用例
const Component = () => {
  const { value: isOpen, toggle, setFalse } = useToggle();
  
  return (
    <div>
      {isOpen && <Modal onClose={setFalse} />}
      <button onClick={toggle}>Toggle Modal</button>
    </div>
  );
};
```

### useCounter - カウンター

```tsx
type UseCounterOptions = {
  min?: number;
  max?: number;
  step?: number;
};

const useCounter = (initialValue: number = 0, options: UseCounterOptions = {}) => {
  const { min = -Infinity, max = Infinity, step = 1 } = options;
  const [count, setCount] = useState(initialValue);
  
  const increment = () => {
    setCount(c => Math.min(c + step, max));
  };
  
  const decrement = () => {
    setCount(c => Math.max(c - step, min));
  };
  
  const reset = () => {
    setCount(initialValue);
  };
  
  const set = (value: number) => {
    setCount(Math.max(min, Math.min(value, max)));
  };
  
  return { count, increment, decrement, reset, set };
};

// 使用例
const Component = () => {
  const { count, increment, decrement, reset } = useCounter(0, {
    min: 0,
    max: 10
  });
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};
```

## 3. データ取得のカスタムフック

### useFetch - 基本形

```tsx
type UseFetchResult<T> = {
  data: T | null;
  loading: boolean;
  error: Error | null;
};

const useFetch = <T,>(url: string): UseFetchResult<T> => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const json = await response.json();
        setData(json);
      } catch (e) {
        setError(e as Error);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return { data, loading, error };
};

// 使用例
type User = {
  id: number;
  name: string;
  email: string;
};

const UserProfile = ({ userId }: { userId: number }) => {
  const { data: user, loading, error } = useFetch<User>(`/api/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return <div>No user found</div>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};
```

### useApi - より実用的

```tsx
type UseApiOptions = {
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
  body?: any;
  headers?: Record<string, string>;
};

type UseApiResult<T> = {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
};

const useApi = <T,>(url: string, options: UseApiOptions = {}): UseApiResult<T> => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const [refetchTrigger, setRefetchTrigger] = useState(0);
  
  const refetch = useCallback(() => {
    setRefetchTrigger(prev => prev + 1);
  }, []);
  
  useEffect(() => {
    const controller = new AbortController();
    
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(url, {
          method: options.method || 'GET',
          headers: {
            'Content-Type': 'application/json',
            ...options.headers
          },
          body: options.body ? JSON.stringify(options.body) : undefined,
          signal: controller.signal
        });
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const json = await response.json();
        setData(json);
      } catch (e) {
        if (e.name !== 'AbortError') {
          setError(e as Error);
        }
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
    
    return () => {
      controller.abort();
    };
  }, [url, options.method, options.body, refetchTrigger]);
  
  return { data, loading, error, refetch };
};
```

## 4. フォーム管理のカスタムフック

### useForm

```tsx
type UseFormReturn<T> = {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  handleChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  handleSubmit: (e: React.FormEvent) => void;
  reset: () => void;
};

const useForm = <T extends Record<string, any>>(
  initialValues: T,
  onSubmit: (values: T) => void,
  validate?: (values: T) => Partial<Record<keyof T, string>>
): UseFormReturn<T> => {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      
      if (Object.keys(validationErrors).length === 0) {
        onSubmit(values);
      }
    } else {
      onSubmit(values);
    }
  };
  
  const reset = () => {
    setValues(initialValues);
    setErrors({});
  };
  
  return { values, errors, handleChange, handleSubmit, reset };
};

// 使用例
type LoginFormData = {
  email: string;
  password: string;
};

const LoginForm = () => {
  const initialValues: LoginFormData = {
    email: '',
    password: ''
  };
  
  const validate = (values: LoginFormData) => {
    const errors: Partial<Record<keyof LoginFormData, string>> = {};
    
    if (!values.email) {
      errors.email = 'メールアドレスは必須です';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      errors.email = 'メールアドレスの形式が正しくありません';
    }
    
    if (!values.password) {
      errors.password = 'パスワードは必須です';
    } else if (values.password.length < 8) {
      errors.password = 'パスワードは8文字以上必要です';
    }
    
    return errors;
  };
  
  const handleSubmit = (values: LoginFormData) => {
    console.log('Submit:', values);
  };
  
  const { values, errors, handleChange, handleSubmit: onSubmit } = useForm(
    initialValues,
    handleSubmit,
    validate
  );
  
  return (
    <form onSubmit={onSubmit}>
      <div>
        <input
          name="email"
          type="email"
          value={values.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {errors.email && <span>{errors.email}</span>}
      </div>
      <div>
        <input
          name="password"
          type="password"
          value={values.password}
          onChange={handleChange}
          placeholder="Password"
        />
        {errors.password && <span>{errors.password}</span>}
      </div>
      <button type="submit">Login</button>
    </form>
  );
};
```

## 5. useLocalStorage - ローカルストレージ

```tsx
const useLocalStorage = <T,>(key: string, initialValue: T) => {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue] as const;
};

// 使用例
const ThemeToggle = () => {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
  
  const toggle = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };
  
  return (
    <button onClick={toggle}>
      Current theme: {theme}
    </button>
  );
};
```

## 6. useDebounce - デバウンス

```tsx
const useDebounce = <T,>(value: T, delay: number): T => {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
};

// 使用例
const SearchBox = () => {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);
  
  useEffect(() => {
    if (debouncedQuery) {
      // デバウンスされた値で検索
      console.log('Searching for:', debouncedQuery);
    }
  }, [debouncedQuery]);
  
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
};
```

## 7. useMediaQuery - レスポンシブ

```tsx
const useMediaQuery = (query: string): boolean => {
  const [matches, setMatches] = useState(() => {
    if (typeof window !== 'undefined') {
      return window.matchMedia(query).matches;
    }
    return false;
  });
  
  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    
    const handleChange = (e: MediaQueryListEvent) => {
      setMatches(e.matches);
    };
    
    mediaQuery.addEventListener('change', handleChange);
    
    return () => {
      mediaQuery.removeEventListener('change', handleChange);
    };
  }, [query]);
  
  return matches;
};

// 使用例
const ResponsiveComponent = () => {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(min-width: 769px) and (max-width: 1024px)');
  const isDesktop = useMediaQuery('(min-width: 1025px)');
  
  return (
    <div>
      {isMobile && <div>Mobile View</div>}
      {isTablet && <div>Tablet View</div>}
      {isDesktop && <div>Desktop View</div>}
    </div>
  );
};
```

## ✅ ベストプラクティス

### 1. use で始める

```tsx
// ✅ カスタムフック
const useCounter = () => {};
const useForm = () => {};

// ❌ use で始まらない
const counter = () => {};
const formHelper = () => {};
```

### 2. 1つの責務を持つ

```tsx
// ✅ 単一の責務
const useCounter = () => {};
const useFetch = () => {};

// ❌ 複数の責務
const useEverything = () => {
  // カウンター、フォーム、データ取得...
};
```

### 3. TypeScript で型を明示

```tsx
// ✅ ジェネリクスで型安全に
const useFetch = <T,>(url: string): UseFetchResult<T> => {};

// ✅ 返り値の型を明示
type UseToggleReturn = {
  value: boolean;
  toggle: () => void;
};
```

## 📝 練習問題

### 問題1: useWindowSize

ウィンドウのサイズを取得するカスタムフックを作成してください。

<details>
<summary>解答</summary>

```tsx
const useWindowSize = () => {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  
  return size;
};
```
</details>

## 🔗 次のステップ

次は [Chapter 8: アーキテクチャ](../08-architecture/) でアプリケーションの設計について学びます。

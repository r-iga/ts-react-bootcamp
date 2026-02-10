# Chapter 7: カスタムフック

## 🎯 学習目標

- カスタムフックの概念と利点を理解する
- ロジックの再利用方法を習得する
- 実践的なカスタムフックを作成できるようになる
- カスタムフックの設計パターンを学ぶ

## 📚 学習トピック

### 7.1 カスタムフックとは

**カスタムフックの定義:**
- `use` で始まる関数
- 他の Hooks を内部で使用
- ステートフルなロジックの再利用

**カスタムフックのメリット:**
- ロジックの抽象化と再利用
- コンポーネントのシンプル化
- テストしやすいコードの実現
- 関心の分離

**命名規則:**
- 必ず `use` で始める（例: `useCounter`, `useFetch`）
- フックのルールが適用される
- 意味のある名前をつける

### 7.2 基本的なカスタムフック

**シンプルなカスタムフック:**
```tsx
import { useState } from 'react';

function useCounter(initialValue: number = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  const reset = () => setCount(initialValue);
  
  return { count, increment, decrement, reset };
}

// 使用例
function Counter() {
  const { count, increment, decrement, reset } = useCounter(0);
  
  return (
    <>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </>
  );
}
```

**トグル機能:**
```tsx
function useToggle(initialValue: boolean = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = () => setValue(prev => !prev);
  const setTrue = () => setValue(true);
  const setFalse = () => setValue(false);
  
  return { value, toggle, setTrue, setFalse };
}
```

**配列の操作:**
```tsx
function useArray<T>(initialArray: T[] = []) {
  const [array, setArray] = useState(initialArray);
  
  const push = (item: T) => setArray(prev => [...prev, item]);
  const remove = (index: number) => 
    setArray(prev => prev.filter((_, i) => i !== index));
  const clear = () => setArray([]);
  
  return { array, push, remove, clear };
}
```

### 7.3 副作用を含むカスタムフック

**useLocalStorage:**
```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  // 初期値の取得
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // 値の更新
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = 
        value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue] as const;
}

// 使用例
function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle Theme (current: {theme})
    </button>
  );
}
```

**useWindowSize:**
```tsx
function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });
  
  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return windowSize;
}
```

**useDebounce:**
```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// 使用例: 検索入力のデバウンス
function SearchInput() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // API 呼び出し
      console.log('Searching for:', debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
    />
  );
}
```

### 7.4 データフェッチング

**useFetch:**
```tsx
type FetchState<T> = {
  data: T | null;
  loading: boolean;
  error: Error | null;
};

function useFetch<T>(url: string) {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null,
  });
  
  useEffect(() => {
    let cancelled = false;
    
    const fetchData = async () => {
      setState({ data: null, loading: true, error: null });
      
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error('Fetch failed');
        const data = await response.json();
        
        if (!cancelled) {
          setState({ data, loading: false, error: null });
        }
      } catch (error) {
        if (!cancelled) {
          setState({ 
            data: null, 
            loading: false, 
            error: error as Error 
          });
        }
      }
    };
    
    fetchData();
    
    return () => {
      cancelled = true;
    };
  }, [url]);
  
  return state;
}

// 使用例
function UserProfile({ userId }: { userId: string }) {
  const { data, loading, error } = useFetch<User>(
    `https://api.example.com/users/${userId}`
  );
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return null;
  
  return <div>{data.name}</div>;
}
```

**useAsync:**
```tsx
function useAsync<T>(
  asyncFunction: () => Promise<T>,
  immediate: boolean = true
) {
  const [status, setStatus] = useState<'idle' | 'pending' | 'success' | 'error'>('idle');
  const [value, setValue] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  
  const execute = useCallback(() => {
    setStatus('pending');
    setValue(null);
    setError(null);
    
    return asyncFunction()
      .then((response) => {
        setValue(response);
        setStatus('success');
      })
      .catch((error) => {
        setError(error);
        setStatus('error');
      });
  }, [asyncFunction]);
  
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);
  
  return { execute, status, value, error };
}
```

### 7.5 フォーム管理

**useForm:**
```tsx
function useForm<T extends Record<string, any>>(initialValues: T) {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };
  
  const setFieldValue = (name: keyof T, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
  };
  
  const setFieldError = (name: keyof T, error: string) => {
    setErrors(prev => ({ ...prev, [name]: error }));
  };
  
  const reset = () => {
    setValues(initialValues);
    setErrors({});
  };
  
  return {
    values,
    errors,
    handleChange,
    setFieldValue,
    setFieldError,
    reset,
  };
}

// 使用例
function LoginForm() {
  const { values, errors, handleChange, setFieldError, reset } = useForm({
    email: '',
    password: '',
  });
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!values.email) {
      setFieldError('email', 'Email is required');
      return;
    }
    
    // Submit logic
    console.log('Submitted:', values);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        value={values.email}
        onChange={handleChange}
      />
      {errors.email && <span>{errors.email}</span>}
      
      <input
        name="password"
        type="password"
        value={values.password}
        onChange={handleChange}
      />
      {errors.password && <span>{errors.password}</span>}
      
      <button type="submit">Login</button>
      <button type="button" onClick={reset}>Reset</button>
    </form>
  );
}
```

### 7.6 DOM 操作

**useClickOutside:**
```tsx
function useClickOutside(
  ref: React.RefObject<HTMLElement>,
  handler: () => void
) {
  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      if (!ref.current || ref.current.contains(event.target as Node)) {
        return;
      }
      handler();
    };
    
    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);
    
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}

// 使用例: ドロップダウンメニュー
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);
  
  useClickOutside(dropdownRef, () => setIsOpen(false));
  
  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Menu</button>
      {isOpen && <div>Dropdown content</div>}
    </div>
  );
}
```

**useEventListener:**
```tsx
function useEventListener<K extends keyof WindowEventMap>(
  eventName: K,
  handler: (event: WindowEventMap[K]) => void,
  element: Window | HTMLElement = window
) {
  const savedHandler = useRef(handler);
  
  useEffect(() => {
    savedHandler.current = handler;
  }, [handler]);
  
  useEffect(() => {
    const eventListener = (event: Event) => 
      savedHandler.current(event as WindowEventMap[K]);
    
    element.addEventListener(eventName, eventListener);
    
    return () => {
      element.removeEventListener(eventName, eventListener);
    };
  }, [eventName, element]);
}
```

### 7.7 カスタムフックの設計パターン

**返り値のパターン:**
```tsx
// 配列で返す（useState スタイル）
function useToggle(initial: boolean) {
  const [value, setValue] = useState(initial);
  const toggle = () => setValue(v => !v);
  return [value, toggle] as const;
}

// オブジェクトで返す
function useCounter(initial: number) {
  const [count, setCount] = useState(initial);
  const increment = () => setCount(c => c + 1);
  return { count, increment };
}
```

**引数のパターン:**
```tsx
// 単一の引数
function useFetch(url: string) { /* ... */ }

// オプションオブジェクト
function useFetch(options: {
  url: string;
  method?: string;
  headers?: Record<string, string>;
}) { /* ... */ }
```

**フックの組み合わせ:**
```tsx
function useUserData(userId: string) {
  const { data, loading, error } = useFetch(`/users/${userId}`);
  const [user, setUser] = useLocalStorage('user', null);
  
  useEffect(() => {
    if (data) {
      setUser(data);
    }
  }, [data, setUser]);
  
  return { user, loading, error };
}
```

**TypeScript のベストプラクティス:**
- ジェネリック型の活用
- 明示的な返り値の型
- `as const` での型推論の改善

## 🔗 関連リソース

- [examples/07-custom-hooks/](../../examples/07-custom-hooks/) - サンプルコード
- [exercises/07-custom-hooks/](../../exercises/07-custom-hooks/) - 演習問題

## ⏱️ 推奨学習時間

**1-2週間（15-20時間）**

## ✅ チェックリスト

- [ ] カスタムフックの概念を理解している
- [ ] 基本的なカスタムフックを作成できる
- [ ] 副作用を含むカスタムフックを作成できる
- [ ] データフェッチング用のフックを実装できる
- [ ] フォーム管理用のフックを実装できる
- [ ] DOM 操作用のフックを実装できる
- [ ] カスタムフックの適切な設計ができる
- [ ] TypeScript でカスタムフックを型安全に実装できる

## 📝 次の章

[Chapter 8: アーキテクチャ](../08-architecture/)

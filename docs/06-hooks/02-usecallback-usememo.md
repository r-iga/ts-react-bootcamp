# useCallback と useMemo

## 📖 概要

useCallback と useMemo はパフォーマンス最適化のためのフックです。
メモ化により、不要な再計算や再レンダリングを防ぎます。

## 🎯 学習目標

- useCallback でコールバックをメモ化できる
- useMemo で計算結果をメモ化できる
- 適切な使い分けができる

## 1. useCallback の基礎

### 問題：毎回新しい関数が作られる

```tsx
import { memo } from 'react';

const Child = memo(function Child({ onClick }: { onClick: () => void }) {
  console.log("Child rendered");
  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ 毎回新しい関数 → Child が再レンダリング
  const handleClick = () => {
    console.log("Clicked");
  };
  
  return (
    <div>
      <Child onClick={handleClick} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
}
```

### useCallback で解決

```tsx
import { memo, useCallback } from 'react';

const Child = memo(function Child({ onClick }: { onClick: () => void }) {
  console.log("Child rendered");
  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  // ✅ 同じ関数参照を返す
  const handleClick = useCallback(() => {
    console.log("Clicked");
  }, []);
  
  return (
    <div>
      <Child onClick={handleClick} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
}
```

## 2. useCallback の構文

### 基本形

```tsx
const memoizedCallback = useCallback(
  () => {
    // コールバック関数
  },
  [/* 依存配列 */]
);
```

### 依存配列

```tsx
function Component() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");
  
  // count を使うので依存配列に含める
  const handleClick = useCallback(() => {
    console.log("Count is:", count);
  }, [count]);
  
  return (
    <div>
      <button onClick={handleClick}>Log count</button>
      <button onClick={() => setCount(count + 1)}>+</button>
      <input value={text} onChange={(e) => setText(e.target.value)} />
    </div>
  );
}
```

## 3. useCallback の実用例

### Form のハンドラー

```tsx
type FormData = {
  name: string;
  email: string;
};

function Form() {
  const [formData, setFormData] = useState<FormData>({
    name: "",
    email: ""
  });
  
  // ✅ 各フィールドのハンドラーをメモ化
  const handleNameChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    setFormData(prev => ({ ...prev, name: e.target.value }));
  }, []);
  
  const handleEmailChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    setFormData(prev => ({ ...prev, email: e.target.value }));
  }, []);
  
  const handleSubmit = useCallback((e: React.FormEvent) => {
    e.preventDefault();
    console.log("Submit:", formData);
  }, [formData]); // formData が変わったら再作成
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={formData.name} onChange={handleNameChange} />
      <input value={formData.email} onChange={handleEmailChange} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### リストアイテムのコールバック

```tsx
type Todo = {
  id: number;
  text: string;
};

const TodoItem = memo(function TodoItem({ 
  todo, 
  onDelete 
}: { 
  todo: Todo; 
  onDelete: (id: number) => void;
}) {
  console.log("TodoItem rendered:", todo.text);
  return (
    <div>
      {todo.text}
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </div>
  );
});

function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([
    { id: 1, text: "Learn React" },
    { id: 2, text: "Learn TypeScript" }
  ]);
  
  // ✅ メモ化しないと全TodoItemが再レンダリング
  const handleDelete = useCallback((id: number) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);
  
  return (
    <div>
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} onDelete={handleDelete} />
      ))}
    </div>
  );
}
```

## 4. useMemo の基礎

### 問題：毎回同じ計算をする

```tsx
function Component({ items }: { items: number[] }) {
  const [filter, setFilter] = useState("");
  
  // ❌ filter が変わらなくても毎回計算
  const total = items.reduce((sum, item) => sum + item, 0);
  
  return <div>Total: {total}</div>;
}
```

### useMemo で解決

```tsx
function Component({ items }: { items: number[] }) {
  const [filter, setFilter] = useState("");
  
  // ✅ items が変わったときのみ計算
  const total = useMemo(() => {
    console.log("Calculating total...");
    return items.reduce((sum, item) => sum + item, 0);
  }, [items]);
  
  return (
    <div>
      <input value={filter} onChange={(e) => setFilter(e.target.value)} />
      <p>Total: {total}</p>
    </div>
  );
}
```

## 5. useMemo の構文

### 基本形

```tsx
const memoizedValue = useMemo(
  () => {
    // 計算処理
    return result;
  },
  [/* 依存配列 */]
);
```

### 重い計算のメモ化

```tsx
function ExpensiveComponent({ data }: { data: number[] }) {
  const [query, setQuery] = useState("");
  
  // ✅ 重い計算をメモ化
  const processedData = useMemo(() => {
    console.log("Processing data...");
    // 時間がかかる処理
    return data
      .map(item => item * 2)
      .filter(item => item > 100)
      .sort((a, b) => a - b);
  }, [data]);
  
  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ul>
        {processedData.map((item, i) => (
          <li key={i}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

## 6. useMemo の実用例

### フィルタリングとソート

```tsx
type User = {
  id: number;
  name: string;
  age: number;
};

function UserList({ users }: { users: User[] }) {
  const [searchQuery, setSearchQuery] = useState("");
  const [sortBy, setSortBy] = useState<"name" | "age">("name");
  
  // ✅ フィルタとソートをメモ化
  const filteredAndSortedUsers = useMemo(() => {
    console.log("Filtering and sorting...");
    
    const filtered = users.filter(user =>
      user.name.toLowerCase().includes(searchQuery.toLowerCase())
    );
    
    return filtered.sort((a, b) => {
      if (sortBy === "name") {
        return a.name.localeCompare(b.name);
      }
      return a.age - b.age;
    });
  }, [users, searchQuery, sortBy]);
  
  return (
    <div>
      <input
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        placeholder="Search..."
      />
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value as "name" | "age")}>
        <option value="name">Sort by Name</option>
        <option value="age">Sort by Age</option>
      </select>
      <ul>
        {filteredAndSortedUsers.map(user => (
          <li key={user.id}>{user.name} ({user.age})</li>
        ))}
      </ul>
    </div>
  );
}
```

### オブジェクトの参照を安定化

```tsx
function Component() {
  const [count, setCount] = useState(0);
  
  // ❌ 毎回新しいオブジェクト
  const options = { limit: 10, offset: 0 };
  
  // ✅ 同じ参照を返す
  const options = useMemo(() => ({
    limit: 10,
    offset: 0
  }), []);
  
  useEffect(() => {
    fetchData(options);
  }, [options]); // options が変わらないので初回のみ実行
  
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

## 7. useCallback vs useMemo

### 違い

```tsx
// useCallback：関数をメモ化
const memoizedCallback = useCallback(
  () => {
    doSomething();
  },
  [dependency]
);

// useMemo：計算結果をメモ化
const memoizedValue = useMemo(
  () => {
    return computeExpensiveValue();
  },
  [dependency]
);

// ✨ useCallback は useMemo の糖衣構文
useCallback(fn, deps) === useMemo(() => fn, deps)
```

### 使い分け

```tsx
// ✅ useCallback：関数を渡す
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);

<Child onClick={handleClick} />

// ✅ useMemo：値を計算
const total = useMemo(() => {
  return items.reduce((sum, item) => sum + item, 0);
}, [items]);

<div>Total: {total}</div>
```

## 8. いつ使うべきか

### ✅ 使うべきケース

```tsx
// 1. memo化されたコンポーネントに渡すコールバック
const Child = memo(ChildComponent);
const callback = useCallback(() => {}, []);
<Child onClick={callback} />

// 2. useEffect の依存配列に含まれる関数・オブジェクト
const options = useMemo(() => ({ limit: 10 }), []);
useEffect(() => {
  fetchData(options);
}, [options]);

// 3. 重い計算
const result = useMemo(() => heavyCalculation(data), [data]);
```

### ❌ 使わなくていいケース

```tsx
// 1. シンプルな計算
const doubled = useMemo(() => count * 2, [count]);
// ✅ 直接計算で十分
const doubled = count * 2;

// 2. すべてのコールバック
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);
// ⚠️ パフォーマンス問題がなければ不要
const handleClick = () => {
  console.log("clicked");
};
```

## ✅ ベストプラクティス

### 1. 早まった最適化はしない

```tsx
// ❌ すべてを useCallback/useMemo でラップ
// ✅ パフォーマンス問題が出てから最適化
```

### 2. 依存配列は正直に

```tsx
// ✅ 使用している値をすべて含める
const callback = useCallback(() => {
  doSomething(count, name);
}, [count, name]);
```

### 3. React DevTools で確認

```tsx
// Profiler を使って実際の効果を測定
// 体感できる改善がない場合は不要
```

## 📝 練習問題

### 問題1: リストのフィルタリング

商品リストを検索クエリでフィルタリングするコンポーネントを作成してください。
useMemo を使ってフィルタリング処理をメモ化してください。

<details>
<summary>解答</summary>

```tsx
type Product = {
  id: number;
  name: string;
  price: number;
};

function ProductList({ products }: { products: Product[] }) {
  const [query, setQuery] = useState("");
  
  const filteredProducts = useMemo(() => {
    return products.filter(product =>
      product.name.toLowerCase().includes(query.toLowerCase())
    );
  }, [products, query]);
  
  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search products..."
      />
      <ul>
        {filteredProducts.map(product => (
          <li key={product.id}>
            {product.name} - ¥{product.price}
          </li>
        ))}
      </ul>
    </div>
  );
}
```
</details>

## 🔗 次のステップ

次は [useRef とDOM操作](./03-useref.md) について学びます。

# useState の基礎

## 📖 概要

useState は React で最も重要なフックです。
コンポーネントに状態（State）を持たせ、変更に応じて自動的にUIを更新できます。

## 🎯 学習目標

- useState の使い方を理解する
- State の更新方法を習得する
- 不変性の重要性を理解する

## 1. useState の基本

### シンプルなカウンター

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

### 構文の説明

```tsx
const [state, setState] = useState(initialValue);
//     ↑      ↑            ↑           ↑
//   現在値  更新関数     フック      初期値
```

## 2. 様々な型の State

### 文字列

```tsx
function NameInput() {
  const [name, setName] = useState("");
  
  return (
    <input
      type="text"
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

### 真偽値

```tsx
function Toggle() {
  const [isOn, setIsOn] = useState(false);
  
  return (
    <button onClick={() => setIsOn(!isOn)}>
      {isOn ? "ON" : "OFF"}
    </button>
  );
}
```

### オブジェクト

```tsx
type User = {
  name: string;
  age: number;
};

function UserForm() {
  const [user, setUser] = useState<User>({
    name: "",
    age: 0
  });
  
  return (
    <div>
      <input
        value={user.name}
        onChange={(e) => setUser({ ...user, name: e.target.value })}
      />
      <input
        type="number"
        value={user.age}
        onChange={(e) => setUser({ ...user, age: Number(e.target.value) })}
      />
    </div>
  );
}
```

### 配列

```tsx
function TodoList() {
  const [todos, setTodos] = useState<string[]>([]);
  
  const addTodo = (text: string) => {
    setTodos([...todos, text]);
  };
  
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>{todo}</li>
      ))}
    </ul>
  );
}
```

## 3. State の更新

### 直接値を渡す

```tsx
const [count, setCount] = useState(0);

// ✅ 新しい値を設定
setCount(10);
setCount(count + 1);
```

### 関数を渡す（更新関数）

```tsx
const [count, setCount] = useState(0);

// ✅ 現在値を元に更新
setCount(prev => prev + 1);
setCount(prev => prev * 2);
```

### 更新関数を使うべきとき

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    // ❌ 連続で呼んでも1回しか増えない
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // count は 1 になる
  };
  
  const incrementCorrect = () => {
    // ✅ 正しく3回増える
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    // count は 3 になる
  };
  
  return <button onClick={incrementCorrect}>+3</button>;
}
```

## 4. オブジェクトの更新（不変性）

### スプレッド構文で更新

```tsx
type User = {
  name: string;
  age: number;
  email: string;
};

function UserProfile() {
  const [user, setUser] = useState<User>({
    name: "Alice",
    age: 25,
    email: "alice@example.com"
  });
  
  // ✅ スプレッド構文で一部を更新
  const updateName = (name: string) => {
    setUser({ ...user, name });
  };
  
  const updateAge = (age: number) => {
    setUser({ ...user, age });
  };
  
  return (
    <div>
      <input value={user.name} onChange={(e) => updateName(e.target.value)} />
      <input type="number" value={user.age} onChange={(e) => updateAge(Number(e.target.value))} />
    </div>
  );
}
```

### ネストしたオブジェクトの更新

```tsx
type User = {
  name: string;
  address: {
    city: string;
    zipCode: string;
  };
};

function UserProfile() {
  const [user, setUser] = useState<User>({
    name: "Alice",
    address: {
      city: "Tokyo",
      zipCode: "100-0001"
    }
  });
  
  // ✅ ネストしたプロパティの更新
  const updateCity = (city: string) => {
    setUser({
      ...user,
      address: {
        ...user.address,
        city
      }
    });
  };
  
  return <input value={user.address.city} onChange={(e) => updateCity(e.target.value)} />;
}
```

## 5. 配列の更新（不変性）

### 要素の追加

```tsx
const [items, setItems] = useState<string[]>([]);

// ✅ 末尾に追加
setItems([...items, "new item"]);
setItems(prev => [...prev, "new item"]);

// ✅ 先頭に追加
setItems(["new item", ...items]);
```

### 要素の削除

```tsx
const [items, setItems] = useState(["Apple", "Banana", "Orange"]);

// ✅ インデックスで削除
const removeAt = (index: number) => {
  setItems(items.filter((_, i) => i !== index));
};

// ✅ 値で削除
const remove = (value: string) => {
  setItems(items.filter(item => item !== value));
};
```

### 要素の更新

```tsx
const [items, setItems] = useState(["Apple", "Banana", "Orange"]);

// ✅ インデックスの要素を更新
const updateAt = (index: number, newValue: string) => {
  setItems(items.map((item, i) => i === index ? newValue : item));
};
```

## 6. 複雑な State の例

### Todoリスト

```tsx
type Todo = {
  id: number;
  text: string;
  completed: boolean;
};

function TodoApp() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  const addTodo = (text: string) => {
    const newTodo: Todo = {
      id: Date.now(),
      text,
      completed: false
    };
    setTodos([...todos, newTodo]);
  };
  
  const toggleTodo = (id: number) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const deleteTodo = (id: number) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  return (
    <div>
      {todos.map(todo => (
        <div key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          <span>{todo.text}</span>
          <button onClick={() => deleteTodo(todo.id)}>Delete</button>
        </div>
      ))}
    </div>
  );
}
```

## 7. TypeScriptでの型指定

### 型推論

```tsx
// ✅ 型推論が効く
const [count, setCount] = useState(0); // number
const [name, setName] = useState(""); // string
const [isOpen, setIsOpen] = useState(false); // boolean
```

### 明示的な型指定

```tsx
// ✅ ジェネリクスで型を指定
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<string[]>([]);

type Status = "idle" | "loading" | "success" | "error";
const [status, setStatus] = useState<Status>("idle");
```

## ✅ ベストプラクティス

### 1. 不変性を守る

```tsx
// ❌ 直接変更しない
const [items, setItems] = useState([1, 2, 3]);
items.push(4); // NG
setItems(items);

// ✅ 新しい配列を作る
setItems([...items, 4]);
```

### 2. 更新関数を使う

```tsx
// ⚠️ 古い値を参照している可能性
setCount(count + 1);

// ✅ 最新の値を使う
setCount(prev => prev + 1);
```

### 3. 複数のStateに分割

```tsx
// ⚠️ 関連性のない値を1つのオブジェクトに
const [state, setState] = useState({
  name: "",
  age: 0,
  isLoggedIn: false,
  cartItems: []
});

// ✅ 個別のStateに分割
const [name, setName] = useState("");
const [age, setAge] = useState(0);
const [isLoggedIn, setIsLoggedIn] = useState(false);
const [cartItems, setCartItems] = useState([]);
```

### 4. 初期値は変数から取得しない

```tsx
// ⚠️ propsやconstantは初回のみ評価される
const [count, setCount] = useState(props.initialCount);

// 初期値が変わっても count は更新されない
```

## 📝 練習問題

### 問題1: カウンター

増減ボタンとリセットボタンを持つカウンターを作成してください。

<details>
<summary>解答</summary>

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(prev => prev + 1)}>+</button>
      <button onClick={() => setCount(prev => prev - 1)}>-</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```
</details>

### 問題2: 入力フォーム

名前とメールアドレスを入力するフォームを作成してください。

<details>
<summary>解答</summary>

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
  
  return (
    <div>
      <input
        value={formData.name}
        onChange={(e) => setFormData({ ...formData, name: e.target.value })}
        placeholder="Name"
      />
      <input
        value={formData.email}
        onChange={(e) => setFormData({ ...formData, email: e.target.value })}
        placeholder="Email"
      />
    </div>
  );
}
```
</details>

## 🔗 次のステップ

次は [再レンダリングの仕組み](./02-rerendering.md) について学びます。

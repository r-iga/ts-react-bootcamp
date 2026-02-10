# Props とイベント処理

## 📖 概要

Reactコンポーネント間でデータを渡し、ユーザーイベントに応答する方法を学びます。
Propsは親から子へのデータの流れ、イベントハンドラーは子から親への通信手段です。

## 🎯 学習目標

- Props でデータを渡せる
- イベントハンドラーを実装できる
- コールバックで親に通知できる

## 1. Props によるデータの受け渡し

### 親から子へ

```tsx
// 子コンポーネント
type UserProps = {
  name: string;
  age: number;
};

function User({ name, age }: UserProps) {
  return <div>{name} ({age})</div>;
}

// 親コンポーネント
function App() {
  return (
    <div>
      <User name="Alice" age={25} />
      <User name="Bob" age={30} />
    </div>
  );
}
```

### オブジェクトを渡す

```tsx
type User = {
  id: number;
  name: string;
  email: string;
};

type UserCardProps = {
  user: User;
};

function UserCard({ user }: UserCardProps) {
  return (
    <div>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}

// 使用
const user = { id: 1, name: "Alice", email: "alice@example.com" };
<UserCard user={user} />
```

## 2. イベントハンドリング

### onClick - クリックイベント

```tsx
function Button() {
  const handleClick = () => {
    console.log("Button clicked!");
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

### インライン関数

```tsx
function Button() {
  return (
    <button onClick={() => console.log("Clicked!")}>
      Click me
    </button>
  );
}
```

⚠️ **パフォーマンスの注意**

```tsx
// ⚠️ 毎回新しい関数が作られる
<button onClick={() => handleClick()}>Click</button>

// ✅ 関数を参照で渡す
<button onClick={handleClick}>Click</button>

// ✅ 引数が必要な場合
<button onClick={() => handleClick(id)}>Click</button>
```

### イベントオブジェクト

```tsx
function Button() {
  const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
    console.log("Clicked at:", event.clientX, event.clientY);
    console.log("Target:", event.currentTarget);
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

## 3. フォームのイベント

### onChange - 入力変更

```tsx
function Input() {
  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    console.log("Value:", event.target.value);
  };
  
  return <input type="text" onChange={handleChange} />;
}
```

### onSubmit - フォーム送信

```tsx
function Form() {
  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault(); // ページリロードを防ぐ
    console.log("Form submitted");
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="text" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### 完全なフォーム例

```tsx
function LoginForm() {
  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    
    const formData = new FormData(event.currentTarget);
    const email = formData.get("email") as string;
    const password = formData.get("password") as string;
    
    console.log({ email, password });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="email" name="email" required />
      <input type="password" name="password" required />
      <button type="submit">Login</button>
    </form>
  );
}
```

## 4. コールバック Props

### 子から親への通知

```tsx
// 子コンポーネント
type ButtonProps = {
  onPress: () => void;
};

function Button({ onPress }: ButtonProps) {
  return <button onClick={onPress}>Click me</button>;
}

// 親コンポーネント
function App() {
  const handlePress = () => {
    console.log("Button was pressed!");
  };
  
  return <Button onPress={handlePress} />;
}
```

### データを渡すコールバック

```tsx
// 子コンポーネント
type SearchBoxProps = {
  onSearch: (query: string) => void;
};

function SearchBox({ onSearch }: SearchBoxProps) {
  const handleSubmit = (event: React.FormEvent) => {
    event.preventDefault();
    const formData = new FormData(event.target as HTMLFormElement);
    const query = formData.get("query") as string;
    onSearch(query);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="text" name="query" />
      <button type="submit">Search</button>
    </form>
  );
}

// 親コンポーネント
function App() {
  const handleSearch = (query: string) => {
    console.log("Searching for:", query);
  };
  
  return <SearchBox onSearch={handleSearch} />;
}
```

## 5. よくあるイベント

### マウスイベント

```tsx
function Box() {
  return (
    <div
      onClick={() => console.log("Clicked")}
      onMouseEnter={() => console.log("Mouse entered")}
      onMouseLeave={() => console.log("Mouse left")}
      onDoubleClick={() => console.log("Double clicked")}
    >
      Hover or click me
    </div>
  );
}
```

### キーボードイベント

```tsx
function Input() {
  const handleKeyDown = (event: React.KeyboardEvent<HTMLInputElement>) => {
    if (event.key === "Enter") {
      console.log("Enter pressed");
    }
    if (event.key === "Escape") {
      console.log("Escape pressed");
    }
  };
  
  return <input type="text" onKeyDown={handleKeyDown} />;
}
```

### フォーカスイベント

```tsx
function Input() {
  return (
    <input
      type="text"
      onFocus={() => console.log("Focused")}
      onBlur={() => console.log("Blurred")}
    />
  );
}
```

## 6. イベントの伝播

### イベントバブリング

```tsx
function Parent() {
  return (
    <div onClick={() => console.log("Parent clicked")}>
      <button onClick={() => console.log("Button clicked")}>
        Click me
      </button>
    </div>
  );
}

// ボタンをクリックすると：
// "Button clicked"
// "Parent clicked"
```

### 伝播を止める

```tsx
function Parent() {
  return (
    <div onClick={() => console.log("Parent clicked")}>
      <button onClick={(e) => {
        e.stopPropagation();
        console.log("Button clicked");
      }}>
        Click me
      </button>
    </div>
  );
}

// ボタンをクリックすると：
// "Button clicked" のみ
```

## 7. Props の命名規則

### イベントハンドラー

```tsx
// ✅ on で始める
type Props = {
  onClick: () => void;
  onChange: (value: string) => void;
  onSubmit: () => void;
};

// ⚠️ 一般的でない名前
type Props = {
  clicked: () => void;
  changed: (value: string) => void;
};
```

### 内部のハンドラー

```tsx
function Component({ onClick }: { onClick: () => void }) {
  // ✅ handle で始める
  const handleClick = () => {
    console.log("Processing...");
    onClick();
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

## 8. 実践例：TodoItem

```tsx
type Todo = {
  id: number;
  text: string;
  completed: boolean;
};

type TodoItemProps = {
  todo: Todo;
  onToggle: (id: number) => void;
  onDelete: (id: number) => void;
};

function TodoItem({ todo, onToggle, onDelete }: TodoItemProps) {
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span
        style={{
          textDecoration: todo.completed ? "line-through" : "none"
        }}
      >
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </div>
  );
}

// 使用
function TodoList() {
  const handleToggle = (id: number) => {
    console.log("Toggle todo:", id);
  };
  
  const handleDelete = (id: number) => {
    console.log("Delete todo:", id);
  };
  
  const todos: Todo[] = [
    { id: 1, text: "Learn React", completed: false }
  ];
  
  return (
    <div>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
          onDelete={handleDelete}
        />
      ))}
    </div>
  );
}
```

## ✅ ベストプラクティス

### 1. コールバックの型を明示

```tsx
// ✅ 型を明示
type Props = {
  onSearch: (query: string) => void;
  onSelect: (id: number) => void;
};
```

### 2. preventDefault を忘れずに

```tsx
// ✅ フォーム送信時
<form onSubmit={(e) => {
  e.preventDefault();
  handleSubmit();
}}>
```

### 3. 不要な再レンダリングを避ける

```tsx
// ⚠️ 毎回新しい関数
<button onClick={() => doSomething()}>Click</button>

// ✅ 関数を参照
<button onClick={doSomething}>Click</button>

// ✅ useCallback で最適化（後述）
const handleClick = useCallback(() => {
  doSomething();
}, []);
```

## 📝 練習問題

### 問題1: カウンターコンポーネント

増減ボタンを持つコンポーネントを作成してください。

```tsx
type CounterProps = {
  value: number;
  onIncrement: () => void;
  onDecrement: () => void;
};
```

<details>
<summary>解答</summary>

```tsx
function Counter({ value, onIncrement, onDecrement }: CounterProps) {
  return (
    <div>
      <button onClick={onDecrement}>-</button>
      <span>{value}</span>
      <button onClick={onIncrement}>+</button>
    </div>
  );
}
```
</details>

## 🔗 次のステップ

次は [Chapter 5: State と再レンダリング](../05-state-rendering/) で状態管理について学びます。

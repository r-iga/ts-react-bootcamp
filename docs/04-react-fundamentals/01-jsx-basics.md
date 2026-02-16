# JSX の基礎

## 📖 概要

JSX は JavaScript の拡張構文で、HTMLに似た記法でUIを記述できます。
Reactの中核となる機能です。

## 🎯 学習目標

- JSX の文法を理解する
- JSX のルールを守れる
- JavaScript 式を埋め込める

## 1. JSX とは

### HTMLに似た構文

```tsx
const element = <h1>Hello, World!</h1>;
```

### 実際は JavaScript

JSXはBabelによってJavaScriptに変換されます。

```tsx
// JSX
const element = <h1>Hello, World!</h1>;

// 変換後（内部的に）
const element = React.createElement('h1', null, 'Hello, World!');
```

## 2. JSX の基本文法

### 要素の作成

```tsx
const heading = <h1>Title</h1>;
const paragraph = <p>This is a paragraph.</p>;
const button = <button>Click me</button>;
```

### 属性の指定

```tsx
const image = <img src="photo.jpg" alt="My photo" />;
const link = <a href="https://example.com">Link</a>;
const button = <button className="primary" disabled>Submit</button>;
```

⚠️ **className と htmlFor**

```tsx
// ❌ class（JavaScriptの予約語）
<div class="container"></div>

// ✅ className
<div className="container"></div>

// ❌ for（JavaScriptの予約語）
<label for="name">Name</label>

// ✅ htmlFor
<label htmlFor="name">Name</label>
```

### 自己クロージングタグ

```tsx
// ✅ 必須：自己クロージングタグには / をつける
<img src="photo.jpg" />
<input type="text" />
<br />

// ❌ エラー
<img src="photo.jpg">
<input type="text">
```

## 3. JavaScript 式の埋め込み

### {} で囲む

```tsx
const name = "Alice";
const age = 25;

const element = (
  <div>
    <h1>Hello, {name}!</h1>
    <p>You are {age} years old.</p>
    <p>Next year you will be {age + 1}.</p>
  </div>
);
```

### 式ならなんでもOK

```tsx
const user = { name: "Alice", age: 25 };

const element = (
  <div>
    <h1>{user.name}</h1>
    <p>{user.age * 2}</p>
    <p>{user.name.toUpperCase()}</p>
    <p>{Math.random()}</p>
  </div>
);
```

### 関数呼び出し

```tsx
const formatDate = (date: Date) => {
  return date.toLocaleDateString();
};

const element = <p>Today is {formatDate(new Date())}</p>;
```

## 4. 条件付きレンダリング

### 三項演算子

```tsx
const isLoggedIn = true;

const element = (
  <div>
    {isLoggedIn ? (
      <p>Welcome back!</p>
    ) : (
      <p>Please log in.</p>
    )}
  </div>
);
```

### && 演算子

```tsx
const hasError = true;

const element = (
  <div>
    {hasError && <p style={{ color: 'red' }}>Error occurred!</p>}
  </div>
);
```

⚠️ **0 の扱いに注意**

```tsx
const count = 0;

// ❌ 0 が表示される
<div>{count && <p>Count: {count}</p>}</div>

// ✅ 明示的な比較
<div>{count > 0 && <p>Count: {count}</p>}</div>

// ✅ または Boolean() で変換
<div>{Boolean(count) && <p>Count: {count}</p>}</div>
```

### if文（JSXの外）

```tsx
const Greeting = ({ isLoggedIn }: { isLoggedIn: boolean }) => {
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  }
  
  return <h1>Please log in.</h1>;
};
```

## 5. リストのレンダリング

### map() を使う

```tsx
const fruits = ["Apple", "Banana", "Orange"];

const element = (
  <ul>
    {fruits.map(fruit => (
      <li>{fruit}</li>
    ))}
  </ul>
);
```

### key プロパティ（必須）

```tsx
const fruits = ["Apple", "Banana", "Orange"];

const element = (
  <ul>
    {fruits.map((fruit, index) => (
      <li key={index}>{fruit}</li>
    ))}
  </ul>
);
```

⚠️ **key はユニークで安定した値を使う**

```tsx
// ❌ インデックスは非推奨（順序が変わると問題）
{items.map((item, index) => <li key={index}>{item}</li>)}

// ✅ ユニークなIDを使う
{items.map(item => <li key={item.id}>{item.name}</li>)}
```

### オブジェクトの配列

```tsx
const users = [
  { id: 1, name: "Alice", age: 25 },
  { id: 2, name: "Bob", age: 30 },
  { id: 3, name: "Charlie", age: 35 }
];

const element = (
  <ul>
    {users.map(user => (
      <li key={user.id}>
        {user.name} ({user.age})
      </li>
    ))}
  </ul>
);
```

## 6. JSX のルール

### 1. 単一のルート要素

```tsx
// ❌ エラー：複数のルート要素
const Component = () => {
  return (
    <h1>Title</h1>
    <p>Content</p>
  );
};

// ✅ div で囲む
const Component = () => {
  return (
    <div>
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
};

// ✅ Fragment を使う（余計なDOMを追加しない）
const Component = () => {
  return (
    <>
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
};
```

### 2. キャメルケースの属性

```tsx
// ✅ キャメルケース
<div
  className="container"
  onClick={handleClick}
  tabIndex={0}
  aria-label="Close"
/>

// onClick, onChange, onSubmit など
// backgroundColor, fontSize など（CSS）
```

### 3. スタイルはオブジェクト

```tsx
// ✅ オブジェクトで指定
<div style={{ color: 'red', fontSize: '16px' }}>
  Styled text
</div>

// または
const style = {
  color: 'red',
  fontSize: '16px'
};
<div style={style}>Styled text</div>
```

## 7. TypeScript での型付け

### JSX.Element

```tsx
const element: JSX.Element = <h1>Hello</h1>;
```

### React.ReactNode（より広い型）

```tsx
const content: React.ReactNode = "text"; // string
const content2: React.ReactNode = 123; // number
const content3: React.ReactNode = <h1>Hello</h1>; // JSX.Element
const content4: React.ReactNode = null; // null/undefined
```

### コンポーネントの返り値

```tsx
const Component = (): JSX.Element => {
  return <div>Hello</div>;
};

// または型推論に任せる
const Component = () => {
  return <div>Hello</div>;
};
```

## ✅ ベストプラクティス

### 1. Fragment を活用

```tsx
// ✅ 不要な div を避ける
<>
  <h1>Title</h1>
  <p>Content</p>
</>
```

### 2. 条件は読みやすく

```tsx
// ❌ 複雑
{isLoggedIn ? isAdmin ? <AdminPanel /> : <UserPanel /> : <LoginForm />}

// ✅ コンポーネント内で分岐
const Dashboard = ({ isLoggedIn, isAdmin }) => {
  if (!isLoggedIn) return <LoginForm />;
  if (isAdmin) return <AdminPanel />;
  return <UserPanel />;
};
```

### 3. key は安定したIDを使う

```tsx
// ✅ ユニークなID
{items.map(item => <Item key={item.id} {...item} />)}

// ⚠️ インデックスは避ける
{items.map((item, i) => <Item key={i} {...item} />)}
```

## 📝 練習問題

### 問題1: ユーザーリストの表示

次のユーザー配列を表示してください。

```tsx
const users = [
  { id: 1, name: "Alice", isActive: true },
  { id: 2, name: "Bob", isActive: false },
  { id: 3, name: "Charlie", isActive: true }
];
```

<details>
<summary>解答</summary>

```tsx
<ul>
  {users.map(user => (
    <li key={user.id}>
      {user.name} {user.isActive && "(Active)"}
    </li>
  ))}
</ul>
```
</details>

## 🔗 次のステップ

次は [コンポーネントの作成](./02-components.md) について学びます。

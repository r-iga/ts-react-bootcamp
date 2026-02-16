# コンポーネントの作成

## 📖 概要

Reactコンポーネントは、UIを構築するための再利用可能な部品です。
関数コンポーネントが現代のReact開発の標準となっています。

## 🎯 学習目標

- 関数コンポーネントを作成できる
- Props を正しく扱える
- コンポーネントの責務を理解する

## 1. 関数コンポーネント

### 基本形

```tsx
const Welcome = () => {
  return <h1>Hello, World!</h1>;
};
```

### TypeScriptでの型付け

```tsx
const Welcome = (): JSX.Element => {
  return <h1>Hello, World!</h1>;
};

// または型推論に任せる（推奨）
const Welcome = () => {
  return <h1>Hello, World!</h1>;
};
```

### アロー関数

```tsx
const Welcome = () => {
  return <h1>Hello, World!</h1>;
};

// または省略形
const Welcome = () => <h1>Hello, World!</h1>;
```

## 2. Props - データの受け渡し

### Props の基本

```tsx
type GreetingProps = {
  name: string;
};

const Greeting = ({ name }: GreetingProps) => {
  return <h1>Hello, {name}!</h1>;
};

// 使用
<Greeting name="Alice" />
```

### 複数のProps

```tsx
type UserCardProps = {
  name: string;
  age: number;
  email: string;
};

const UserCard = ({ name, age, email }: UserCardProps) => {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
    </div>
  );
};

// 使用
<UserCard name="Alice" age={25} email="alice@example.com" />
```

### オプショナルなProps

```tsx
type ButtonProps = {
  text: string;
  variant?: "primary" | "secondary"; // オプショナル
};

const Button = ({ text, variant = "primary" }: ButtonProps) => {
  return (
    <button className={`btn btn-${variant}`}>
      {text}
    </button>
  );
};

// 使用
<Button text="Click me" />
<Button text="Cancel" variant="secondary" />
```

### デフォルト値

```tsx
type CardProps = {
  title: string;
  description?: string;
  variant?: "default" | "outlined";
};

const Card = ({ 
  title, 
  description = "No description",
  variant = "default"
}: CardProps) => {
  return (
    <div className={`card card-${variant}`}>
      <h3>{title}</h3>
      <p>{description}</p>
    </div>
  );
};
```

## 3. children Props

### children とは

コンポーネントの内側に渡される要素。

```tsx
type CardProps = {
  title: string;
  children: React.ReactNode;
};

const Card = ({ title, children }: CardProps) => {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div className="card-content">
        {children}
      </div>
    </div>
  );
};

// 使用
<Card title="My Card">
  <p>This is the content.</p>
  <button>Click me</button>
</Card>
```

### React.ReactNode とは

```typescript
// React.ReactNode は以下を含む
type ReactNode = 
  | string 
  | number 
  | boolean 
  | null 
  | undefined
  | ReactElement 
  | ReactNode[];

// 使用例
const text: React.ReactNode = "Hello";
const element: React.ReactNode = <div>Hello</div>;
const array: React.ReactNode = [<div key="1">A</div>, <div key="2">B</div>];
```

## 4. コンポーネントの合成

### 小さなコンポーネントを組み合わせる

```tsx
const Avatar = ({ src, alt }: { src: string; alt: string }) => {
  return <img src={src} alt={alt} className="avatar" />;
};

const UserName = ({ name }: { name: string }) => {
  return <h3>{name}</h3>;
};

const UserCard = ({ user }: { user: User }) => {
  return (
    <div className="user-card">
      <Avatar src={user.avatar} alt={user.name} />
      <UserName name={user.name} />
      <p>{user.email}</p>
    </div>
  );
};
```

## 5. Props のスプレッド

### 残りのPropsを渡す

```tsx
type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
  variant?: "primary" | "secondary";
};

const Button = ({ variant = "primary", children, ...rest }: ButtonProps) => {
  return (
    <button className={`btn btn-${variant}`} {...rest}>
      {children}
    </button>
  );
};

// onClick, disabled などのHTML属性がすべて使える
<Button onClick={handleClick} disabled={isLoading}>
  Submit
</Button>
```

## 6. コンポーネントの命名規則

### PascalCase（パスカルケース）

```tsx
// ✅ 推奨
const UserCard = () => {};
const TodoList = () => {};
const MyComponent = () => {};

// ❌ NG
const userCard = () => {};
const todolist = () => {};
```

### ファイル名

```
// ✅ 推奨
UserCard.tsx
TodoList.tsx
MyComponent.tsx

// または
user-card.tsx
todo-list.tsx
```

## 7. コンポーネントの責務

### 単一責任の原則

```tsx
// ❌ 詰め込みすぎ
const UserDashboard = () => {
  return (
    <div>
      {/* プロフィール */}
      <div>...</div>
      {/* 統計情報 */}
      <div>...</div>
      {/* 最近のアクティビティ */}
      <div>...</div>
    </div>
  );
};

// ✅ 責務を分離
const UserProfile = ({ user }: { user: User }) => {
  return <div>...</div>;
};

const UserStats = ({ stats }: { stats: Stats }) => {
  return <div>...</div>;
};

const RecentActivity = ({ activities }: { activities: Activity[] }) => {
  return <div>...</div>;
};

const UserDashboard = ({ user, stats, activities }) => {
  return (
    <div>
      <UserProfile user={user} />
      <UserStats stats={stats} />
      <RecentActivity activities={activities} />
    </div>
  );
};
```

## 8. Propsの型定義パターン

### 基本パターン

```tsx
type Props = {
  title: string;
  count: number;
};

const Component = ({ title, count }: Props) => {
  return <div>{title}: {count}</div>;
};
```

### インライン型定義

```tsx
const Component = ({ title, count }: { title: string; count: number }) => {
  return <div>{title}: {count}</div>;
};

// シンプルなコンポーネントならOK
// 複雑なら型エイリアスを使う
```

### ジェネリックコンポーネント

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
};

const List = <T,>({ items, renderItem }: ListProps<T>) => {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item)}</li>
      ))}
    </ul>
  );
};

// 使用
<List 
  items={users} 
  renderItem={(user) => <span>{user.name}</span>} 
/>
```

## ✅ ベストプラクティス

### 1. Props の型を定義する

```tsx
// ✅ 型付き
type Props = {
  name: string;
  age: number;
};

const Component = ({ name, age }: Props) => {};

// ❌ 型なし
const Component = ({ name, age }) => {};
```

### 2. 分割代入で受け取る

```tsx
// ✅ 分割代入
const Component = ({ name, age }: Props) => {
  return <div>{name}</div>;
};

// ⚠️ props オブジェクトで受け取ると冗長
const Component = (props: Props) => {
  return <div>{props.name}</div>;
};
```

### 3. コンポーネントは小さく保つ

```tsx
// ✅ 100行以下が目安
// 責務が明確
// テストしやすい
```

### 4. ファイル構成

```
components/
  UserCard/
    UserCard.tsx      # メインコンポーネント
    UserCard.test.tsx # テスト
    UserCard.css      # スタイル
    index.ts          # エクスポート
```

## 📝 練習問題

### 問題1: 商品カードの作成

次のような商品カードコンポーネントを作成してください。

- 商品名
- 価格
- 在庫の有無（"In Stock" / "Out of Stock"）

```tsx
type Product = {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
};
```

<details>
<summary>解答</summary>

```tsx
type ProductCardProps = {
  product: Product;
};

const ProductCard = ({ product }: ProductCardProps) => {
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>¥{product.price.toLocaleString()}</p>
      <p className={product.inStock ? "in-stock" : "out-of-stock"}>
        {product.inStock ? "In Stock" : "Out of Stock"}
      </p>
    </div>
  );
};
```
</details>

## 🔗 次のステップ

次は [Props とイベント処理](./03-props-events.md) について学びます。

# TypeScript の型ユーティリティ

## 📖 概要

TypeScript には既存の型から新しい型を作るためのユーティリティ型が用意されています。
これらを使いこなすことで、型安全性を高く保ちながら効率的に開発できます。

## 🎯 学習目標

- 主要なユーティリティ型を使える
- Reactで実用的な型を作れる
- 型の変換ができる

## 1. Partial<T> - 部分的オプション

すべてのプロパティをオプションにする。

```typescript
type User = {
  id: number;
  name: string;
  email: string;
};

// すべてオプションに
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; }

// 更新用の型
function updateUser(id: number, updates: Partial<User>) {
  // updates は部分的でOK
}

updateUser(1, { name: "New Name" }); // ✅ OK
updateUser(1, { email: "new@example.com" }); // ✅ OK
```

## 2. Required<T> - 必須化

すべてのプロパティを必須にする。

```typescript
type PartialUser = {
  id?: number;
  name?: string;
  email?: string;
};

type RequiredUser = Required<PartialUser>;
// { id: number; name: string; email: string; }
```

## 3. Readonly<T> - 読み取り専用

すべてのプロパティを読み取り専用にする。

```typescript
type User = {
  id: number;
  name: string;
};

type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = { id: 1, name: "Alice" };

// ❌ エラー：変更不可
user.name = "Bob";
```

### Reactでの活用

```typescript
type Props = Readonly<{
  title: string;
  onClose: () => void;
}>;

// コンポーネントのPropsは変更されるべきではない
function Modal({ title, onClose }: Props) {
  // ...
}
```

## 4. Pick<T, K> - プロパティの選択

特定のプロパティだけを選択して新しい型を作る。

```typescript
type User = {
  id: number;
  name: string;
  email: string;
  age: number;
};

// name と email だけ
type UserProfile = Pick<User, "name" | "email">;
// { name: string; email: string; }

// Reactでの使用例
type UserCardProps = Pick<User, "name" | "age">;

function UserCard({ name, age }: UserCardProps) {
  return <div>{name} ({age})</div>;
}
```

## 5. Omit<T, K> - プロパティの除外

特定のプロパティを除外して新しい型を作る。

```typescript
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

// password を除外
type PublicUser = Omit<User, "password">;
// { id: number; name: string; email: string; }

// 複数除外
type UserPreview = Omit<User, "email" | "password">;
// { id: number; name: string; }
```

### Reactでの活用

```typescript
// ボタンの Props から onClick を除外
type CustomButtonProps = Omit<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  "onClick"
> & {
  onPress: () => void;
};
```

## 6. Record<K, T> - レコード型

キーと値の型を指定してオブジェクト型を作る。

```typescript
// string キーで number 値
type StringNumberRecord = Record<string, number>;

const scores: StringNumberRecord = {
  math: 90,
  english: 85,
  science: 92
};

// ユニオン型のキー
type Status = "idle" | "loading" | "success" | "error";
type StatusMessages = Record<Status, string>;

const messages: StatusMessages = {
  idle: "待機中",
  loading: "読み込み中...",
  success: "成功しました",
  error: "エラーが発生しました"
};
```

### Reactでの活用

```typescript
type FormErrors = Record<string, string[]>;

const [errors, setErrors] = useState<FormErrors>({});

// バリデーションエラー
setErrors({
  email: ["メールアドレスが無効です"],
  password: ["8文字以上必要です", "大文字を含めてください"]
});
```

## 7. Exclude<T, U> - 型の除外

ユニオン型から特定の型を除外する。

```typescript
type Status = "idle" | "loading" | "success" | "error";

// "error" を除外
type NonErrorStatus = Exclude<Status, "error">;
// "idle" | "loading" | "success"

// 複数除外
type ActiveStatus = Exclude<Status, "idle" | "error">;
// "loading" | "success"
```

## 8. Extract<T, U> - 型の抽出

ユニオン型から特定の型だけを抽出する。

```typescript
type Status = "idle" | "loading" | "success" | "error";

// "loading" と "error" を抽出
type PendingStatus = Extract<Status, "loading" | "error">;
// "loading" | "error"
```

## 9. NonNullable<T> - null/undefined を除外

```typescript
type MaybeString = string | null | undefined;

type DefiniteString = NonNullable<MaybeString>;
// string

// 配列の要素から null/undefined を除外
type Item = { id: number; name: string } | null;
type Items = NonNullable<Item>[];
```

## 10. ReturnType<T> - 関数の返り値の型

```typescript
function getUser() {
  return { id: 1, name: "Alice", age: 25 };
}

type User = ReturnType<typeof getUser>;
// { id: number; name: string; age: number; }

// API関数の返り値の型
async function fetchUsers() {
  const response = await fetch("/api/users");
  return response.json();
}

type UsersResponse = ReturnType<typeof fetchUsers>;
```

## 11. Parameters<T> - 関数の引数の型

```typescript
function createUser(name: string, age: number) {
  return { name, age };
}

type CreateUserParams = Parameters<typeof createUser>;
// [string, number]

// 引数を分割代入
const [name, age]: CreateUserParams = ["Alice", 25];
```

## 12. 実践的な組み合わせ

### フォームの型

```typescript
type User = {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
};

// フォーム用：id と createdAt を除外
type UserFormData = Omit<User, "id" | "createdAt">;

// 更新用：部分的でOK
type UserUpdateData = Partial<UserFormData>;
```

### API レスポンスの型

```typescript
type ApiResponse<T> = {
  data: T;
  status: "success" | "error";
  message?: string;
};

type UsersResponse = ApiResponse<User[]>;
type UserResponse = ApiResponse<User>;
```

### Reactコンポーネントの型

```typescript
// HTMLボタンの属性を継承
type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
  variant: "primary" | "secondary";
  isLoading?: boolean;
};

function Button({ variant, isLoading, children, ...props }: ButtonProps) {
  return (
    <button {...props} disabled={isLoading || props.disabled}>
      {children}
    </button>
  );
}
```

## ✅ ベストプラクティス

### 1. DRY - 型の再利用

```typescript
// ✅ 基本の型を定義
type User = {
  id: number;
  name: string;
  email: string;
};

// 派生型を作る
type UserPreview = Pick<User, "id" | "name">;
type UserFormData = Omit<User, "id">;

// ❌ 重複して定義しない
type UserPreview = {
  id: number;
  name: string;
};
```

### 2. 適切なユーティリティ型の選択

```typescript
// ✅ 更新用は Partial
function updateUser(id: number, data: Partial<User>) {}

// ✅ 表示用は Pick
type UserCard = Pick<User, "name" | "avatar">;

// ✅ 除外は Omit
type PublicUser = Omit<User, "password">;
```

### 3. 複雑な型は型エイリアスに

```typescript
// ✅ わかりやすい名前をつける
type FormState<T> = {
  data: T;
  errors: Partial<Record<keyof T, string[]>>;
  isSubmitting: boolean;
};

// 使用
const [state, setState] = useState<FormState<UserFormData>>({
  data: initialData,
  errors: {},
  isSubmitting: false
});
```

## 📝 練習問題

### 問題1: ユーザー型の派生

```typescript
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
  role: "admin" | "user";
  createdAt: Date;
};

// 1. パスワードを除いた公開用の型
// 2. 登録フォーム用の型（id と createdAt を除外）
// 3. 更新フォーム用の型（部分的に更新可能）
```

<details>
<summary>解答</summary>

```typescript
// 1. 公開用
type PublicUser = Omit<User, "password">;

// 2. 登録フォーム用
type UserRegisterData = Omit<User, "id" | "createdAt">;

// 3. 更新フォーム用
type UserUpdateData = Partial<Omit<User, "id" | "createdAt">>;
```
</details>

## 🔗 次のステップ

次は [Chapter 3: DOM操作](../03-dom-manipulation/) に進みましょう。

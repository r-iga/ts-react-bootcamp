# 分割代入（Destructuring）

## 📖 概要

分割代入は、配列やオブジェクトから値を取り出して、個別の変数に代入する便利な構文です。
コードを簡潔に書けるため、モダンな JavaScript では頻繁に使われます。

## 🎯 学習目標

- 配列の分割代入を使える
- オブジェクトの分割代入を使える
- デフォルト値を設定できる
- ネストした構造を分割代入できる

## 1. 配列の分割代入

### 基本的な使い方

```typescript
const colors = ["red", "green", "blue"];

// 従来の方法
const first = colors[0];
const second = colors[1];
const third = colors[2];

// ✅ 分割代入：簡潔
const [first, second, third] = colors;

console.log(first);  // "red"
console.log(second); // "green"
console.log(third);  // "blue"
```

### 一部だけ取得

```typescript
const colors = ["red", "green", "blue", "yellow"];

// 最初の2つだけ取得
const [first, second] = colors;

console.log(first);  // "red"
console.log(second); // "green"
```

### 要素をスキップ

```typescript
const colors = ["red", "green", "blue"];

// 2番目をスキップ
const [first, , third] = colors;

console.log(first); // "red"
console.log(third); // "blue"
```

### デフォルト値

配列に要素がない場合のデフォルト値を設定できます。

```typescript
const colors = ["red"];

const [first, second = "green", third = "blue"] = colors;

console.log(first);  // "red"
console.log(second); // "green"（デフォルト値）
console.log(third);  // "blue"（デフォルト値）
```

### 変数の交換

```typescript
let a = 1;
let b = 2;

// 従来の方法
let temp = a;
a = b;
b = temp;

// ✅ 分割代入：簡潔
[a, b] = [b, a];

console.log(a); // 2
console.log(b); // 1
```

### Rest 要素（残りすべて）

```typescript
const numbers = [1, 2, 3, 4, 5];

const [first, second, ...rest] = numbers;

console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]
```

## 2. オブジェクトの分割代入

### 基本的な使い方

```typescript
const user = {
  name: "Alice",
  age: 25,
  city: "Tokyo"
};

// 従来の方法
const name = user.name;
const age = user.age;
const city = user.city;

// ✅ 分割代入：簡潔
const { name, age, city } = user;

console.log(name); // "Alice"
console.log(age);  // 25
console.log(city); // "Tokyo"
```

### 変数名を変える

```typescript
const user = {
  name: "Alice",
  age: 25
};

// name を userName に、age を userAge にリネーム
const { name: userName, age: userAge } = user;

console.log(userName); // "Alice"
console.log(userAge);  // 25

// name は存在しない
console.log(name); // ReferenceError
```

### デフォルト値

```typescript
const user = {
  name: "Alice"
  // age プロパティがない
};

const { name, age = 0 } = user;

console.log(name); // "Alice"
console.log(age);  // 0（デフォルト値）
```

### リネームとデフォルト値の組み合わせ

```typescript
const user = {
  name: "Alice"
};

const { name: userName, age: userAge = 0 } = user;

console.log(userName); // "Alice"
console.log(userAge);  // 0
```

### ネストしたオブジェクト

```typescript
const user = {
  name: "Alice",
  address: {
    city: "Tokyo",
    zipCode: "100-0001"
  }
};

// ネストした値を取得
const {
  name,
  address: { city, zipCode }
} = user;

console.log(name);    // "Alice"
console.log(city);    // "Tokyo"
console.log(zipCode); // "100-0001"

// ⚠️ address 自体は変数として存在しない
console.log(address); // ReferenceError
```

address も変数として欲しい場合：

```typescript
const {
  name,
  address,
  address: { city, zipCode }
} = user;

console.log(address); // { city: "Tokyo", zipCode: "100-0001" }
console.log(city);    // "Tokyo"
```

### Rest プロパティ（残りすべて）

```typescript
const user = {
  name: "Alice",
  age: 25,
  city: "Tokyo",
  country: "Japan"
};

const { name, age, ...rest } = user;

console.log(name); // "Alice"
console.log(age);  // 25
console.log(rest); // { city: "Tokyo", country: "Japan" }
```

## 3. 関数の引数での分割代入

非常によく使われるパターンです。

### オブジェクトの分割代入

```typescript
// ❌ 従来の方法
function greet(user) {
  console.log(`Hello, ${user.name}! You are ${user.age} years old.`);
}

// ✅ 分割代入：引数で直接
function greet({ name, age }) {
  console.log(`Hello, ${name}! You are ${age} years old.`);
}

greet({ name: "Alice", age: 25 });
// "Hello, Alice! You are 25 years old."
```

### TypeScript での型付け

```typescript
type User = {
  name: string;
  age: number;
  city?: string;
};

function greet({ name, age, city = "Unknown" }: User) {
  console.log(`${name}, ${age}, ${city}`);
}

greet({ name: "Alice", age: 25 });
// "Alice, 25, Unknown"
```

### デフォルト値と組み合わせ

```typescript
function createUser({
  name = "Guest",
  age = 0,
  isActive = false
} = {}) {
  return { name, age, isActive };
}

console.log(createUser());
// { name: "Guest", age: 0, isActive: false }

console.log(createUser({ name: "Alice", age: 25 }));
// { name: "Alice", age: 25, isActive: false }
```

⚠️ `= {}` を忘れると、引数なしで呼び出せません。

```typescript
// ❌ 引数なしで呼び出すとエラー
function func({ x, y }) { }
func(); // TypeError

// ✅ デフォルトの空オブジェクトを設定
function func({ x, y } = {}) { }
func(); // OK
```

### 配列の分割代入

```typescript
function printCoordinates([x, y]: [number, number]) {
  console.log(`X: ${x}, Y: ${y}`);
}

printCoordinates([10, 20]); // "X: 10, Y: 20"
```

## 4. React での典型的な使用例

### Props の分割代入

```typescript
// ❌ props を毎回参照
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.age}</p>
      <p>{props.city}</p>
    </div>
  );
}

// ✅ 分割代入で簡潔に
function UserCard({ name, age, city }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{age}</p>
      <p>{city}</p>
    </div>
  );
}
```

### useState の分割代入

```typescript
import { useState } from 'react';

function Counter() {
  // useState は配列を返す
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

### useEffect などのフックも同様

```typescript
import { useQuery } from '@tanstack/react-query';

function Users() {
  // オブジェクトの分割代入
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error</div>;
  
  return <div>{/* data を使う */}</div>;
}
```

## 5. 配列とオブジェクトの組み合わせ

```typescript
const users = [
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 35 }
];

// 配列の1番目の要素（オブジェクト）から name を取得
const [, { name: secondName }] = users;

console.log(secondName); // "Bob"
```

### 実用例：APIレスポンスの処理

```typescript
const response = {
  data: {
    user: {
      profile: {
        name: "Alice",
        email: "alice@example.com"
      }
    }
  },
  status: 200
};

const {
  data: {
    user: {
      profile: { name, email }
    }
  },
  status
} = response;

console.log(name);   // "Alice"
console.log(email);  // "alice@example.com"
console.log(status); // 200
```

## ✅ ベストプラクティス

### 1. 必要な値だけ取り出す

```typescript
const user = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  age: 25,
  address: { /* ... */ },
  preferences: { /* ... */ }
  // ... 他にも多数のプロパティ
};

// ❌ すべてを分割代入
const { id, name, email, age, address, preferences, ... } = user;

// ✅ 必要なものだけ
const { name, email } = user;
```

### 2. 深いネストは避ける

```typescript
// ❌ 読みにくい
const {
  data: {
    user: {
      profile: {
        contact: {
          email
        }
      }
    }
  }
} = response;

// ✅ 段階的に分割
const { data } = response;
const { user } = data;
const { profile } = user;
const { email } = profile.contact;
```

### 3. 関数の引数では積極的に使う

```typescript
// ✅ 推奨：何を受け取るか明確
function updateUser({ id, name, email }: User) {
  // ...
}

// ⚠️ オブジェクトをそのまま受け取ると中身が不明
function updateUser(user: User) {
  // user.id, user.name, user.email ...
}
```

### 4. TypeScript では型も明示

```typescript
// ✅ 型付き分割代入
const { name, age }: { name: string; age: number } = user;

// または型エイリアス
type User = {
  name: string;
  age: number;
};

const { name, age }: User = user;
```

## 🔍 よくある質問

### Q: オブジェクトの分割代入で順番は関係ある？

A: **ありません**。プロパティ名でマッチングされます。

```typescript
const user = { name: "Alice", age: 25 };

// どちらも同じ結果
const { name, age } = user;
const { age, name } = user;
```

### Q: 配列の分割代入で順番は関係ある？

A: **あります**。配列は順序が重要です。

```typescript
const colors = ["red", "green"];

const [first, second] = colors;
// first = "red", second = "green"
```

### Q: 存在しないプロパティを分割代入したら？

A: `undefined` になります。

```typescript
const user = { name: "Alice" };
const { name, age } = user;

console.log(name); // "Alice"
console.log(age);  // undefined
```

## 📝 練習問題

### 問題1: 配列の分割代入

次の配列から1番目と3番目の要素を取得してください。

```typescript
const fruits = ["apple", "banana", "orange", "grape"];

// ここに分割代入を書く
const [___, ___, ___] = fruits;

console.log(first); // "apple"
console.log(third); // "orange"
```

<details>
<summary>解答</summary>

```typescript
const [first, , third] = fruits;
```
</details>

### 問題2: オブジェクトの分割代入

ユーザー情報から name と city を取得し、age にはデフォルト値 0 を設定してください。

```typescript
const user = {
  name: "Alice",
  address: {
    city: "Tokyo"
  }
};

// ここに分割代入を書く

console.log(name); // "Alice"
console.log(city); // "Tokyo"
console.log(age);  // 0
```

<details>
<summary>解答</summary>

```typescript
const {
  name,
  address: { city },
  age = 0
} = user;
```
</details>

### 問題3: 関数の引数

次の関数を分割代入を使って書き換えてください。

```typescript
function printUser(user) {
  console.log(`Name: ${user.name}`);
  console.log(`Age: ${user.age}`);
  console.log(`City: ${user.city}`);
}
```

<details>
<summary>解答</summary>

```typescript
function printUser({ name, age, city }) {
  console.log(`Name: ${name}`);
  console.log(`Age: ${age}`);
  console.log(`City: ${city}`);
}
```
</details>

## 🔗 次のステップ

次は [Spread演算子 / Rest構文](./05-spread-rest.md) について学びます。

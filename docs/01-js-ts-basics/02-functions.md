# 関数

## 📖 概要

関数は**処理をまとめて再利用可能にする**ための仕組みです。
JavaScript/TypeScript には複数の関数定義方法があり、それぞれ特徴があります。

## 🎯 学習目標

- 関数宣言と関数式の違いを理解する
- アロー関数を使いこなす
- TypeScript で関数に型付けできる
- デフォルト引数と可変長引数を使える

## 1. 関数宣言（Function Declaration）

最も基本的な関数の定義方法です。

```typescript
function greet(name: string): string {
  return `Hello, ${name}!`;
}

console.log(greet("Alice")); // "Hello, Alice!"
```

### 構文

```typescript
function 関数名(引数1: 型, 引数2: 型): 戻り値の型 {
  // 処理
  return 戻り値;
}
```

### 特徴

- `function` キーワードで定義
- 関数名が必須
- **巻き上げ（Hoisting）される**（定義前に呼び出し可能）

```typescript
// ✅ 定義前に呼び出せる
console.log(add(1, 2)); // 3

function add(a: number, b: number): number {
  return a + b;
}
```

## 2. 関数式（Function Expression）

関数を変数に代入する方法です。

```typescript
const greet = function(name: string): string {
  return `Hello, ${name}!`;
};

console.log(greet("Bob")); // "Hello, Bob!"
```

### 特徴

- 変数に関数を代入
- 関数名は省略可能（無名関数）
- **巻き上げされない**（定義前の呼び出しはエラー）

```typescript
// ❌ 定義前には呼び出せない
console.log(add(1, 2)); // ReferenceError

const add = function(a: number, b: number): number {
  return a + b;
};
```

## 3. アロー関数（Arrow Function）

ES6 で追加された**モダンな関数定義方法**です。簡潔に書けます。

```typescript
const greet = (name: string): string => {
  return `Hello, ${name}!`;
};

console.log(greet("Charlie")); // "Hello, Charlie!"
```

### 短縮記法

**1行の場合、`return` と `{}` を省略できます。**

```typescript
// 通常
const add = (a: number, b: number): number => {
  return a + b;
};

// 短縮版
const add = (a: number, b: number): number => a + b;

console.log(add(3, 4)); // 7
```

### 引数が1つの場合

**引数が1つの場合、`()` も省略できます。**（TypeScript では型を付ける場合は省略不可）

```typescript
// JavaScript なら
const double = x => x * 2;

// TypeScript では型を付ける場合
const double = (x: number): number => x * 2;

console.log(double(5)); // 10
```

### 引数がない場合

```typescript
const sayHello = (): string => "Hello!";

console.log(sayHello()); // "Hello!"
```

### ⚠️ オブジェクトを返す場合の注意

オブジェクトリテラルを返す場合は `()` で囲む必要があります。

```typescript
// ❌ エラー：関数本体と間違えられる
const createUser = (name: string) => { name: name };

// ✅ 正しい：() で囲む
const createUser = (name: string) => ({ name: name });

console.log(createUser("Alice")); // { name: "Alice" }
```

## 4. 関数宣言 vs 関数式 vs アロー関数

### どれを使うべき？

**推奨：基本的にアロー関数を使う**

```typescript
// ✅ 推奨：アロー関数
const add = (a: number, b: number): number => a + b;
const greet = (name: string): string => `Hello, ${name}!`;

// ⚠️ 巻き上げが必要な場合のみ関数宣言
function initialize() {
  // 初期化処理
}

initialize(); // 定義前に呼び出せる
```

### 比較表

| 特徴 | 関数宣言 | 関数式 | アロー関数 |
|------|---------|--------|-----------|
| 巻き上げ | ✅ あり | ❌ なし | ❌ なし |
| 簡潔さ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| this の扱い | 動的 | 動的 | 静的（後で学習） |
| 推奨度 | △ | △ | ✅ |

## 5. 引数と戻り値の型付け

### 基本的な型付け

```typescript
// 引数と戻り値に型を指定
function add(a: number, b: number): number {
  return a + b;
}

// アロー関数でも同じ
const subtract = (a: number, b: number): number => a - b;
```

### 型推論

戻り値の型は省略できます（推論される）。

```typescript
// 戻り値の型は number と推論される
const multiply = (a: number, b: number) => a * b;
```

### 複数の型を返す関数

```typescript
// string または number を返す
const getValue = (useString: boolean): string | number => {
  return useString ? "text" : 123;
};
```

### void 型（戻り値なし）

```typescript
const logMessage = (message: string): void => {
  console.log(message);
  // return なし
};
```

## 6. デフォルト引数

引数にデフォルト値を設定できます。

```typescript
const greet = (name: string = "Guest"): string => {
  return `Hello, ${name}!`;
};

console.log(greet());        // "Hello, Guest!"
console.log(greet("Alice")); // "Hello, Alice!"
```

### 複数のデフォルト引数

```typescript
const createUser = (
  name: string = "Anonymous",
  age: number = 0,
  country: string = "Unknown"
) => {
  return { name, age, country };
};

console.log(createUser());
// { name: "Anonymous", age: 0, country: "Unknown" }

console.log(createUser("Alice", 25));
// { name: "Alice", age: 25, country: "Unknown" }

console.log(createUser("Bob", 30, "Japan"));
// { name: "Bob", age: 30, country: "Japan" }
```

### TypeScript での型推論

デフォルト値から型が推論されます。

```typescript
// age は number と推論される
const setAge = (age = 0) => {
  console.log(age);
};
```

## 7. オプショナル引数

引数を省略可能にできます（`?` を使用）。

```typescript
const greet = (name: string, greeting?: string): string => {
  if (greeting) {
    return `${greeting}, ${name}!`;
  }
  return `Hello, ${name}!`;
};

console.log(greet("Alice"));           // "Hello, Alice!"
console.log(greet("Bob", "Good morning")); // "Good morning, Bob!"
```

### デフォルト引数との違い

```typescript
// オプショナル引数：undefined になる可能性がある
const func1 = (x?: number) => {
  console.log(x); // number | undefined
};

// デフォルト引数：常に値が入る
const func2 = (x: number = 0) => {
  console.log(x); // number
};

func1();    // undefined
func2();    // 0
```

## 8. 可変長引数（Rest Parameters）

任意の個数の引数を受け取れます。

```typescript
const sum = (...numbers: number[]): number => {
  return numbers.reduce((total, num) => total + num, 0);
};

console.log(sum(1, 2, 3));       // 6
console.log(sum(1, 2, 3, 4, 5)); // 15
console.log(sum());              // 0
```

### 構文

```typescript
const func = (...args: 型[]): 戻り値の型 => {
  // args は配列として扱える
};
```

### 通常の引数と組み合わせ

```typescript
// 最初の引数は通常、残りは可変長
const greetAll = (greeting: string, ...names: string[]): string => {
  return names.map(name => `${greeting}, ${name}!`).join("\n");
};

console.log(greetAll("Hello", "Alice", "Bob", "Charlie"));
// Hello, Alice!
// Hello, Bob!
// Hello, Charlie!
```

## 9. 高階関数（Higher-Order Functions）

関数を引数に取る、または関数を返す関数です。

### 関数を引数に取る

```typescript
const calculate = (
  a: number,
  b: number,
  operation: (x: number, y: number) => number
): number => {
  return operation(a, b);
};

const add = (x: number, y: number) => x + y;
const multiply = (x: number, y: number) => x * y;

console.log(calculate(5, 3, add));      // 8
console.log(calculate(5, 3, multiply)); // 15
```

### 関数を返す

```typescript
const createMultiplier = (factor: number) => {
  return (value: number) => value * factor;
};

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

## ✅ ベストプラクティス

### 1. アロー関数を優先する

```typescript
// ✅ 推奨：アロー関数
const add = (a: number, b: number) => a + b;

// ⚠️ 特別な理由がない限り避ける
function add(a: number, b: number) {
  return a + b;
}
```

### 2. 1行で書けるなら短縮記法を使う

```typescript
// ✅ 推奨：簡潔
const double = (x: number) => x * 2;

// ❌ 冗長
const double = (x: number) => {
  return x * 2;
};
```

### 3. 型は明示的に（特に公開API）

```typescript
// ✅ 推奨：引数と戻り値の型を明示
export const calculateTotal = (price: number, tax: number): number => {
  return price * (1 + tax);
};

// ⚠️ 戻り値の型省略（関数内部なら可）
const helper = (x: number) => x * 2;
```

### 4. 関数は小さく、単一責任に

```typescript
// ❌ 複数の責任を持つ
const processUser = (user: User) => {
  validateUser(user);
  saveToDatabase(user);
  sendEmail(user);
  logActivity(user);
};

// ✅ 小さな関数に分割
const validateUser = (user: User) => { /* ... */ };
const saveUser = (user: User) => { /* ... */ };
const notifyUser = (user: User) => { /* ... */ };
```

## 🔍 よくある質問

### Q: function と arrow function、どっちを使う？

A: **基本的にアロー関数**を使いましょう。関数宣言は巻き上げが必要な特殊なケースのみ。

### Q: 戻り値の型は書かなくてもいい？

A: 省略可能ですが、**公開される関数やライブラリでは明示**が推奨されます。

### Q: デフォルト引数とオプショナル引数の使い分けは？

A:
- **デフォルト値がある** → デフォルト引数
- **呼び出し側で判断させたい** → オプショナル引数

## 📝 練習問題

### 問題1: アロー関数への変換

次の関数をアロー関数に書き換えてください。

```typescript
function multiply(a: number, b: number): number {
  return a * b;
}
```

<details>
<summary>解答</summary>

```typescript
const multiply = (a: number, b: number): number => a * b;
```
</details>

### 問題2: デフォルト引数

ユーザー情報を作成する関数を作成してください。
- `name` は必須
- `age` はデフォルト 0
- `country` はデフォルト "Unknown"

<details>
<summary>解答</summary>

```typescript
const createUser = (
  name: string,
  age: number = 0,
  country: string = "Unknown"
) => {
  return { name, age, country };
};
```
</details>

### 問題3: 可変長引数

任意の個数の数値を受け取り、その平均を返す関数を作成してください。

<details>
<summary>解答</summary>

```typescript
const average = (...numbers: number[]): number => {
  if (numbers.length === 0) return 0;
  const sum = numbers.reduce((total, num) => total + num, 0);
  return sum / numbers.length;
};

console.log(average(1, 2, 3, 4, 5)); // 3
```
</details>

## 🔗 次のステップ

次は [制御構文・式](./03-control-flow.md) について学びます。

# 変数定義

## 📖 概要

JavaScript には変数を宣言する3つの方法があります：`var`、`let`、`const`。
モダンな JavaScript では **`let` と `const` を使い、`var` は使いません**。

## 🎯 学習目標

- `let` と `const` の違いを理解する
- スコープの概念を理解する
- TypeScript での型推論と型アノテーションを使える

## 1. const - 再代入不可の変数

`const` は**再代入できない変数**を宣言します。値が変わらないことが保証されます。

```typescript
const name = "Alice";
console.log(name); // "Alice"

// ❌ エラー：再代入できない
name = "Bob"; // TypeError: Assignment to constant variable.
```

### const の使いどころ

基本的に**すべての変数を `const` で宣言**し、後で変更が必要になったら `let` に変更するのが推奨されます。

```typescript
const PI = 3.14159;
const MAX_SIZE = 100;
const API_URL = "https://api.example.com";
```

### ⚠️ オブジェクトや配列の中身は変更可能

`const` は再代入を防ぐだけで、**オブジェクトや配列の中身は変更できます**。

```typescript
const user = { name: "Alice", age: 25 };

// ✅ プロパティの変更は可能
user.age = 26;
console.log(user); // { name: "Alice", age: 26 }

// ❌ 再代入はエラー
user = { name: "Bob", age: 30 }; // エラー！
```

```typescript
const items = [1, 2, 3];

// ✅ 配列の操作は可能
items.push(4);
console.log(items); // [1, 2, 3, 4]

// ❌ 再代入はエラー
items = [5, 6, 7]; // エラー！
```

## 2. let - 再代入可能な変数

`let` は**再代入できる変数**を宣言します。値が変わる可能性がある場合に使います。

```typescript
let count = 0;
console.log(count); // 0

count = 1;
console.log(count); // 1

count = count + 1;
console.log(count); // 2
```

### let の使いどころ

カウンター、フラグ、ループ変数など、**値が変わる変数**に使います。

```typescript
// カウンター
let counter = 0;
counter++;

// フラグ
let isLoggedIn = false;
isLoggedIn = true;

// ループ
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

## 3. スコープ（変数の有効範囲）

### ブロックスコープ

`let` と `const` は**ブロックスコープ**を持ちます。`{}` で囲まれた範囲内でのみ有効です。

```typescript
{
  const x = 10;
  console.log(x); // 10
}

// ❌ ブロックの外ではアクセスできない
console.log(x); // ReferenceError: x is not defined
```

### if 文のスコープ

```typescript
const age = 20;

if (age >= 18) {
  const message = "成人です";
  console.log(message); // "成人です"
}

// ❌ if ブロックの外ではアクセスできない
console.log(message); // ReferenceError: message is not defined
```

### 関数スコープ

関数内で宣言された変数は、その関数内でのみ有効です。

```typescript
function greet() {
  const greeting = "Hello!";
  console.log(greeting); // "Hello!"
}

greet();

// ❌ 関数の外ではアクセスできない
console.log(greeting); // ReferenceError: greeting is not defined
```

### ループのスコープ

```typescript
for (let i = 0; i < 3; i++) {
  console.log(i); // 0, 1, 2
}

// ❌ ループの外ではアクセスできない
console.log(i); // ReferenceError: i is not defined
```

## 4. var（使わない）

`var` は古い変数宣言方法です。**モダンな JavaScript では使いません**。

### var の問題点

**問題1: 再宣言が可能**
```typescript
var x = 10;
var x = 20; // ✅ エラーにならない（混乱の元）
console.log(x); // 20
```

**問題2: 関数スコープのみ（ブロックスコープなし）**
```typescript
if (true) {
  var message = "Hello";
}
console.log(message); // "Hello" ← if の外でもアクセスできてしまう
```

**問題3: 巻き上げ（Hoisting）**
```typescript
console.log(x); // undefined（エラーにならない）
var x = 10;
```

### ✅ let/const を使うべき理由

```typescript
// const/let はブロックスコープ
if (true) {
  const message = "Hello";
}
console.log(message); // ❌ エラー（意図通り）

// const/let は再宣言不可
let y = 10;
let y = 20; // ❌ エラー（意図通り）
```

## 5. TypeScript での型付け

### 型推論（Type Inference）

TypeScript は値から自動的に型を推論します。

```typescript
const name = "Alice"; // string 型と推論される
const age = 25;       // number 型と推論される
const isActive = true; // boolean 型と推論される

// ❌ 型が合わないとエラー
name = 123; // エラー：Type 'number' is not assignable to type 'string'
```

### 型アノテーション（Type Annotation）

明示的に型を指定することもできます。

```typescript
const name: string = "Alice";
const age: number = 25;
const isActive: boolean = true;
```

### 型アノテーションが必要な場合

初期値がない場合や、型を明確にしたい場合に使います。

```typescript
// 初期値がない場合は型を指定
let username: string;
username = "Alice";

// 複数の型を許容する場合（後で学習）
let value: string | number;
value = "text";
value = 123;
```

### 配列とオブジェクトの型

```typescript
// 配列
const numbers: number[] = [1, 2, 3];
const names: string[] = ["Alice", "Bob"];

// オブジェクト
const user: { name: string; age: number } = {
  name: "Alice",
  age: 25
};
```

## ✅ ベストプラクティス

### 1. デフォルトは const を使う

```typescript
// ✅ 推奨：変更しない値は const
const API_KEY = "abc123";
const MAX_RETRY = 3;

// ❌ 非推奨：変更しないのに let
let API_KEY = "abc123";
```

### 2. 変更が必要な場合のみ let

```typescript
// ✅ 推奨：値が変わる場合は let
let counter = 0;
counter++;

// ❌ 非推奨：変更するのに const
const counter = 0;
counter++; // エラー！
```

### 3. var は使わない

```typescript
// ❌ 非推奨：var は使わない
var x = 10;

// ✅ 推奨：let か const を使う
let x = 10;
const y = 20;
```

### 4. 変数名は意味のある名前に

```typescript
// ❌ 非推奨：意味不明な変数名
const x = "Alice";
const y = 25;

// ✅ 推奨：意味のある変数名
const userName = "Alice";
const userAge = 25;
```

### 5. 変数のスコープを最小限に

```typescript
// ❌ 非推奨：不要に広いスコープ
const result = 0;
if (condition) {
  // result を使う
}

// ✅ 推奨：必要な場所でのみ宣言
if (condition) {
  const result = 0;
  // result を使う
}
```

## 🔍 よくある質問

### Q: const と let、どっちを使えばいい？

A: **迷ったら const を使う**。後で変更が必要になったら let に変更する。

### Q: const なのにオブジェクトの中身を変更できるのはなぜ？

A: const は**再代入を防ぐ**だけです。オブジェクトへの参照は変わっていないので、中身の変更は可能です。

```typescript
const user = { name: "Alice" };

// user という変数が指すオブジェクトは変わっていない
user.name = "Bob"; // ✅ OK

// user という変数が別のオブジェクトを指そうとしている
user = { name: "Bob" }; // ❌ エラー
```

### Q: TypeScript で型を書かなくてもいいの？

A: はい。TypeScript は型推論が優秀なので、明らかな場合は型を省略できます。

```typescript
// 型推論で十分
const name = "Alice"; // string と推論される

// 型を明示する必要がある場合
let value: string | number; // 複数の型を許容
```

## 📝 練習問題

### 問題1: 適切な変数宣言

次の変数を `const` または `let` で宣言してください。

```typescript
// ユーザーの名前（変更しない）
_____ userName = "Alice";

// カウンター（変更する）
_____ count = 0;
count++;

// 最大値（変更しない）
_____ MAX_VALUE = 100;

// フラグ（変更する）
_____ isActive = false;
isActive = true;
```

<details>
<summary>解答</summary>

```typescript
const userName = "Alice";
let count = 0;
const MAX_VALUE = 100;
let isActive = false;
```
</details>

### 問題2: スコープの理解

次のコードの出力は何でしょうか？

```typescript
const x = 10;

if (true) {
  const x = 20;
  console.log(x); // ?
}

console.log(x); // ?
```

<details>
<summary>解答</summary>

```
20
10
```

if ブロック内の `x` は別の変数（ブロックスコープ）です。
</details>

## 🔗 次のステップ

次は [関数](./02-functions.md) について学びます。

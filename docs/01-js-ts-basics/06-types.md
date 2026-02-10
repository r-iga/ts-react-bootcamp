# 変数の型

## 📖 概要

JavaScript の型システムとTypeScript での型定義方法を学びます。
型を正しく理解することで、バグを減らし、より安全なコードを書けるようになります。

## 🎯 学習目標

- プリミティブ型とオブジェクト型の違いを理解する
- TypeScript で適切に型を定義できる
- 型エイリアスとインターフェースを使い分けられる
- ユニオン型とインターセクション型を使える

## 1. プリミティブ型（Primitive Types）

プリミティブ型は**基本的なデータ型**で、値そのものを保持します。

### string（文字列）

```typescript
const name: string = "Alice";
const greeting: string = 'Hello';
const message: string = `Hello, ${name}!`; // テンプレートリテラル

// ❌ 型エラー
const wrong: string = 123; // Type 'number' is not assignable to type 'string'
```

### number（数値）

整数と小数の区別はありません。

```typescript
const age: number = 25;
const price: number = 99.99;
const hex: number = 0xff;    // 16進数
const binary: number = 0b1010; // 2進数
const octal: number = 0o777;  // 8進数

// ❌ 型エラー
const wrong: number = "123";
```

### boolean（真偽値）

```typescript
const isActive: boolean = true;
const isCompleted: boolean = false;

// ❌ 型エラー
const wrong: boolean = 1; // true/false のみ
```

### null と undefined

```typescript
const nothing: null = null;
const notDefined: undefined = undefined;

// null と undefined は異なる型
const value1: undefined = null; // ❌ 型エラー
const value2: null = undefined; // ❌ 型エラー
```

### symbol（シンボル）

一意な値を作成します（あまり使わない）。

```typescript
const sym1: symbol = Symbol("key");
const sym2: symbol = Symbol("key");

console.log(sym1 === sym2); // false（同じキーでも別物）
```

### bigint（大きな整数）

`Number.MAX_SAFE_INTEGER` を超える整数（あまり使わない）。

```typescript
const big: bigint = 9007199254740991n;
const alsoHuge: bigint = BigInt(9007199254740991);
```

## 2. オブジェクト型（Object Types）

オブジェクト型は**参照を保持**します。

### object

```typescript
const user: object = { name: "Alice", age: 25 };

// ⚠️ object 型ではプロパティにアクセスできない
console.log(user.name); // ❌ Property 'name' does not exist on type 'object'
```

### 適切なオブジェクト型定義

```typescript
// ✅ プロパティを明示的に定義
const user: { name: string; age: number } = {
  name: "Alice",
  age: 25
};

console.log(user.name); // ✅ OK
```

### array（配列）

```typescript
// 配列型の定義方法1
const numbers: number[] = [1, 2, 3, 4, 5];

// 配列型の定義方法2（あまり使わない）
const numbers2: Array<number> = [1, 2, 3, 4, 5];

// 複数の型
const mixed: (string | number)[] = ["text", 123, "hello"];

// 2次元配列
const matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6]
];
```

### function（関数）

```typescript
// 関数型の定義
const add: (a: number, b: number) => number = (a, b) => a + b;

// または
type AddFunction = (a: number, b: number) => number;
const add: AddFunction = (a, b) => a + b;
```

## 3. 値のコピー vs 参照のコピー

### プリミティブ型：値のコピー

```typescript
let a = 10;
let b = a; // 値がコピーされる

b = 20;

console.log(a); // 10（a は変わらない）
console.log(b); // 20
```

### オブジェクト型：参照のコピー

```typescript
const obj1 = { name: "Alice" };
const obj2 = obj1; // 参照がコピーされる

obj2.name = "Bob";

console.log(obj1.name); // "Bob"（!! obj1 も変わる）
console.log(obj2.name); // "Bob"
```

### 配列も同様

```typescript
const arr1 = [1, 2, 3];
const arr2 = arr1; // 参照のコピー

arr2.push(4);

console.log(arr1); // [1, 2, 3, 4]（!! arr1 も変わる）
console.log(arr2); // [1, 2, 3, 4]
```

### 独立したコピーを作る

```typescript
// Spread演算子で新しいオブジェクト/配列を作成
const obj1 = { name: "Alice" };
const obj2 = { ...obj1 };

obj2.name = "Bob";
console.log(obj1.name); // "Alice"（変わらない）

const arr1 = [1, 2, 3];
const arr2 = [...arr1];

arr2.push(4);
console.log(arr1); // [1, 2, 3]（変わらない）
```

## 4. TypeScript の型システム

### any 型（使わない）

すべての型を受け入れる（型チェックが無効）。

```typescript
let value: any = 123;
value = "text"; // OK
value = true;   // OK

// ⚠️ 型チェックなし（危険）
value.nonExistent(); // エラーにならない（実行時にエラー）
```

**❌ any は使わないでください。型安全性が失われます。**

### unknown 型

any より安全な「不明な型」。

```typescript
let value: unknown = 123;

// ❌ 型チェックなしでは使えない
console.log(value.toString()); // エラー

// ✅ 型ガードで確認してから使う
if (typeof value === "number") {
  console.log(value.toFixed(2)); // OK
}
```

### void 型

戻り値がない関数。

```typescript
const logMessage = (message: string): void => {
  console.log(message);
  // return なし
};
```

### never 型

決して値を返さない。

```typescript
// 常に例外を投げる
const throwError = (message: string): never => {
  throw new Error(message);
};

// 無限ループ
const infiniteLoop = (): never => {
  while (true) {}
};
```

## 5. 型エイリアス（Type Alias）

型に名前を付けて再利用できます。

```typescript
// 基本的な型エイリアス
type UserId = number;
type UserName = string;

const id: UserId = 123;
const name: UserName = "Alice";

// オブジェクト型
type User = {
  id: number;
  name: string;
  email: string;
  age?: number; // オプショナル
};

const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com"
  // age は省略可能
};
```

### readonly プロパティ

```typescript
type User = {
  readonly id: number; // 読み取り専用
  name: string;
};

const user: User = { id: 1, name: "Alice" };

// ❌ readonly プロパティは変更不可
user.id = 2; // Cannot assign to 'id' because it is a read-only property
```

## 6. インターフェース（Interface）

型エイリアスと似ていますが、オブジェクトの形を定義するのに特化しています。

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;
}

const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com"
};
```

### 型エイリアス vs インターフェース

**どちらも使えるが、プロジェクトで統一する。**

```typescript
// 型エイリアス
type User = {
  id: number;
  name: string;
};

// インターフェース
interface User {
  id: number;
  name: string;
}
```

**主な違い：**

| 機能 | Type | Interface |
|------|------|-----------|
| オブジェクト定義 | ✅ | ✅ |
| ユニオン型 | ✅ | ❌ |
| インターセクション | ✅ | ✅（extends） |
| 拡張 | `&` | `extends` |
| 宣言のマージ | ❌ | ✅ |

**推奨：**
- React コンポーネントの Props → どちらでも OK
- API レスポンスの型 → Type
- ライブラリの公開 API → Interface

## 7. ユニオン型（Union Types）

複数の型のうちいずれか。

```typescript
// string または number
let value: string | number;

value = "text"; // ✅ OK
value = 123;    // ✅ OK
value = true;   // ❌ エラー

// null 許容型
let name: string | null = null;
name = "Alice"; // ✅ OK
```

### リテラル型との組み合わせ

```typescript
type Status = "pending" | "approved" | "rejected";

let status: Status = "pending"; // ✅ OK
status = "approved";            // ✅ OK
status = "unknown";             // ❌ エラー
```

### 型ガード

```typescript
const processValue = (value: string | number) => {
  if (typeof value === "string") {
    // この中では value は string 型
    console.log(value.toUpperCase());
  } else {
    // この中では value は number 型
    console.log(value.toFixed(2));
  }
};
```

## 8. インターセクション型（Intersection Types）

複数の型を結合します。

```typescript
type Person = {
  name: string;
  age: number;
};

type Employee = {
  employeeId: number;
  department: string;
};

// Person と Employee 両方のプロパティを持つ
type Staff = Person & Employee;

const staff: Staff = {
  name: "Alice",
  age: 25,
  employeeId: 12345,
  department: "Engineering"
};
```

## 9. リテラル型（Literal Types）

特定の値のみを許容する型。

```typescript
// 文字列リテラル型
let direction: "left" | "right" | "up" | "down";
direction = "left";  // ✅ OK
direction = "north"; // ❌ エラー

// 数値リテラル型
let diceRoll: 1 | 2 | 3 | 4 | 5 | 6;
diceRoll = 3;  // ✅ OK
diceRoll = 7;  // ❌ エラー

// 真偽値リテラル型
let isTrue: true; // true のみ許容
isTrue = true;  // ✅ OK
isTrue = false; // ❌ エラー
```

## 10. 型アサーション（Type Assertion）

TypeScript コンパイラに型を伝える（使いすぎ注意）。

```typescript
// as を使う
const input = document.getElementById("myInput") as HTMLInputElement;
input.value = "Hello";

// <> を使う（JSX では使えない）
const input2 = <HTMLInputElement>document.getElementById("myInput");
```

**⚠️ 型アサーションは最終手段。型推論や型ガードを優先する。**

## ✅ ベストプラクティス

### 1. any を避ける

```typescript
// ❌ 非推奨
const data: any = fetchData();

// ✅ 推奨：適切な型を定義
type ApiResponse = {
  id: number;
  name: string;
};
const data: ApiResponse = fetchData();
```

### 2. 型推論を活用

```typescript
// ❌ 冗長：型が明らか
const name: string = "Alice";
const count: number = 0;

// ✅ 簡潔：型推論に任せる
const name = "Alice";    // string と推論される
const count = 0;         // number と推論される
```

### 3. 明示的な型定義（関数のシグネチャ）

```typescript
// ✅ 推奨：引数と戻り値の型を明示
const calculateTotal = (price: number, tax: number): number => {
  return price * (1 + tax);
};
```

### 4. オブジェクトは型を定義

```typescript
// ❌ インラインで定義（再利用できない）
const user: { id: number; name: string } = {
  id: 1,
  name: "Alice"
};

// ✅ 型エイリアス/インターフェースで定義
type User = {
  id: number;
  name: string;
};

const user: User = {
  id: 1,
  name: "Alice"
};
```

## 🔍 よくある質問

### Q: type と interface、どっちを使う？

A: どちらでも OK ですが、**プロジェクトで統一**してください。
- React の Props → どちらでも
- ユニオン型が必要 → type

### Q: 型推論と明示的な型、どっちが良い？

A: 
- **型が明らか** → 型推論
- **関数の引数・戻り値** → 明示的に
- **公開 API** → 明示的に

### Q: null と undefined の違いは？

A:
- **null**: 意図的に「値がない」
- **undefined**: 未定義・未初期化

## 📝 練習問題

### 問題1: 型定義

ユーザー情報の型を定義してください。
- id: 数値（必須）
- name: 文字列（必須）
- email: 文字列（オプショナル）
- age: 数値（オプショナル）

<details>
<summary>解答</summary>

```typescript
type User = {
  id: number;
  name: string;
  email?: string;
  age?: number;
};

// または
interface User {
  id: number;
  name: string;
  email?: string;
  age?: number;
}
```
</details>

### 問題2: ユニオン型

ステータスを表す型を定義してください。
"pending", "processing", "completed", "failed" のいずれか。

<details>
<summary>解答</summary>

```typescript
type Status = "pending" | "processing" | "completed" | "failed";
```
</details>

### 問題3: 関数の型

2つの数値を受け取り、大きい方を返す関数を型付きで実装してください。

<details>
<summary>解答</summary>

```typescript
const max = (a: number, b: number): number => {
  return a > b ? a : b;
};
```
</details>

## 🔗 次のステップ

これで Chapter 1 は完了です！
次は [Chapter 2: 配列とオブジェクト](../02-array-object/) に進みましょう。

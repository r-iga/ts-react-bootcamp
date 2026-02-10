# 配列メソッド

## 📖 概要

配列は JavaScript で最も頻繁に使うデータ構造の1つです。
配列メソッドを使いこなすことで、ループを書かずに簡潔なコードが書けるようになります。

## 🎯 学習目標

- map, filter, reduce などの高階関数を使える
- 配列を効率的に操作できる
- TypeScript で配列に適切に型付けできる
- React で配列を使ったレンダリングができる

## 1. 基本操作メソッド

### push / pop - 末尾の操作

```typescript
const fruits = ["apple", "banana"];

// 末尾に追加（元の配列を変更）
fruits.push("orange");
console.log(fruits); // ["apple", "banana", "orange"]

// 末尾から削除（元の配列を変更）
const last = fruits.pop();
console.log(last);   // "orange"
console.log(fruits); // ["apple", "banana"]
```

⚠️ **push/pop は元の配列を変更**します（ミュータブル）。

### unshift / shift - 先頭の操作

```typescript
const fruits = ["banana", "orange"];

// 先頭に追加（元の配列を変更）
fruits.unshift("apple");
console.log(fruits); // ["apple", "banana", "orange"]

// 先頭から削除（元の配列を変更）
const first = fruits.shift();
console.log(first);  // "apple"
console.log(fruits); // ["banana", "orange"]
```

### splice - 要素の追加・削除

```typescript
const fruits = ["apple", "banana", "orange", "grape"];

// インデックス 1 から 2個削除
const removed = fruits.splice(1, 2);
console.log(removed); // ["banana", "orange"]
console.log(fruits);  // ["apple", "grape"]

// インデックス 1 に要素を挿入
fruits.splice(1, 0, "kiwi", "mango");
console.log(fruits); // ["apple", "kiwi", "mango", "grape"]
```

### slice - 部分配列の取得

```typescript
const fruits = ["apple", "banana", "orange", "grape", "kiwi"];

// インデックス 1 から 3 まで（3は含まない）
const sliced = fruits.slice(1, 3);
console.log(sliced); // ["banana", "orange"]

// 元の配列は変わらない
console.log(fruits); // ["apple", "banana", "orange", "grape", "kiwi"]

// 全体のコピー
const copy = fruits.slice();
```

### concat - 配列の結合

```typescript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

const combined = arr1.concat(arr2);
console.log(combined); // [1, 2, 3, 4, 5, 6]

// 元の配列は変わらない
console.log(arr1); // [1, 2, 3]

// ✅ 現代的な書き方：Spread演算子
const combined2 = [...arr1, ...arr2];
```

### join - 文字列への変換

```typescript
const words = ["Hello", "World", "!"];

const sentence = words.join(" ");
console.log(sentence); // "Hello World !"

const csv = ["apple", "banana", "orange"].join(",");
console.log(csv); // "apple,banana,orange"
```

## 2. 高階関数

関数を引数に取るメソッド。配列操作の中心的存在です。

### map - 要素の変換

各要素を変換して新しい配列を作成します。

```typescript
const numbers = [1, 2, 3, 4, 5];

// 各要素を2倍に
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// 元の配列は変わらない
console.log(numbers); // [1, 2, 3, 4, 5]
```

```typescript
// オブジェクトの変換
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" }
];

const names = users.map(user => user.name);
console.log(names); // ["Alice", "Bob"]

// プロパティを追加
const usersWithStatus = users.map(user => ({
  ...user,
  status: "active"
}));
```

### filter - 要素の絞り込み

条件に合う要素のみを抽出します。

```typescript
const numbers = [1, 2, 3, 4, 5, 6];

// 偶数のみ
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6]

// 3より大きい数
const greaterThan3 = numbers.filter(num => num > 3);
console.log(greaterThan3); // [4, 5, 6]
```

```typescript
// オブジェクトの絞り込み
const users = [
  { name: "Alice", age: 25 },
  { name: "Bob", age: 17 },
  { name: "Charlie", age: 30 }
];

const adults = users.filter(user => user.age >= 18);
// [{ name: "Alice", age: 25 }, { name: "Charlie", age: 30 }]
```

### reduce - 集約処理

配列を1つの値に集約します。

```typescript
const numbers = [1, 2, 3, 4, 5];

// 合計を計算
const sum = numbers.reduce((total, num) => total + num, 0);
console.log(sum); // 15

// 積を計算
const product = numbers.reduce((result, num) => result * num, 1);
console.log(product); // 120
```

```typescript
// 最大値を見つける
const max = numbers.reduce((max, num) => num > max ? num : max, numbers[0]);
console.log(max); // 5

// オブジェクトを作成
const fruits = ["apple", "banana", "apple", "orange", "banana", "apple"];

const count = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {} as Record<string, number>);

console.log(count); // { apple: 3, banana: 2, orange: 1 }
```

### find / findIndex - 検索

条件に合う最初の要素を見つけます。

```typescript
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
  { id: 3, name: "Charlie" }
];

// 要素を見つける
const user = users.find(user => user.id === 2);
console.log(user); // { id: 2, name: "Bob" }

// 見つからない場合は undefined
const notFound = users.find(user => user.id === 10);
console.log(notFound); // undefined

// インデックスを見つける
const index = users.findIndex(user => user.id === 2);
console.log(index); // 1
```

### some / every - 条件判定

```typescript
const numbers = [1, 2, 3, 4, 5];

// いずれかが条件を満たすか
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven); // true

// すべてが条件を満たすか
const allPositive = numbers.every(num => num > 0);
console.log(allPositive); // true

const allEven = numbers.every(num => num % 2 === 0);
console.log(allEven); // false
```

### forEach - ループ処理

```typescript
const fruits = ["apple", "banana", "orange"];

fruits.forEach((fruit, index) => {
  console.log(`${index}: ${fruit}`);
});
// 0: apple
// 1: banana
// 2: orange
```

⚠️ forEach は返り値がないため、**map や filter が使える場合はそちらを優先**してください。

## 3. その他の便利なメソッド

### includes - 要素の存在チェック

```typescript
const fruits = ["apple", "banana", "orange"];

console.log(fruits.includes("banana")); // true
console.log(fruits.includes("grape"));  // false
```

### indexOf / lastIndexOf - インデックスを取得

```typescript
const numbers = [1, 2, 3, 2, 1];

console.log(numbers.indexOf(2));     // 1（最初の位置）
console.log(numbers.lastIndexOf(2)); // 3（最後の位置）
console.log(numbers.indexOf(10));    // -1（見つからない）
```

### sort - ソート

```typescript
const numbers = [3, 1, 4, 1, 5, 9];

// ❌ 文字列として比較される
numbers.sort();
console.log(numbers); // [1, 1, 3, 4, 5, 9]（たまたま正しい）

// 数値の場合は比較関数が必要
const nums = [10, 5, 40, 25, 1000, 1];
nums.sort((a, b) => a - b); // 昇順
console.log(nums); // [1, 5, 10, 25, 40, 1000]

nums.sort((a, b) => b - a); // 降順
console.log(nums); // [1000, 40, 25, 10, 5, 1]
```

⚠️ **sort は元の配列を変更**します。

```typescript
// イミュータブルなソート
const sorted = [...numbers].sort((a, b) => a - b);
```

### reverse - 反転

```typescript
const numbers = [1, 2, 3, 4, 5];

numbers.reverse();
console.log(numbers); // [5, 4, 3, 2, 1]
```

⚠️ **reverse は元の配列を変更**します。

### flat - 配列のフラット化

```typescript
const nested = [1, [2, 3], [4, [5, 6]]];

console.log(nested.flat());    // [1, 2, 3, 4, [5, 6]]（1階層）
console.log(nested.flat(2));   // [1, 2, 3, 4, 5, 6]（2階層）
console.log(nested.flat(Infinity)); // 完全にフラット
```

### flatMap - map + flat

```typescript
const sentences = ["Hello World", "Good Morning"];

const words = sentences.flatMap(sentence => sentence.split(" "));
console.log(words); // ["Hello", "World", "Good", "Morning"]

// map + flat と同じ
const words2 = sentences.map(s => s.split(" ")).flat();
```

## 4. メソッドチェーン

複数のメソッドを連続して呼び出せます。

```typescript
const users = [
  { name: "Alice", age: 25, active: true },
  { name: "Bob", age: 17, active: false },
  { name: "Charlie", age: 30, active: true },
  { name: "David", age: 22, active: true }
];

// アクティブな成人ユーザーの名前を取得
const names = users
  .filter(user => user.active)       // アクティブのみ
  .filter(user => user.age >= 18)   // 18歳以上のみ
  .map(user => user.name)            // 名前のみ
  .sort();                            // ソート

console.log(names); // ["Alice", "Charlie", "David"]
```

## 5. TypeScript での型付け

### 配列の型定義

```typescript
// 基本的な配列型
const numbers: number[] = [1, 2, 3];
const names: string[] = ["Alice", "Bob"];

// ジェネリック構文（あまり使わない）
const numbers2: Array<number> = [1, 2, 3];

// オブジェクトの配列
type User = {
  id: number;
  name: string;
};

const users: User[] = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" }
];
```

### 読み取り専用配列

```typescript
const numbers: readonly number[] = [1, 2, 3];

// ❌ 変更不可
numbers.push(4);              // エラー
numbers[0] = 10;              // エラー

// ReadonlyArray 型も使える
const names: ReadonlyArray<string> = ["Alice", "Bob"];
```

### メソッドの型推論

```typescript
const numbers = [1, 2, 3];

// map の返り値は number[] と推論される
const doubled = numbers.map(n => n * 2);

// filter の返り値も number[] と推論される
const evens = numbers.filter(n => n % 2 === 0);
```

## ✅ ベストプラクティス

### 1. イミュータブルなメソッドを優先

```typescript
// ✅ 推奨：新しい配列を返す
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);

// ⚠️ 元の配列を変更
numbers.push(4);
numbers.sort();
```

### 2. forEach より map/filter

```typescript
// ❌ forEach で新しい配列を作成
const doubled: number[] = [];
numbers.forEach(n => doubled.push(n * 2));

// ✅ map を使う
const doubled = numbers.map(n => n * 2);
```

### 3. find が見つからない場合の処理

```typescript
const user = users.find(u => u.id === id);

// ❌ undefined チェックなし
console.log(user.name); // エラーの可能性

// ✅ オプショナルチェーン
console.log(user?.name);

// または
if (user) {
  console.log(user.name);
}
```

## 📝 練習問題

### 問題1: map と filter の組み合わせ

年齢が18歳以上のユーザーの名前のみを取得してください。

```typescript
const users = [
  { name: "Alice", age: 25 },
  { name: "Bob", age: 17 },
  { name: "Charlie", age: 30 }
];
```

<details>
<summary>解答</summary>

```typescript
const names = users
  .filter(user => user.age >= 18)
  .map(user => user.name);

console.log(names); // ["Alice", "Charlie"]
```
</details>

### 問題2: reduce で合計と平均

数値配列の合計と平均を計算してください。

```typescript
const scores = [85, 92, 78, 95, 88];
```

<details>
<summary>解答</summary>

```typescript
const sum = scores.reduce((total, score) => total + score, 0);
const average = sum / scores.length;

console.log(sum);     // 438
console.log(average); // 87.6
```
</details>

## 🔗 次のステップ

次は [オブジェクト操作](./02-object-operations.md) について学びます。

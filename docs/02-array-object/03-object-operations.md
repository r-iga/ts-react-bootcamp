# オブジェクト操作

## 📖 概要

オブジェクトはJavaScriptで最も重要なデータ構造です。
Object のメソッドを使いこなすことで、オブジェクトを効率的に操作できます。

## 🎯 学習目標

- Object の静的メソッドを使える
- プロパティアクセスの方法を理解する
- オブジェクトを安全に操作できる

## 1. Object.keys() - キーの取得

オブジェクトのすべてのキー（プロパティ名）を配列で取得します。

```typescript
const user = {
  name: "Alice",
  age: 25,
  city: "Tokyo"
};

const keys = Object.keys(user);
console.log(keys); // ["name", "age", "city"]

// for...of で反復
for (const key of Object.keys(user)) {
  console.log(key);
}
```

## 2. Object.values() - 値の取得

オブジェクトのすべての値を配列で取得します。

```typescript
const user = {
  name: "Alice",
  age: 25,
  city: "Tokyo"
};

const values = Object.values(user);
console.log(values); // ["Alice", 25, "Tokyo"]
```

## 3. Object.entries() - キーと値のペア

キーと値のペアを配列の配列として取得します。

```typescript
const user = {
  name: "Alice",
  age: 25,
  city: "Tokyo"
};

const entries = Object.entries(user);
console.log(entries);
// [["name", "Alice"], ["age", 25], ["city", "Tokyo"]]

// 分割代入で使いやすい
for (const [key, value] of Object.entries(user)) {
  console.log(`${key}: ${value}`);
}
// name: Alice
// age: 25
// city: Tokyo
```

## 4. Object.fromEntries() - エントリから作成

エントリの配列からオブジェクトを作成します。

```typescript
const entries = [
  ["name", "Alice"],
  ["age", 25],
  ["city", "Tokyo"]
];

const user = Object.fromEntries(entries);
console.log(user);
// { name: "Alice", age: 25, city: "Tokyo" }
```

### 活用例：オブジェクトの変換

```typescript
const prices = {
  apple: 100,
  banana: 80,
  orange: 120
};

// すべての価格を1.1倍に
const newPrices = Object.fromEntries(
  Object.entries(prices).map(([key, value]) => [key, value * 1.1])
);

console.log(newPrices);
// { apple: 110, banana: 88, orange: 132 }
```

## 5. Object.assign() - マージ

複数のオブジェクトを結合します。

```typescript
const defaults = { color: "blue", size: "medium" };
const options = { size: "large", disabled: false };

const merged = Object.assign({}, defaults, options);
console.log(merged);
// { color: "blue", size: "large", disabled: false }

// ✅ Spread演算子の方が推奨される
const merged2 = { ...defaults, ...options };
```

## 6. プロパティアクセス

### ドット記法

```typescript
const user = { name: "Alice", age: 25 };

console.log(user.name); // "Alice"
user.age = 26;
```

### ブラケット記法

```typescript
const user = { name: "Alice", age: 25 };

console.log(user["name"]); // "Alice"
user["age"] = 26;

// 動的なキー
const key = "name";
console.log(user[key]); // "Alice"
```

### 動的なプロパティ名

```typescript
const key = "dynamicKey";
const value = "dynamicValue";

const obj = {
  [key]: value,
  [key + "2"]: value + "2"
};

console.log(obj);
// { dynamicKey: "dynamicValue", dynamicKey2: "dynamicValue2" }
```

## 7. プロパティの存在チェック

### in 演算子

```typescript
const user = { name: "Alice", age: 25 };

console.log("name" in user);     // true
console.log("email" in user);    // false
console.log("toString" in user); // true（継承されたプロパティも含む）
```

### hasOwnProperty

```typescript
const user = { name: "Alice", age: 25 };

console.log(user.hasOwnProperty("name"));     // true
console.log(user.hasOwnProperty("toString")); // false（継承は除外）

// TypeScript で推奨される書き方
console.log(Object.hasOwn(user, "name")); // true
```

### undefined チェック

```typescript
const user = { name: "Alice", age: undefined };

// プロパティは存在するが値が undefined
console.log(user.age);              // undefined
console.log("age" in user);         // true
console.log(user.hasOwnProperty("age")); // true
```

## 8. Object.freeze() - 不変オブジェクト

オブジェクトを変更不可にします。

```typescript
const config = Object.freeze({
  apiUrl: "https://api.example.com",
  timeout: 3000
});

// ❌ 変更できない
config.timeout = 5000; // エラー（strictモード）
config.newProp = "value"; // エラー

console.log(config.timeout); // 3000（変わらない）
```

⚠️ **シャローフリーズ**：1階層のみ

```typescript
const obj = Object.freeze({
  name: "Alice",
  address: {
    city: "Tokyo"
  }
});

// ❌ 1階層目は変更不可
obj.name = "Bob"; // エラー

// ✅ ネストしたオブジェクトは変更可能
obj.address.city = "Osaka"; // OK（!!）
```

## 9. TypeScript での型安全な操作

### Record 型

```typescript
// キーと値の型を指定
const userAges: Record<string, number> = {
  alice: 25,
  bob: 30,
  charlie: 35
};

// ❌ 型エラー
userAges.david = "40";
```

### キーの型安全な取得

```typescript
type User = {
  name: string;
  age: number;
  email: string;
};

const user: User = {
  name: "Alice",
  age: 25,
  email: "alice@example.com"
};

// keyof で型安全に
const getProperty = <T, K extends keyof T>(obj: T, key: K): T[K] => {
  return obj[key];
};

const name = getProperty(user, "name"); // string
const age = getProperty(user, "age");   // number
```

## ✅ ベストプラクティス

### 1. Spread演算子を優先

```typescript
// ✅ 推奨
const merged = { ...obj1, ...obj2 };

// ⚠️ Object.assign より Spread が読みやすい
const merged = Object.assign({}, obj1, obj2);
```

### 2. for...in より Object.keys()

```typescript
const obj = { a: 1, b: 2, c: 3 };

// ⚠️ for...in（プロトタイプチェーンも含む）
for (const key in obj) {
  console.log(key);
}

// ✅ Object.keys()（自身のプロパティのみ）
for (const key of Object.keys(obj)) {
  console.log(key);
}
```

## 📝 練習問題

### 問題1: オブジェクトの変換

商品の価格を全て10%引きにしてください。

```typescript
const products = {
  apple: 100,
  banana: 80,
  orange: 120
};
```

<details>
<summary>解答</summary>

```typescript
const discounted = Object.fromEntries(
  Object.entries(products).map(([key, price]) => [key, price * 0.9])
);
```
</details>

## 🔗 次のステップ

次は [Map/Set オブジェクト](./04-map-set.md) について学びます。

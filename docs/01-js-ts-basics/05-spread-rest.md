# Spread演算子 / Rest構文

## 📖 概要

Spread演算子（`...`）と Rest構文（`...`）は、同じ記号ですが異なる用途で使われます。
配列やオブジェクトの操作を簡潔に書けるため、モダンな JavaScript では必須の知識です。

## 🎯 学習目標

- Spread演算子で配列・オブジェクトを展開できる
- Rest構文で残りの要素をまとめて取得できる
- イミュータブルな操作ができる
- React での実用的な使い方を理解する

## 1. Spread演算子（配列）

### 配列の展開

配列の要素を個別の値として展開します。

```typescript
const numbers = [1, 2, 3];

console.log(...numbers);
// 1 2 3（個別の値として展開）

// 関数の引数として展開
console.log(Math.max(...numbers));
// 3（Math.max(1, 2, 3) と同じ）
```

### 配列の結合

```typescript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// 従来の方法
const combined1 = arr1.concat(arr2);

// ✅ Spread演算子：簡潔
const combined2 = [...arr1, ...arr2];

console.log(combined2); // [1, 2, 3, 4, 5, 6]
```

### 配列のコピー

```typescript
const original = [1, 2, 3];

// ❌ 参照のコピー（元の配列も変わる）
const copy1 = original;
copy1.push(4);
console.log(original); // [1, 2, 3, 4]（!! 変わってしまう）

// ✅ Spread演算子で新しい配列を作成
const copy2 = [...original];
copy2.push(4);
console.log(original); // [1, 2, 3]（元の配列は変わらない）
```

### 要素の追加

```typescript
const numbers = [2, 3, 4];

// 先頭に追加
const withFirst = [1, ...numbers];
console.log(withFirst); // [1, 2, 3, 4]

// 末尾に追加
const withLast = [...numbers, 5];
console.log(withLast); // [2, 3, 4, 5]

// 途中に追加
const withMiddle = [1, 2, ...numbers, 5, 6];
console.log(withMiddle); // [1, 2, 2, 3, 4, 5, 6]
```

### 文字列の展開

```typescript
const str = "Hello";
const chars = [...str];

console.log(chars); // ["H", "e", "l", "l", "o"]
```

## 2. Spread演算子（オブジェクト）

### オブジェクトの展開

```typescript
const user = { name: "Alice", age: 25 };
const address = { city: "Tokyo", country: "Japan" };

// オブジェクトの結合
const combined = { ...user, ...address };

console.log(combined);
// { name: "Alice", age: 25, city: "Tokyo", country: "Japan" }
```

### オブジェクトのコピー

```typescript
const original = { name: "Alice", age: 25 };

// ❌ 参照のコピー
const copy1 = original;
copy1.age = 30;
console.log(original.age); // 30（!! 変わってしまう）

// ✅ Spread演算子で新しいオブジェクトを作成
const copy2 = { ...original };
copy2.age = 30;
console.log(original.age); // 25（元のオブジェクトは変わらない）
```

### プロパティの上書き

```typescript
const user = { name: "Alice", age: 25, city: "Tokyo" };

// age を上書き
const updated = { ...user, age: 30 };

console.log(updated);
// { name: "Alice", age: 30, city: "Tokyo" }
```

⚠️ 順序が重要です。

```typescript
const user = { name: "Alice", age: 25 };

// age: 30 が user.age に上書きされる
const result1 = { age: 30, ...user };
console.log(result1.age); // 25

// user.age が age: 30 で上書きされる
const result2 = { ...user, age: 30 };
console.log(result2.age); // 30
```

### プロパティの追加

```typescript
const user = { name: "Alice", age: 25 };

// email プロパティを追加
const withEmail = { ...user, email: "alice@example.com" };

console.log(withEmail);
// { name: "Alice", age: 25, email: "alice@example.com" }
```

### ⚠️ シャローコピー（浅いコピー）

Spread演算子は**1階層のみコピー**します。ネストしたオブジェクトは参照がコピーされます。

```typescript
const user = {
  name: "Alice",
  address: {
    city: "Tokyo"
  }
};

const copy = { ...user };

// ネストしたオブジェクトは参照が共有される
copy.address.city = "Osaka";
console.log(user.address.city); // "Osaka"（!! 元も変わる）

// 1階層目は独立している
copy.name = "Bob";
console.log(user.name); // "Alice"（元は変わらない）
```

ディープコピーが必要な場合：

```typescript
// 手動でネストもコピー
const deepCopy = {
  ...user,
  address: { ...user.address }
};

// またはstructuredClone（モダンブラウザ）
const deepCopy2 = structuredClone(user);

// またはJSON（日付やundefinedは扱えない）
const deepCopy3 = JSON.parse(JSON.stringify(user));
```

## 3. Rest構文（残りの要素をまとめる）

### 配列のRest

```typescript
const numbers = [1, 2, 3, 4, 5];

// 最初の2つと残り
const [first, second, ...rest] = numbers;

console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]
```

⚠️ Restは最後にしか置けません。

```typescript
// ❌ エラー
const [first, ...rest, last] = numbers;

// ✅ OK
const [first, ...rest] = numbers;
```

### オブジェクトのRest

```typescript
const user = {
  id: 1,
  name: "Alice",
  age: 25,
  city: "Tokyo"
};

// name と残りのプロパティ
const { name, ...rest } = user;

console.log(name); // "Alice"
console.log(rest); // { id: 1, age: 25, city: "Tokyo" }
```

### プロパティの除外

特定のプロパティを除外したオブジェクトを作成できます。

```typescript
const user = {
  id: 1,
  name: "Alice",
  password: "secret",
  email: "alice@example.com"
};

// password を除外
const { password, ...userWithoutPassword } = user;

console.log(userWithoutPassword);
// { id: 1, name: "Alice", email: "alice@example.com" }
```

### 関数の可変長引数

```typescript
// すべての引数を配列として受け取る
const sum = (...numbers: number[]): number => {
  return numbers.reduce((total, num) => total + num, 0);
};

console.log(sum(1, 2, 3));       // 6
console.log(sum(1, 2, 3, 4, 5)); // 15
```

### 通常の引数とRestの組み合わせ

```typescript
const multiply = (factor: number, ...numbers: number[]): number[] => {
  return numbers.map(num => num * factor);
};

console.log(multiply(2, 1, 2, 3)); // [2, 4, 6]
console.log(multiply(10, 5, 10));  // [50, 100]
```

## 4. React での実用例

### State の更新（イミュータブル）

```typescript
// 配列への要素追加
const [items, setItems] = useState([1, 2, 3]);

// ✅ Spread演算子で新しい配列を作成
setItems([...items, 4]); // [1, 2, 3, 4]

// 配列からの要素削除
const removeItem = (id: number) => {
  setItems(items.filter(item => item !== id));
};

// 配列の要素更新
const updateItem = (id: number, newValue: number) => {
  setItems(items.map(item => item === id ? newValue : item));
};
```

```typescript
// オブジェクトのプロパティ更新
const [user, setUser] = useState({
  name: "Alice",
  age: 25,
  city: "Tokyo"
});

// ✅ Spread演算子で新しいオブジェクトを作成
setUser({ ...user, age: 26 });

// 複数プロパティの更新
setUser({ ...user, age: 26, city: "Osaka" });
```

### Props の展開

```typescript
const userProps = {
  name: "Alice",
  age: 25,
  city: "Tokyo"
};

// ❌ 1つずつ渡す
<UserCard name={userProps.name} age={userProps.age} city={userProps.city} />

// ✅ Spread演算子で一括展開
<UserCard {...userProps} />
```

### Props の一部上書き

```typescript
const defaultProps = {
  size: "medium",
  color: "blue",
  disabled: false
};

// デフォルトProps + 一部上書き
<Button {...defaultProps} color="red" />
// size="medium", color="red", disabled=false
```

### 子コンポーネントへの Props 受け渡し

```typescript
type ButtonProps = {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
};

// 親コンポーネント
function Container(props: ButtonProps) {
  const { label, ...restProps } = props;
  
  // label 以外の Props を子に渡す
  return (
    <div>
      <h2>{label}</h2>
      <Button {...restProps} />
    </div>
  );
}
```

## 5. TypeScript での型付け

### Spread演算子の型

```typescript
const arr1: number[] = [1, 2, 3];
const arr2: number[] = [4, 5, 6];

// 型は自動的に推論される
const combined = [...arr1, ...arr2]; // number[]
```

### Rest構文の型

```typescript
// 可変長引数
const sum = (...numbers: number[]): number => {
  return numbers.reduce((a, b) => a + b, 0);
};

// 分割代入のRest
const [first, ...rest]: [number, ...number[]] = [1, 2, 3, 4];
```

### オブジェクトのRest型

```typescript
type User = {
  id: number;
  name: string;
  age: number;
};

const user: User = { id: 1, name: "Alice", age: 25 };

const { id, ...rest } = user;
// rest の型: { name: string; age: number }
```

## ✅ ベストプラクティス

### 1. イミュータブルな更新に使う

```typescript
// ✅ 推奨：新しい配列/オブジェクトを作成
const newArray = [...oldArray, newItem];
const newObject = { ...oldObject, key: newValue };

// ❌ 非推奨：元の配列/オブジェクトを変更
oldArray.push(newItem);
oldObject.key = newValue;
```

### 2. シャローコピーに注意

```typescript
// ⚠️ ネストしたオブジェクトは参照が共有される
const copy = { ...user };

// ✅ ネストもコピーする
const deepCopy = {
  ...user,
  address: { ...user.address }
};
```

### 3. 大きなオブジェクトのコピーは避ける

```typescript
// ❌ 毎回すべてコピー（パフォーマンス低下）
const updated = { ...hugeObject, key: newValue };

// ✅ 必要な部分だけ更新
const updated = {
  ...hugeObject,
  section: {
    ...hugeObject.section,
    key: newValue
  }
};

// または Immer ライブラリを使う（後で学習）
```

### 4. Props の展開は慎重に

```typescript
// ⚠️ 何が渡されるか不明確
<Component {...props} />

// ✅ 必要なものだけ展開
const { onClick, ...restProps } = props;
<Component onClick={onClick} {...restProps} />
```

## 📝 練習問題

### 問題1: 配列の操作

次の配列に要素を追加・削除してください。

```typescript
const numbers = [2, 3, 4];

// 先頭に 1 を追加
const withFirst = ___;

// 末尾に 5 を追加
const withLast = ___;

// 3 を削除
const without3 = ___;
```

<details>
<summary>解答</summary>

```typescript
const withFirst = [1, ...numbers];
const withLast = [...numbers, 5];
const without3 = numbers.filter(n => n !== 3);
```
</details>

### 問題2: オブジェクトの更新

ユーザー情報を更新してください。

```typescript
const user = { name: "Alice", age: 25, city: "Tokyo" };

// age を 26 に更新
const updated1 = ___;

// city を "Osaka" に更新し、country: "Japan" を追加
const updated2 = ___;
```

<details>
<summary>解答</summary>

```typescript
const updated1 = { ...user, age: 26 };
const updated2 = { ...user, city: "Osaka", country: "Japan" };
```
</details>

### 問題3: プロパティの除外

password プロパティを除外したオブジェクトを作成してください。

```typescript
const user = {
  id: 1,
  name: "Alice",
  password: "secret",
  email: "alice@example.com"
};

// password を除外
const safeUser = ___;
```

<details>
<summary>解答</summary>

```typescript
const { password, ...safeUser } = user;
```
</details>

## 🔗 次のステップ

次は [変数の型](./06-types.md) について学びます。

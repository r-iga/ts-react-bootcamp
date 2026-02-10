# 制御構文・式

## 📖 概要

プログラムの流れを制御する構文と、条件分岐や繰り返しを実現する方法を学びます。
モダンな JavaScript の便利な演算子も含めて解説します。

## 🎯 学習目標

- if、switch、for、while を使える
- 三項演算子で簡潔に条件分岐できる
- Nullish 合体演算子とオプショナルチェーンを使える
- 厳密等価演算子を正しく使える

## 1. if 文

条件によって処理を分岐します。

```typescript
const age = 20;

if (age >= 18) {
  console.log("成人です");
}
// "成人です"
```

### if-else

```typescript
const age = 15;

if (age >= 18) {
  console.log("成人です");
} else {
  console.log("未成年です");
}
// "未成年です"
```

### if-else if-else

```typescript
const score = 75;

if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else if (score >= 70) {
  console.log("C");
} else {
  console.log("D");
}
// "C"
```

### ネストした if

```typescript
const age = 25;
const hasLicense = true;

if (age >= 18) {
  if (hasLicense) {
    console.log("運転できます");
  } else {
    console.log("免許が必要です");
  }
} else {
  console.log("18歳未満は運転できません");
}
```

## 2. 三項演算子（Ternary Operator）

簡潔な条件分岐に使います。

### 基本構文

```typescript
const 結果 = 条件 ? 真の場合の値 : 偽の場合の値;
```

### 例

```typescript
const age = 20;
const status = age >= 18 ? "成人" : "未成年";
console.log(status); // "成人"
```

### if-else との比較

```typescript
// if-else
let message;
if (isLoggedIn) {
  message = "ようこそ";
} else {
  message = "ログインしてください";
}

// ✅ 三項演算子：簡潔
const message = isLoggedIn ? "ようこそ" : "ログインしてください";
```

### ネストした三項演算子

```typescript
const score = 75;
const grade = score >= 90 ? "A" :
              score >= 80 ? "B" :
              score >= 70 ? "C" : "D";
console.log(grade); // "C"
```

⚠️ 3段階以上のネストは読みにくいので避けましょう。

## 3. 論理演算子（Logical Operators）

### && (AND) - かつ

両方が true の場合のみ true

```typescript
const age = 25;
const hasLicense = true;

if (age >= 18 && hasLicense) {
  console.log("運転できます");
}
```

### || (OR) - または

どちらか一方が true なら true

```typescript
const isWeekend = true;
const isHoliday = false;

if (isWeekend || isHoliday) {
  console.log("休みです");
}
```

### ! (NOT) - 否定

true と false を反転

```typescript
const isLoggedIn = false;

if (!isLoggedIn) {
  console.log("ログインしてください");
}
```

### 短絡評価（Short-circuit Evaluation）

```typescript
// && は左が false なら右を評価しない
const user = null;
const name = user && user.name; // null（エラーにならない）

// || は左が true なら右を評価しない
const userName = inputName || "Guest"; // inputName が空なら "Guest"
```

## 4. Nullish 合体演算子（??）

`null` または `undefined` の場合のみ右側の値を返します。

```typescript
const value1 = null ?? "デフォルト";
console.log(value1); // "デフォルト"

const value2 = undefined ?? "デフォルト";
console.log(value2); // "デフォルト"

const value3 = 0 ?? "デフォルト";
console.log(value3); // 0（!! 0 は null ではない）

const value4 = "" ?? "デフォルト";
console.log(value4); // ""（!! 空文字は null ではない）
```

### || との違い

```typescript
// || は falsy な値すべてで右側を返す
const count1 = 0 || 10;
console.log(count1); // 10（0 は falsy）

// ?? は null/undefined のみで右側を返す
const count2 = 0 ?? 10;
console.log(count2); // 0（✅ 0 は有効な値）
```

### 使い分け

```typescript
// ✅ 0 や空文字を有効な値として扱いたい場合は ??
const port = config.port ?? 3000;
const userName = user.name ?? "Anonymous";

// || は真偽値的な判定の場合
const isEnabled = config.enabled || false;
```

## 5. オプショナルチェーン（?.）

プロパティが存在しない可能性がある場合、エラーを防ぎます。

```typescript
const user = {
  name: "Alice",
  address: {
    city: "Tokyo"
  }
};

// ✅ 安全にアクセス
console.log(user?.address?.city); // "Tokyo"
console.log(user?.profile?.bio);  // undefined（エラーにならない）

// ❌ オプショナルチェーンなしだとエラー
console.log(user.profile.bio); // TypeError: Cannot read property 'bio' of undefined
```

### 配列の要素アクセス

```typescript
const users = [{ name: "Alice" }, { name: "Bob" }];

console.log(users?.[0]?.name); // "Alice"
console.log(users?.[10]?.name); // undefined
```

### 関数呼び出し

```typescript
const obj = {
  method: () => "Hello"
};

console.log(obj.method?.()); // "Hello"
console.log(obj.missing?.()); // undefined（エラーにならない）
```

### 従来の書き方との比較

```typescript
// 従来の書き方
const city = user && user.address && user.address.city;

// ✅ オプショナルチェーン：簡潔
const city = user?.address?.city;
```

## 6. 厳密等価演算子（=== と !==）

### === と == の違い

```typescript
// == は型変換してから比較（非推奨）
console.log(5 == "5");   // true（!! 型が違うのに true）
console.log(0 == false); // true
console.log(null == undefined); // true

// === は型も含めて比較（推奨）
console.log(5 === "5");   // false（✅ 型が違う）
console.log(0 === false); // false
console.log(null === undefined); // false
```

### ✅ 常に === を使う

```typescript
// ✅ 推奨
if (value === 10) {
  console.log("10です");
}

// ❌ 非推奨
if (value == 10) {
  console.log("10です");
}
```

### !== も同様

```typescript
// ✅ 推奨
if (value !== null) {
  console.log("null ではない");
}

// ❌ 非推奨
if (value != null) {
  console.log("null ではない");
}
```

## 7. switch 文

複数の条件分岐をシンプルに書けます。

```typescript
const day = "Monday";

switch (day) {
  case "Monday":
    console.log("月曜日");
    break;
  case "Tuesday":
    console.log("火曜日");
    break;
  case "Wednesday":
    console.log("水曜日");
    break;
  default:
    console.log("その他の曜日");
}
```

### break の重要性

`break` を忘れると次の case も実行されます（フォールスルー）。

```typescript
const value = 1;

switch (value) {
  case 1:
    console.log("1");
    // break なし！
  case 2:
    console.log("2");
    break;
}
// 出力：
// 1
// 2
```

### 複数の case をまとめる

```typescript
const day = "Saturday";

switch (day) {
  case "Saturday":
  case "Sunday":
    console.log("週末");
    break;
  case "Monday":
  case "Tuesday":
  case "Wednesday":
  case "Thursday":
  case "Friday":
    console.log("平日");
    break;
  default:
    console.log("不明");
}
```

### switch vs if-else

```typescript
// switch が適している
switch (status) {
  case "pending": return "保留中";
  case "approved": return "承認済み";
  case "rejected": return "却下";
  default: return "不明";
}

// if-else が適している
if (age < 18) {
  return "未成年";
} else if (age < 65) {
  return "成人";
} else {
  return "高齢者";
}
```

## 8. for ループ

繰り返し処理を行います。

### 基本的な for ループ

```typescript
for (let i = 0; i < 5; i++) {
  console.log(i);
}
// 0, 1, 2, 3, 4
```

### 配列の走査

```typescript
const fruits = ["apple", "banana", "orange"];

for (let i = 0; i < fruits.length; i++) {
  console.log(fruits[i]);
}
```

### for...of（配列の要素を直接取得）

```typescript
const fruits = ["apple", "banana", "orange"];

for (const fruit of fruits) {
  console.log(fruit);
}
// apple, banana, orange
```

### for...in（オブジェクトのキーを取得）

```typescript
const user = { name: "Alice", age: 25, city: "Tokyo" };

for (const key in user) {
  console.log(`${key}: ${user[key]}`);
}
// name: Alice
// age: 25
// city: Tokyo
```

⚠️ 配列には `for...in` を使わない（インデックスが文字列になる）

```typescript
// ❌ 非推奨
const arr = [1, 2, 3];
for (const index in arr) {
  console.log(typeof index); // "string"（!! 数値ではない）
}

// ✅ 推奨
for (const value of arr) {
  console.log(value);
}
```

## 9. while ループ

条件が真の間、繰り返します。

```typescript
let count = 0;

while (count < 5) {
  console.log(count);
  count++;
}
// 0, 1, 2, 3, 4
```

### do-while

必ず1回は実行されます。

```typescript
let count = 0;

do {
  console.log(count);
  count++;
} while (count < 5);
// 0, 1, 2, 3, 4
```

## 10. break と continue

### break - ループを抜ける

```typescript
for (let i = 0; i < 10; i++) {
  if (i === 5) {
    break; // ループを終了
  }
  console.log(i);
}
// 0, 1, 2, 3, 4
```

### continue - 次の反復へ

```typescript
for (let i = 0; i < 5; i++) {
  if (i === 2) {
    continue; // 2 をスキップ
  }
  console.log(i);
}
// 0, 1, 3, 4
```

## ✅ ベストプラクティス

### 1. === を使う（== は使わない）

```typescript
// ✅ 推奨
if (value === 10) { }

// ❌ 非推奨
if (value == 10) { }
```

### 2. 可読性の高い条件式

```typescript
// ❌ わかりにくい
if (!!user && user.age >= 18 && user.verified === true) { }

// ✅ わかりやすい
const isAdult = user?.age >= 18;
const isVerified = user.verified;
if (user && isAdult && isVerified) { }
```

### 3. 早期リターン

```typescript
// ❌ ネストが深い
function process(user) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        return doSomething();
      }
    }
  }
  return null;
}

// ✅ 早期リターン
function process(user) {
  if (!user) return null;
  if (!user.isActive) return null;
  if (!user.hasPermission) return null;
  
  return doSomething();
}
```

### 4. for...of を優先

```typescript
// ✅ 推奨：for...of
for (const item of items) {
  console.log(item);
}

// ⚠️ 必要な場合のみインデックス使用
for (let i = 0; i < items.length; i++) {
  console.log(i, items[i]);
}
```

## 📝 練習問題

### 問題1: オプショナルチェーン

次のコードを安全に書き換えてください。

```typescript
const user = getUser(); // undefined の可能性がある

// エラーが発生する可能性
const city = user.address.city;
```

<details>
<summary>解答</summary>

```typescript
const city = user?.address?.city;
```
</details>

### 問題2: Nullish 合体演算子

デフォルト値を設定してください。`0` は有効な値として扱います。

```typescript
const config = { port: 0, timeout: null };

// port と timeout にデフォルト値を設定
const port = ___;
const timeout = ___;

console.log(port);    // 0
console.log(timeout); // 3000
```

<details>
<summary>解答</summary>

```typescript
const port = config.port ?? 3000;
const timeout = config.timeout ?? 3000;
```
</details>

## 🔗 次のステップ

次は [分割代入](./04-destructuring.md) について学びます。

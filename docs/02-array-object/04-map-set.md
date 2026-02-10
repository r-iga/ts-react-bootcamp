# Map と Set

## 📖 概要

Map と Set は ES6で追加されたコレクション型です。
オブジェクトや配列よりも効率的な操作ができる場合があります。

## 🎯 学習目標

- Map と Set の使い方を習得する
- オブジェクトとの違いを理解する
- 適切な使い分けができる

## 1. Map の基礎

### Map とは

キーと値のペアを保持するコレクション。オブジェクトと似ていますが、違いがあります。

```typescript
// 作成
const map = new Map<string, number>();

// 設定
map.set("apple", 100);
map.set("banana", 80);
map.set("orange", 120);

// 取得
console.log(map.get("apple")); // 100
console.log(map.get("grape")); // undefined

// サイズ
console.log(map.size); // 3

// 存在チェック
console.log(map.has("apple")); // true

// 削除
map.delete("banana");
console.log(map.has("banana")); // false

// 全削除
map.clear();
console.log(map.size); // 0
```

### 初期値を渡す

```typescript
const map = new Map([
  ["apple", 100],
  ["banana", 80],
  ["orange", 120]
]);
```

## 2. Map の反復

### for...of

```typescript
const map = new Map([
  ["apple", 100],
  ["banana", 80]
]);

// エントリを反復
for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}

// キーを反復
for (const key of map.keys()) {
  console.log(key);
}

// 値を反復
for (const value of map.values()) {
  console.log(value);
}
```

### forEach

```typescript
map.forEach((value, key) => {
  console.log(`${key}: ${value}`);
});
```

### 配列メソッドと組み合わせ

```typescript
// Map から配列へ
const entries = [...map];
const entries2 = Array.from(map);

// 配列メソッドを使用
const prices = [...map.values()];
const total = prices.reduce((sum, price) => sum + price, 0);
```

## 3. Map vs Object

### Map の利点

```typescript
// ✅ Map: どんなキーでもOK
const map = new Map();
map.set(1, "number key");
map.set(true, "boolean key");
map.set({ id: 1 }, "object key");

// ⚠️ Object: キーは文字列化される
const obj: Record<any, string> = {};
obj[1] = "number key";
obj[true] = "boolean key";
console.log(Object.keys(obj)); // ["1", "true"]
```

### サイズの取得

```typescript
// ✅ Map: 簡単
const map = new Map([["a", 1], ["b", 2]]);
console.log(map.size); // 2

// ⚠️ Object: プロパティを数える必要がある
const obj = { a: 1, b: 2 };
console.log(Object.keys(obj).length); // 2
```

### 順序の保証

```typescript
// ✅ Map: 挿入順序が保証される
const map = new Map([
  ["c", 3],
  ["a", 1],
  ["b", 2]
]);
console.log([...map.keys()]); // ["c", "a", "b"]

// ⚠️ Object: 順序は保証されない（整数キーは例外）
```

## 4. Set の基礎

### Set とは

重複のない値のコレクション。

```typescript
// 作成
const set = new Set<string>();

// 追加
set.add("apple");
set.add("banana");
set.add("apple"); // 重複は無視される

console.log(set.size); // 2

// 存在チェック
console.log(set.has("apple")); // true

// 削除
set.delete("banana");

// 全削除
set.clear();
```

### 初期値を渡す

```typescript
const set = new Set(["apple", "banana", "orange"]);

// 配列から重複を削除
const numbers = [1, 2, 3, 2, 1, 4];
const uniqueNumbers = [...new Set(numbers)];
console.log(uniqueNumbers); // [1, 2, 3, 4]
```

## 5. Set の反復

### for...of

```typescript
const set = new Set(["apple", "banana", "orange"]);

for (const value of set) {
  console.log(value);
}
```

### forEach

```typescript
set.forEach(value => {
  console.log(value);
});
```

### 配列メソッドと組み合わせ

```typescript
const set = new Set([1, 2, 3, 4, 5]);

// Set から配列へ
const array = [...set];

// フィルター
const evenNumbers = [...set].filter(n => n % 2 === 0);

// マップ
const doubled = [...set].map(n => n * 2);
```

## 6. Set の操作

### 和集合（Union）

```typescript
const setA = new Set([1, 2, 3]);
const setB = new Set([3, 4, 5]);

const union = new Set([...setA, ...setB]);
console.log([...union]); // [1, 2, 3, 4, 5]
```

### 積集合（Intersection）

```typescript
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);

const intersection = new Set(
  [...setA].filter(x => setB.has(x))
);
console.log([...intersection]); // [2, 3]
```

### 差集合（Difference）

```typescript
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);

const difference = new Set(
  [...setA].filter(x => !setB.has(x))
);
console.log([...difference]); // [1]
```

## 7. React での活用例

### 重複を排除

```typescript
function TagList({ tags }: { tags: string[] }) {
  const uniqueTags = [...new Set(tags)];
  
  return (
    <div>
      {uniqueTags.map(tag => (
        <span key={tag}>{tag}</span>
      ))}
    </div>
  );
}
```

### 選択状態の管理

```typescript
function ItemList() {
  const [selected, setSelected] = useState<Set<number>>(new Set());
  
  const toggleItem = (id: number) => {
    setSelected(prev => {
      const next = new Set(prev);
      if (next.has(id)) {
        next.delete(id);
      } else {
        next.add(id);
      }
      return next;
    });
  };
  
  return (
    <div>
      {items.map(item => (
        <div key={item.id}>
          <input
            type="checkbox"
            checked={selected.has(item.id)}
            onChange={() => toggleItem(item.id)}
          />
          {item.name}
        </div>
      ))}
    </div>
  );
}
```

### キャッシュ管理

```typescript
const cache = new Map<string, any>();

function fetchData(key: string) {
  if (cache.has(key)) {
    return Promise.resolve(cache.get(key));
  }
  
  return fetch(`/api/${key}`)
    .then(res => res.json())
    .then(data => {
      cache.set(key, data);
      return data;
    });
}
```

## ✅ ベストプラクティス

### 1. 適切な使い分け

```typescript
// ✅ 単純なキー・値: Object
const config = { apiUrl: "...", timeout: 3000 };

// ✅ 頻繁な追加・削除: Map
const userCache = new Map<string, User>();

// ✅ 重複排除: Set
const uniqueIds = new Set<number>();
```

### 2. 型安全に

```typescript
// ✅ ジェネリクスで型指定
const map = new Map<string, number>();
const set = new Set<string>();

// ❌ 型なし
const map = new Map();
```

### 3. イミュータブルに更新

```typescript
// ✅ 新しい Map/Set を作成
setMap(prev => new Map(prev).set(key, value));
setSet(prev => new Set(prev).add(value));
```

## 📝 練習問題

### 問題1: 配列の重複排除

```typescript
const numbers = [1, 2, 3, 2, 1, 4, 5, 4];
// 重複を排除して昇順にソート
```

<details>
<summary>解答</summary>

```typescript
const unique = [...new Set(numbers)].sort((a, b) => a - b);
// [1, 2, 3, 4, 5]
```
</details>

### 問題2: Map でグループ化

```typescript
const users = [
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 25 }
];
// 年齢でグループ化
```

<details>
<summary>解答</summary>

```typescript
const grouped = new Map<number, typeof users>();

users.forEach(user => {
  const group = grouped.get(user.age) || [];
  grouped.set(user.age, [...group, user]);
});
```
</details>

## 🔗 次のステップ

次は [TypeScript の型ユーティリティ](./05-typescript-types.md) について学びます。

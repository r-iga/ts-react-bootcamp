# イミュータブルな操作

## 📖 概要

イミュータブル（immutable）とは「不変」を意味し、元のデータを変更せずに新しいデータを作成する操作のことです。
React では State をイミュータブルに扱うことが**非常に重要**です。

## 🎯 学習目標

- イミュータブルとミュータブルの違いを理解する
- React で State をイミュータブルに更新できる
- 配列とオブジェクトのイミュータブル操作を使いこなす

## 1. なぜイミュータブルが重要か

### React での State 管理

React は State の変更を**参照の変化**で検知します。

```typescript
// ❌ ミュータブルな更新（React が検知できない）
const [items, setItems] = useState([1, 2, 3]);

const addItem = () => {
  items.push(4);           // 元の配列を変更
  setItems(items);         // 同じ参照を渡す
  // React は変更を検知できない！
};

// ✅ イミュータブルな更新（React が検知できる）
const addItem = () => {
  setItems([...items, 4]); // 新しい配列を作成
  // React は参照の変化を検知して再レンダリング
};
```

### その他のメリット

1. **予測可能**: 元のデータが変わらないので、バグが減る
2. **デバッグしやすい**: 変更の履歴を追える
3. **パフォーマンス最適化**: `React.memo` や `useMemo` が正しく動作する

## 2. 配列のイミュータブル操作

### 要素の追加

```typescript
const numbers = [1, 2, 3];

// ❌ ミュータブル
numbers.push(4);

// ✅ イミュータブル：末尾に追加
const newNumbers1 = [...numbers, 4];
console.log(newNumbers1); // [1, 2, 3, 4]

// ✅ イミュータブル：先頭に追加
const newNumbers2 = [0, ...numbers];
console.log(newNumbers2); // [0, 1, 2, 3]

// ✅ イミュータブル：途中に追加
const newNumbers3 = [
  ...numbers.slice(0, 2),  // 最初の2要素
  99,                       // 新しい要素
  ...numbers.slice(2)       // 残りの要素
];
console.log(newNumbers3); // [1, 2, 99, 3]
```

### 要素の削除

```typescript
const numbers = [1, 2, 3, 4, 5];

// ❌ ミュータブル
numbers.splice(2, 1);

// ✅ イミュータブル：インデックスで削除
const newNumbers1 = numbers.filter((_, index) => index !== 2);
console.log(newNumbers1); // [1, 2, 4, 5]

// ✅ イミュータブル：値で削除
const newNumbers2 = numbers.filter(num => num !== 3);
console.log(newNumbers2); // [1, 2, 4, 5]
```

### 要素の更新

```typescript
const users = [
  { id: 1, name: "Alice", age: 25 },
  { id: 2, name: "Bob", age: 30 },
  { id: 3, name: "Charlie", age: 35 }
];

// ❌ ミュータブル
users[1].age = 31;

// ✅ イミュータブル：特定の要素を更新
const updatedUsers = users.map(user =>
  user.id === 2
    ? { ...user, age: 31 }  // id が 2 なら age を更新
    : user                   // それ以外はそのまま
);
```

### 複数要素の更新

```typescript
const products = [
  { id: 1, name: "Apple", inStock: true },
  { id: 2, name: "Banana", inStock: true },
  { id: 3, name: "Orange", inStock: false }
];

// すべての在庫を false に
const outOfStock = products.map(product => ({
  ...product,
  inStock: false
}));
```

### ソート（イミュータブル）

```typescript
const numbers = [3, 1, 4, 1, 5];

// ❌ ミュータブル
numbers.sort();

// ✅ イミュータブル：コピーしてからソート
const sorted = [...numbers].sort((a, b) => a - b);

console.log(numbers); // [3, 1, 4, 1, 5]（元の配列は変わらない）
console.log(sorted);  // [1, 1, 3, 4, 5]
```

### 配列のコピー

```typescript
const original = [1, 2, 3];

// ❌ 参照のコピー
const copy1 = original;
copy1.push(4);
console.log(original); // [1, 2, 3, 4]（!! 変わってしまう）

// ✅ シャローコピー（1階層のみ）
const copy2 = [...original];
const copy3 = original.slice();
const copy4 = Array.from(original);

copy2.push(4);
console.log(original); // [1, 2, 3]（元は変わらない）
```

⚠️ **シャローコピーの注意点**

```typescript
const users = [
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 }
];

const copy = [...users];

// ネストしたオブジェクトは参照が共有される
copy[0].age = 26;
console.log(users[0].age); // 26（!! 元も変わる）
```

## 3. オブジェクトのイミュータブル操作

### プロパティの更新

```typescript
const user = {
  name: "Alice",
  age: 25,
  city: "Tokyo"
};

// ❌ ミュータブル
user.age = 26;

// ✅ イミュータブル：1つのプロパティを更新
const updatedUser1 = { ...user, age: 26 };

// ✅ イミュータブル：複数のプロパティを更新
const updatedUser2 = {
  ...user,
  age: 26,
  city: "Osaka"
};
```

### プロパティの追加

```typescript
const user = {
  name: "Alice",
  age: 25
};

// ✅ email プロパティを追加
const userWithEmail = {
  ...user,
  email: "alice@example.com"
};
```

### プロパティの削除

```typescript
const user = {
  id: 1,
  name: "Alice",
  password: "secret",
  email: "alice@example.com"
};

// ✅ password を除外
const { password, ...safeUser } = user;

console.log(safeUser);
// { id: 1, name: "Alice", email: "alice@example.com" }
```

### ネストしたオブジェクトの更新

```typescript
const user = {
  name: "Alice",
  address: {
    city: "Tokyo",
    zipCode: "100-0001"
  }
};

// ❌ ミュータブル
user.address.city = "Osaka";

// ✅ イミュータブル：ネストも新しいオブジェクトに
const updatedUser = {
  ...user,
  address: {
    ...user.address,
    city: "Osaka"
  }
};

console.log(user.address.city);        // "Tokyo"（元は変わらない）
console.log(updatedUser.address.city); // "Osaka"
```

### 深くネストした更新

```typescript
const state = {
  user: {
    profile: {
      contact: {
        email: "old@example.com"
      }
    }
  }
};

// ✅ 各階層を新しいオブジェクトに
const newState = {
  ...state,
  user: {
    ...state.user,
    profile: {
      ...state.user.profile,
      contact: {
        ...state.user.profile.contact,
        email: "new@example.com"
      }
    }
  }
};
```

⚠️ 深いネストは読みにくいので、後で学ぶ **Immer** などのライブラリを使うと便利です。

### オブジェクトのコピー

```typescript
const original = { name: "Alice", age: 25 };

// ❌ 参照のコピー
const copy1 = original;
copy1.age = 26;
console.log(original.age); // 26（!! 変わってしまう）

// ✅ シャローコピー
const copy2 = { ...original };
const copy3 = Object.assign({}, original);

copy2.age = 26;
console.log(original.age); // 25（元は変わらない）
```

## 4. React での実践例

### State の更新パターン

```typescript
// Todo アプリの例
type Todo = {
  id: number;
  text: string;
  completed: boolean;
};

const [todos, setTodos] = useState<Todo[]>([
  { id: 1, text: "Learn React", completed: false },
  { id: 2, text: "Build app", completed: false }
]);

// ✅ 要素の追加
const addTodo = (text: string) => {
  const newTodo: Todo = {
    id: Date.now(),
    text,
    completed: false
  };
  setTodos([...todos, newTodo]);
};

// ✅ 要素の削除
const deleteTodo = (id: number) => {
  setTodos(todos.filter(todo => todo.id !== id));
};

// ✅ 要素の更新
const toggleTodo = (id: number) => {
  setTodos(todos.map(todo =>
    todo.id === id
      ? { ...todo, completed: !todo.completed }
      : todo
  ));
};

// ✅ すべてクリア
const clearTodos = () => {
  setTodos([]);
};
```

### フォーム State の更新

```typescript
type FormData = {
  name: string;
  email: string;
  age: number;
};

const [formData, setFormData] = useState<FormData>({
  name: "",
  email: "",
  age: 0
});

// ✅ 1つのフィールドを更新
const handleNameChange = (name: string) => {
  setFormData({ ...formData, name });
};

// ✅ 汎用的な更新関数
const handleChange = (field: keyof FormData, value: any) => {
  setFormData({ ...formData, [field]: value });
};
```

## 5. Immer ライブラリ（補足）

深くネストした State の更新を直感的に書けるライブラリです。

```typescript
import { produce } from 'immer';

const state = {
  user: {
    profile: {
      contact: {
        email: "old@example.com"
      }
    }
  }
};

// ✅ Immer を使うとミュータブルな書き方ができる
const newState = produce(state, draft => {
  draft.user.profile.contact.email = "new@example.com";
});

// 元の state は変わらない（イミュータブル）
console.log(state.user.profile.contact.email);    // "old@example.com"
console.log(newState.user.profile.contact.email); // "new@example.com"
```

**Immer は React の State 更新にも使えます。**

```typescript
setUser(produce(draft => {
  draft.profile.bio = "New bio";
}));
```

## ✅ ベストプラクティス

### 1. React では常にイミュータブル

```typescript
// ❌ 絶対にダメ
state.value = newValue;
items.push(newItem);
user.age = 30;

// ✅ 必ず新しいオブジェクト/配列を作成
setState({ ...state, value: newValue });
setItems([...items, newItem]);
setUser({ ...user, age: 30 });
```

### 2. map/filter/reduce を活用

```typescript
// ✅ 関数型プログラミングのメソッドはイミュータブル
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);

// ❌ ループで配列を変更
for (let i = 0; i < numbers.length; i++) {
  numbers[i] = numbers[i] * 2;
}
```

### 3. 深いネストは避ける

```typescript
// ❌ 深すぎるネスト
const state = {
  data: {
    user: {
      profile: {
        settings: {
          theme: "dark"
        }
      }
    }
  }
};

// ✅ フラットな構造に
const state = {
  theme: "dark",
  userId: 1,
  // ...
};
```

## 🔍 よくある質問

### Q: イミュータブルだとパフォーマンスが悪い？

A: **いいえ**。Spread演算子は浅いコピーなので高速です。React の最適化にも必要です。

### Q: 配列のソートはどうすればいい？

A: コピーしてからソートしてください。

```typescript
const sorted = [...array].sort((a, b) => a - b);
```

### Q: 深いネストの更新が大変

A: **Immer** ライブラリを使うと楽になります。後で学習します。

## 📝 練習問題

### 問題1: Todo の更新

次の Todo リストの id=2 の完了状態を反転してください。

```typescript
const todos = [
  { id: 1, text: "Learn React", completed: false },
  { id: 2, text: "Build app", completed: false },
  { id: 3, text: "Deploy", completed: false }
];
```

<details>
<summary>解答</summary>

```typescript
const updatedTodos = todos.map(todo =>
  todo.id === 2
    ? { ...todo, completed: !todo.completed }
    : todo
);
```
</details>

### 問題2: ネストしたオブジェクトの更新

ユーザーの住所の都市名を更新してください。

```typescript
const user = {
  name: "Alice",
  address: {
    city: "Tokyo",
    country: "Japan"
  }
};

// city を "Osaka" に更新
```

<details>
<summary>解答</summary>

```typescript
const updatedUser = {
  ...user,
  address: {
    ...user.address,
    city: "Osaka"
  }
};
```
</details>

##🔗 次のステップ

次は [Chapter 3: DOM操作](../03-dom-manipulation/) に進みましょう。

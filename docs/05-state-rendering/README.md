# Chapter 5: State とレンダリング

## 🎯 学習目標

- React の State の概念を理解する
- useState フックの使い方を習得する
- State 変更によるレンダリングの仕組みを理解する
- State のリフトアップパターンを学ぶ

## 📚 学習トピック

### 5.1 State とは

**State の概念:**
- コンポーネントが保持する動的なデータ
- Props との違い（読み取り専用 vs 変更可能）
- State の変更がレンダリングをトリガーする

**なぜ State が必要か:**
- インタラクティブな UI
- ユーザー入力への応答
- 時間経過による変化

**State の設計原則:**
- 最小限の State
- 派生可能なものは State にしない
- 単一の情報源（Single Source of Truth）

### 5.2 useState フック

**基本的な使い方:**
```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

**useState の仕組み:**
- 配列の分割代入
- 現在の State 値
- State 更新関数
- 初期値の設定

**TypeScript での型付け:**
```tsx
const [name, setName] = useState<string>('');
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Item[]>([]);
```

**複数の State:**
```tsx
const [name, setName] = useState('');
const [age, setAge] = useState(0);
const [isActive, setIsActive] = useState(false);
```

**オブジェクトの State:**
```tsx
const [user, setUser] = useState({
  name: 'John',
  age: 25,
  email: 'john@example.com'
});
```

**配列の State:**
```tsx
const [items, setItems] = useState<string[]>([]);
```

### 5.3 State の更新

**直接更新（値の置き換え）:**
```tsx
setCount(10);
setName('Alice');
setIsActive(true);
```

**関数による更新:**
```tsx
setCount(prevCount => prevCount + 1);
setItems(prevItems => [...prevItems, newItem]);
```

**いつ関数更新を使うべきか:**
- 前の State に依存する更新
- 複数の更新が短時間に発生する場合
- イベントハンドラ内での複数回更新

**オブジェクトのイミュータブル更新:**
```tsx
// ❌ 直接変更（React が検知できない）
user.name = 'Alice';
setUser(user);

// ✅ イミュータブル更新
setUser({ ...user, name: 'Alice' });
```

**配列のイミュータブル更新:**
```tsx
// 追加
setItems([...items, newItem]);

// 削除
setItems(items.filter(item => item.id !== id));

// 更新
setItems(items.map(item => 
  item.id === id ? { ...item, completed: true } : item
));
```

**ネストしたオブジェクトの更新:**
```tsx
setUser({
  ...user,
  address: {
    ...user.address,
    city: 'Tokyo'
  }
});
```

### 5.4 再レンダリング

**再レンダリングとは:**
- コンポーネント関数の再実行
- 新しい仮想 DOM の生成
- 差分検出と DOM 更新

**再レンダリングのトリガー:**
1. 自身の State が変更された
2. 親から受け取る Props が変更された
3. 親コンポーネントが再レンダリングされた
4. Context の値が変更された

**State 更新の非同期性:**
```tsx
// ❌ すぐには反映されない
setCount(count + 1);
console.log(count); // 古い値が表示される

// ✅ 関数更新を使う
setCount(prevCount => {
  console.log(prevCount); // 最新の値
  return prevCount + 1;
});
```

**バッチ更新:**
- React は複数の State 更新をまとめる
- パフォーマンスの最適化
- React 18 での自動バッチング

**State とクロージャ:**
- イベントハンドラ内の State
- useEffect 内の State
- 古い値を参照する問題と解決策

### 5.5 フォームと State

**制御されたコンポーネント:**
```tsx
function Form() {
  const [text, setText] = useState('');
  
  return (
    <input
      value={text}
      onChange={(e) => setText(e.target.value)}
    />
  );
}
```

**複数の入力フィールド:**
```tsx
const [form, setForm] = useState({
  name: '',
  email: '',
  message: ''
});

const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  const { name, value } = e.target;
  setForm({ ...form, [name]: value });
};
```

**チェックボックスとラジオボタン:**
```tsx
const [isChecked, setIsChecked] = useState(false);
const [selected, setSelected] = useState('');
```

**フォームの送信:**
```tsx
const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault();
  // 送信処理
};
```

### 5.6 State のリフトアップ

**State のリフトアップとは:**
- 複数の子コンポーネントが同じ State を共有
- State を共通の親コンポーネントに移動
- Props でデータを渡し、コールバックで更新

**リフトアップの例:**
```tsx
function Parent() {
  const [value, setValue] = useState('');
  
  return (
    <>
      <ChildA value={value} onChange={setValue} />
      <ChildB value={value} />
    </>
  );
}

function ChildA({ value, onChange }: Props) {
  return (
    <input value={value} onChange={(e) => onChange(e.target.value)} />
  );
}

function ChildB({ value }: Props) {
  return <p>Value: {value}</p>;
}
```

**いつリフトアップするか:**
- 複数のコンポーネントが同じデータを必要とする
- 兄弟コンポーネント間でデータを同期したい
- 親が子の State を知る必要がある

**Props ドリリング問題:**
- 深いネストでの Props 受け渡しの煩雑さ
- Context や状態管理ライブラリでの解決（後の章で学習）

### 5.7 State 設計のベストプラクティス

**State を最小限にする:**
```tsx
// ❌ 派生データを State にしている
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');

// ✅ 派生データは計算する
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const fullName = `${firstName} ${lastName}`;
```

**State の正規化:**
- 重複を避ける
- ID による参照
- 更新の一貫性

**State の分割 vs 統合:**
- 関連する State はまとめる
- 独立した State は分ける
- 更新頻度を考慮

## 🔗 関連リソース

- [examples/05-state-rendering/](../../examples/05-state-rendering/) - サンプルコード
- [exercises/05-state-rendering/](../../exercises/05-state-rendering/) - 演習問題

## ⏱️ 推奨学習時間

**2週間（15-20時間）**

## ✅ チェックリスト

- [ ] useState フックを使って State を管理できる
- [ ] State を正しくイミュータブルに更新できる
- [ ] 再レンダリングのトリガーを理解している
- [ ] 関数更新のパターンを使える
- [ ] 制御されたコンポーネントでフォームを扱える
- [ ] State のリフトアップができる
- [ ] State 設計のベストプラクティスを適用できる

## 📝 次の章

[Chapter 6: 主要な Hooks](../06-hooks/)

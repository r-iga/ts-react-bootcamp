# 再レンダリングの仕組み

## 📖 概要

Reactは State や Props が変更されると、コンポーネントを再レンダリングしてUIを更新します。
この仕組みを理解することで、パフォーマンスの問題を避けられます。

## 🎯 学習目標

- 再レンダリングのタイミングを理解する
- 不要な再レンダリングを回避できる
- パフォーマンスの最適化ができる

## 1. 再レンダリングとは

### レンダリングのフロー

```
State/Props変更
    ↓
コンポーネント関数を再実行
    ↓
新しい仮想DOMを生成
    ↓
前回との差分を検出
    ↓
実際のDOMを更新
```

### 再レンダリングのトリガー

```tsx
const Counter = () => {
  const [count, setCount] = useState(0);
  
  console.log("Rendered"); // State変更ごとに実行される
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
};
```

## 2. 再レンダリングが起きるとき

### 1. State が変更されたとき

```tsx
const Component = () => {
  const [count, setCount] = useState(0);
  
  // ✅ State変更で再レンダリング
  const increment = () => setCount(count + 1);
  
  return <button onClick={increment}>Count: {count}</button>;
};
```

### 2. Props が変更されたとき

```tsx
const Child = ({ name }: { name: string }) => {
  console.log("Child rendered");
  return <p>Hello, {name}!</p>;
};

const Parent = () => {
  const [name, setName] = useState("Alice");
  
  // Parent が再レンダリング → Child も再レンダリング
  return (
    <div>
      <Child name={name} />
      <button onClick={() => setName("Bob")}>Change</button>
    </div>
  );
};
```

### 3. 親コンポーネントが再レンダリングされたとき

```tsx
const Child = () => {
  console.log("Child rendered");
  return <p>I am child</p>;
};

const Parent = () => {
  const [count, setCount] = useState(0);
  
  // Parent が再レンダリング → Child も再レンダリング（Propsが変わらなくても）
  return (
    <div>
      <Child />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
};
```

## 3. 同じ値で State を更新したとき

### 再レンダリングされない

```tsx
const Component = () => {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    setCount(0); // ✅ 同じ値なので再レンダリングされない
  };
  
  return <button onClick={handleClick}>Count: {count}</button>;
};
```

### オブジェクトの場合（参照の比較）

```tsx
const Component = () => {
  const [user, setUser] = useState({ name: "Alice" });
  
  const handleClick = () => {
    // ❌ 内容は同じでも、新しいオブジェクト = 再レンダリング
    setUser({ name: "Alice" });
  };
  
  return <button onClick={handleClick}>{user.name}</button>;
};
```

## 4. React.memo で最適化

### 子コンポーネントの再レンダリングを防ぐ

```tsx
import { memo } from 'react';

// ⚠️ memo なし：親の再レンダリングで常に再レンダリング
const Child = ({ name }: { name: string }) => {
  console.log("Child rendered");
  return <p>Hello, {name}!</p>;
};

// ✅ memo あり：Props が変わらなければ再レンダリングしない
const MemoizedChild = memo(({ name }: { name: string }) => {
  console.log("Child rendered");
  return <p>Hello, {name}!</p>;
});

const Parent = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <MemoizedChild name="Alice" />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
};
```

### memo が役立つとき

```tsx
// ✅ 重い計算をするコンポーネント
const ExpensiveComponent = memo(({ data }) => {
  const result = heavyCalculation(data);
  return <div>{result}</div>;
});

// ✅ 頻繁に再レンダリングされる親を持つ子
const Parent = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <MemoizedExpensiveComponent data={staticData} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
};
```

## 5. Props に関数やオブジェクトを渡すときの注意

### 問題：毎回新しい参照が作られる

```tsx
import { memo } from 'react';

const Child = memo(({ onClick }: { onClick: () => void }) => {
  console.log("Child rendered");
  return <button onClick={onClick}>Click</button>;
});

const Parent = () => {
  const [count, setCount] = useState(0);
  
  // ❌ 毎回新しい関数が作られる → memo が効かない
  return (
    <div>
      <Child onClick={() => console.log("clicked")} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
};
```

### 解決策：useCallback（後述）

```tsx
import { memo, useCallback } from 'react';

const Child = memo(({ onClick }: { onClick: () => void }) => {
  console.log("Child rendered");
  return <button onClick={onClick}>Click</button>;
});

const Parent = () => {
  const [count, setCount] = useState(0);
  
  // ✅ 同じ関数参照を返す
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []);
  
  return (
    <div>
      <Child onClick={handleClick} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
};
```

## 6. key によるリセット

### key が変わるとコンポーネントを再作成

```tsx
const Form = ({ userId }: { userId: number }) => {
  const [name, setName] = useState("");
  
  // userId が変わっても name は残る
  return (
    <input
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
};

const Parent = () => {
  const [userId, setUserId] = useState(1);
  
  // ✅ key を指定すると userId 変更時にフォームがリセットされる
  return (
    <div>
      <Form key={userId} userId={userId} />
      <button onClick={() => setUserId(userId + 1)}>Next User</button>
    </div>
  );
};
```

## 7. 仮想DOMと実際のDOM

### 仮想DOM

```tsx
// React内部で作られる軽量なオブジェクト
{
  type: 'div',
  props: {
    children: [
      { type: 'h1', props: { children: 'Hello' } },
      { type: 'p', props: { children: 'World' } }
    ]
  }
}
```

### 差分検出（Reconciliation）

```tsx
const Component = () => {
  const [count, setCount] = useState(0);
  
  // count が変わると：
  // 1. 新しい仮想DOMを生成
  // 2. 前回の仮想DOMと比較
  // 3. 変更された部分だけを実際のDOMに反映
  return (
    <div>
      <h1>Title</h1>           {/* 変更なし：更新されない */}
      <p>Count: {count}</p>    {/* 変更あり：<p>のテキストだけ更新 */}
    </div>
  );
};
```

## ✅ ベストプラクティス

### 1. 早まった最適化はしない

```tsx
// ❌ すべてを memo でラップする必要はない
const Component = memo(() => {
  return <div>Simple component</div>;
});

// ✅ パフォーマンス問題が出てから最適化
const Component = () => {
  return <div>Simple component</div>;
};
```

### 2. 安定した key を使う

```tsx
// ❌ インデックスを key にすると並び替えで問題
{items.map((item, index) => <Item key={index} {...item} />)}

// ✅ 安定したIDを使う
{items.map(item => <Item key={item.id} {...item} />)}
```

### 3. State は適切な場所に

```tsx
// ❌ グローバルな State で全体が再レンダリング
const App = () => {
  const [searchQuery, setSearchQuery] = useState("");
  
  return (
    <div>
      <Header />
      <SearchBox value={searchQuery} onChange={setSearchQuery} />
      <Results query={searchQuery} />
      <Footer />
    </div>
  );
};

// ✅ 必要な場所だけに State を持つ
const App = () => {
  return (
    <div>
      <Header />
      <SearchSection />  {/* この中だけで searchQuery を管理 */}
      <Footer />
    </div>
  );
};
```

## 📝 練習問題

### 問題1: 再レンダリングの理解

次のコードで「Increment」ボタンをクリックしたとき、何回「Child rendered」がログ出力されるでしょうか？

```tsx
const Child = () => {
  console.log("Child rendered");
  return <p>I am child</p>;
};

const Parent = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <Child />
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

<details>
<summary>解答</summary>

毎回1回ログ出力される。Parent が再レンダリングされると、Child も常に再レンダリングされる。
</details>

## 🔗 次のステップ

次は [Chapter 6: Hooks の活用](../06-hooks/) で useEffect や useCallback について学びます。

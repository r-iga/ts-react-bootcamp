# Chapter 6: 主要な Hooks

## 🎯 学習目標

- React の主要な組み込み Hooks を理解する
- 各 Hook の使用場面と使い方を習得する
- Hooks のルールを理解し、正しく使えるようになる
- パフォーマンス最適化の基礎を学ぶ

## 📚 学習トピック

### 6.1 Hooks のルール

**Hooks の2つの重要なルール:**
1. **トップレベルでのみ呼び出す**
   - ループ、条件分岐、ネストした関数内で呼び出さない
   - React が Hooks の呼び出し順序を保証する必要がある

2. **React 関数内でのみ呼び出す**
   - 関数コンポーネント内
   - カスタムフック内
   - 通常の JavaScript 関数内では呼び出さない

**ESLint プラグイン:**
- `eslint-plugin-react-hooks`
- ルール違反を自動検出

### 6.2 useEffect フック

**useEffect とは:**
- 副作用（Side Effects）を実行するための Hook
- 外部システムとの同期
- コンポーネントのライフサイクルに対応

**基本的な使い方:**
```tsx
import { useEffect } from 'react';

useEffect(() => {
  // 副作用の処理
  console.log('Effect ran');
});
```

**依存配列:**
```tsx
// 毎回実行
useEffect(() => {
  console.log('Runs on every render');
});

// 初回のみ実行
useEffect(() => {
  console.log('Runs only once');
}, []);

// 特定の値が変わった時のみ実行
useEffect(() => {
  console.log('Runs when count changes');
}, [count]);
```

**クリーンアップ関数:**
```tsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log('Tick');
  }, 1000);
  
  // クリーンアップ
  return () => {
    clearInterval(timer);
  };
}, []);
```

**よくある使用例:**
- データフェッチング
- DOM の直接操作
- タイマーの設定
- イベントリスナーの登録
- サブスクリプション

**useEffect の実行タイミング:**
- レンダリング後（非同期）
- ブラウザの描画後
- クリーンアップは次の Effect 実行前 or アンマウント時

**注意点:**
- 無限ループに注意
- 依存配列の適切な指定
- ESLint の exhaustive-deps ルール

### 6.3 useRef フック

**useRef とは:**
- ミュータブルな値を保持する
- 再レンダリングをトリガーしない
- レンダリング間で値を保持

**基本的な使い方:**
```tsx
import { useRef } from 'react';

const countRef = useRef(0);
countRef.current = countRef.current + 1;
```

**DOM 要素への参照:**
```tsx
const inputRef = useRef<HTMLInputElement>(null);

const focusInput = () => {
  inputRef.current?.focus();
};

return <input ref={inputRef} />;
```

**useRef の使用場面:**
- DOM 要素にアクセス
- タイマー ID の保存
- 前回の値の保持
- レンダリングと関係ない値の保存

**useRef vs useState:**
- useRef: 再レンダリング不要な値
- useState: UI に反映する値

**TypeScript での型付け:**
```tsx
const inputRef = useRef<HTMLInputElement>(null);
const divRef = useRef<HTMLDivElement>(null);
const valueRef = useRef<number>(0);
```

### 6.4 useReducer フック

**useReducer とは:**
- 複雑な State 管理のための Hook
- Redux に似たパターン
- State の更新ロジックを集約

**基本的な使い方:**
```tsx
import { useReducer } from 'react';

type State = { count: number };
type Action = { type: 'increment' } | { type: 'decrement' };

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      return state;
  }
};

const Counter = () => {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  
  return (
    <>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </>
  );
};
```

**useReducer のメリット:**
- 複雑な State 更新ロジックの整理
- 関連する State を1箇所で管理
- テストしやすい
- パフォーマンス最適化（コールバックの安定性）

**useState vs useReducer:**
- useState: シンプルな State
- useReducer: 複雑な State、複数の関連する State

**TypeScript でのパターン:**
```tsx
type Action =
  | { type: 'SET_NAME'; payload: string }
  | { type: 'SET_AGE'; payload: number }
  | { type: 'RESET' };
```

### 6.5 useContext フック

**useContext とは:**
- Context の値を読み取るための Hook
- Props ドリリングの解決
- グローバルな State 管理

**Context の作成:**
```tsx
import { createContext, useContext } from 'react';

type ThemeContextType = 'light' | 'dark';
const ThemeContext = createContext<ThemeContextType>('light');
```

**Provider での値の提供:**
```tsx
const App = () => {
  const [theme, setTheme] = useState<ThemeContextType>('light');
  
  return (
    <ThemeContext.Provider value={theme}>
      <Page />
    </ThemeContext.Provider>
  );
};
```

**useContext での値の取得:**
```tsx
const Button = () => {
  const theme = useContext(ThemeContext);
  
  return (
    <button className={theme === 'dark' ? 'dark-btn' : 'light-btn'}>
      Click me
    </button>
  );
};
```

**Context の適切な使用:**
- テーマ設定
- ユーザー認証情報
- 言語設定
- グローバルな設定

**Context の注意点:**
- 過度な使用は避ける
- パフォーマンスへの影響
- 値の分割を検討

### 6.6 useMemo フック

**useMemo とは:**
- 計算結果をメモ化する Hook
- 重い計算の最適化
- 参照の安定性

**基本的な使い方:**
```tsx
import { useMemo } from 'react';

const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);
```

**使用場面:**
- 重い計算処理
- オブジェクト・配列の参照安定化
- 子コンポーネントへの Props

**使いすぎない:**
- 軽い計算には不要
- メモ化自体にもコストがある
- プロファイリングして最適化

**具体例:**
```tsx
const sortedItems = useMemo(() => {
  return items.sort((a, b) => a.name.localeCompare(b.name));
}, [items]);

const filteredList = useMemo(() => {
  return list.filter(item => item.category === category);
}, [list, category]);
```

### 6.7 useCallback フック

**useCallback とは:**
- 関数をメモ化する Hook
- 子コンポーネントの再レンダリング防止
- 依存配列が変わらない限り同じ関数インスタンスを返す

**基本的な使い方:**
```tsx
import { useCallback } from 'react';

const handleClick = useCallback(() => {
  console.log('Clicked with value:', value);
}, [value]);
```

**使用場面:**
- React.memo された子コンポーネントへの Props
- useEffect の依存配列に含める関数
- カスタムフックから返す関数

**useMemo との違い:**
- useMemo: 計算結果をメモ化
- useCallback: 関数自体をメモ化
- `useCallback(fn, deps)` = `useMemo(() => fn, deps)`

**具体例:**
```tsx
const MemoizedChild = React.memo(Child);

const Parent = () => {
  const [count, setCount] = useState(0);
  
  // ❌ 毎回新しい関数が作られる
  const handleClick = () => {
    console.log('Clicked');
  };
  
  // ✅ 関数インスタンスが安定
  const handleClickMemo = useCallback(() => {
    console.log('Clicked');
  }, []);
  
  return <MemoizedChild onClick={handleClickMemo} />;
};
```

### 6.8 その他の組み込み Hooks

**useId:**
- ユニークな ID の生成
- アクセシビリティ用の ID

```tsx
const id = useId();
return (
  <>
    <label htmlFor={id}>Name:</label>
    <input id={id} />
  </>
);
```

**useImperativeHandle:**
- ref を介して親に公開する値のカスタマイズ
- forwardRef と併用

**useLayoutEffect:**
- DOM 変更直後、描画前に同期実行
- レイアウト測定など特殊なケースのみ

**useDebugValue:**
- React DevTools でのカスタムフックのデバッグ情報表示

## 🔗 関連リソース

- [examples/06-hooks/](../../examples/06-hooks/) - サンプルコード
- [exercises/06-hooks/](../../exercises/06-hooks/) - 演習問題

## ⏱️ 推奨学習時間

**2週間（20-25時間）**

## ✅ チェックリスト

- [ ] Hooks のルールを理解し、守れる
- [ ] useEffect で副作用を適切に扱える
- [ ] クリーンアップ関数を正しく使える
- [ ] useRef で DOM 要素にアクセスできる
- [ ] useReducer で複雑な State を管理できる
- [ ] useContext で Context の値を取得できる
- [ ] useMemo と useCallback でパフォーマンス最適化ができる
- [ ] 各 Hook の使い分けができる

## 📝 次の章

[Chapter 7: カスタムフック](../07-custom-hooks/)

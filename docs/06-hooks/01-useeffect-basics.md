# useEffect の基礎

## 📖 概要

useEffect は副作用（Side Effect）を扱うフックです。
データ取得、購読、DOM操作など、レンダリング外の処理に使用します。

## 🎯 学習目標

- useEffect の使い方を理解する
- 依存配列の役割を理解する
- クリーンアップを正しく実装できる

## 1. useEffect の基本

### 基本構文

```tsx
import { useEffect } from 'react';

function Component() {
  useEffect(() => {
    // 副作用の処理
    console.log("Effect ran");
  });
  
  return <div>Component</div>;
}
```

### 実行タイミング

```tsx
function Component() {
  console.log("1. Rendering");
  
  useEffect(() => {
    console.log("2. Effect ran");
  });
  
  return <div>Component</div>;
}

// 出力順序：
// 1. Rendering
// 2. Effect ran
```

## 2. 依存配列

### 毎回実行（依存配列なし）

```tsx
useEffect(() => {
  console.log("Runs after every render");
});
```

### 初回のみ実行（空の依存配列）

```tsx
useEffect(() => {
  console.log("Runs only once on mount");
}, []); // ✅ 空の配列
```

### 特定の値が変わったときのみ実行

```tsx
function Component() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("");
  
  useEffect(() => {
    console.log("Count changed:", count);
  }, [count]); // ✅ count が変わったときのみ
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <input value={name} onChange={(e) => setName(e.target.value)} />
    </div>
  );
}
```

## 3. よくある使用例

### データ取得

```tsx
function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    setLoading(true);
    
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]); // userId が変わったら再取得
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return <div>{user.name}</div>;
}
```

### ドキュメントタイトルの変更

```tsx
function PageTitle({ title }: { title: string }) {
  useEffect(() => {
    document.title = title;
  }, [title]);
  
  return <h1>{title}</h1>;
}
```

### ローカルストレージへの保存

```tsx
function Counter() {
  const [count, setCount] = useState(() => {
    const saved = localStorage.getItem("count");
    return saved ? parseInt(saved) : 0;
  });
  
  useEffect(() => {
    localStorage.setItem("count", String(count));
  }, [count]);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

## 4. クリーンアップ関数

### なぜ必要か

```tsx
function Timer() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    // ❌ クリーンアップなし：メモリリーク
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    
    // ここでタイマーが止まらない！
  }, []);
  
  return <div>Count: {count}</div>;
}
```

### クリーンアップの実装

```tsx
function Timer() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    
    // ✅ クリーンアップ関数を返す
    return () => {
      clearInterval(id);
      console.log("Cleanup");
    };
  }, []);
  
  return <div>Count: {count}</div>;
}
```

### イベントリスナーのクリーンアップ

```tsx
function Mouse Tracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener("mousemove", handleMouseMove);
    
    // ✅ イベントリスナーを削除
    return () => {
      window.removeEventListener("mousemove", handleMouseMove);
    };
  }, []);
  
  return <div>Position: {position.x}, {position.y}</div>;
}
```

## 5. 非同期処理

### async/await は直接使えない

```tsx
// ❌ useEffect に async を直接渡せない
useEffect(async () => {
  const data = await fetchData();
}, []);
```

### 正しい書き方

```tsx
function Component() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // ✅ 内部で async 関数を定義
    const fetchData = async () => {
      const response = await fetch("/api/data");
      const json = await response.json();
      setData(json);
    };
    
    fetchData();
  }, []);
  
  return <div>{data}</div>;
}
```

### AbortController でキャンセル

```tsx
function SearchResults({ query }: { query: string }) {
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(`/api/search?q=${query}`, {
      signal: controller.signal
    })
      .then(res => res.json())
      .then(data => setResults(data))
      .catch(err => {
        if (err.name === "AbortError") {
          console.log("Fetch aborted");
        }
      });
    
    // ✅ クリーンアップでリクエストをキャンセル
    return () => {
      controller.abort();
    };
  }, [query]);
  
  return <div>{/* results を表示 */}</div>;
}
```

## 6. よくある間違い

### 依存配列の漏れ

```tsx
function Component() {
  const [count, setCount] = useState(0);
  
  // ❌ count を使っているのに依存配列にない
  useEffect(() => {
    console.log("Count is:", count);
  }, []); // count が変わっても古い値を参照
  
  // ✅ 正しい
  useEffect(() => {
    console.log("Count is:", count);
  }, [count]);
}
```

### 無限ループ

```tsx
function Component() {
  const [data, setData] = useState([]);
  
  // ❌ 無限ループ
  useEffect(() => {
    fetch("/api/data")
      .then(res => res.json())
      .then(json => setData(json)); // State更新 → 再レンダリング → 再実行
  }); // 依存配列なし
  
  // ✅ 正しい
  useEffect(() => {
    fetch("/api/data")
      .then(res => res.json())
      .then(json => setData(json));
  }, []); // 初回のみ
}
```

### オブジェクト・配列を依存配列に

```tsx
function Component() {
  const options = { limit: 10 }; // 毎回新しいオブジェクト
  
  // ❌ options は毎回新しい参照 → 無限ループ
  useEffect(() => {
    fetchData(options);
  }, [options]);
  
  // ✅ 解決策1：値を依存配列に
  useEffect(() => {
    fetchData({ limit: 10 });
  }, []); // または [limit] のように分解
  
  // ✅ 解決策2：useMemo（後述）
  const options = useMemo(() => ({ limit: 10 }), []);
}
```

## 7. 複数の useEffect

### 関心事ごとに分ける

```tsx
function Component({ userId }: { userId: number }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  
  // ✅ ユーザー情報を取得
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]);
  
  // ✅ 投稿を取得
  useEffect(() => {
    fetch(`/api/users/${userId}/posts`)
      .then(res => res.json())
      .then(data => setPosts(data));
  }, [userId]);
  
  // ✅ タイトルを更新
  useEffect(() => {
    if (user) {
      document.title = `${user.name}'s Profile`;
    }
  }, [user]);
  
  return <div>{/* ... */}</div>;
}
```

## ✅ ベストプラクティス

### 1. 依存配列は正直に

```tsx
// ✅ ESLint の exhaustive-deps ルールを有効に
// 使用しているすべての値を依存配列に含める
useEffect(() => {
  doSomething(count, name);
}, [count, name]);
```

### 2. クリーンアップを忘れずに

```tsx
// ✅ タイマー、イベントリスナー、購読は必ずクリーンアップ
useEffect(() => {
  const id = setInterval(() => {}, 1000);
  return () => clearInterval(id);
}, []);
```

### 3. useEffect の代わりを検討

```tsx
// ⚠️ 計算だけなら useEffect は不要
const [count, setCount] = useState(0);
const [doubled, setDoubled] = useState(0);

useEffect(() => {
  setDoubled(count * 2);
}, [count]);

// ✅ 計算結果を変数に
const doubled = count * 2;
```

## 📝 練習問題

### 問題1: カウントダウンタイマー

10秒からカウントダウンするタイマーを作成してください。0になったら停止します。

<details>
<summary>解答</summary>

```tsx
function Countdown() {
  const [count, setCount] = useState(10);
  
  useEffect(() => {
    if (count === 0) return;
    
    const id = setInterval(() => {
      setCount(c => c - 1);
    }, 1000);
    
    return () => clearInterval(id);
  }, [count]);
  
  return <div>{count}</div>;
}
```
</details>

## 🔗 次のステップ

次は [useCallback と useMemo](./02-usecallback-usememo.md) について学びます。

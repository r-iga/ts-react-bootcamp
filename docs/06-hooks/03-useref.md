# useRef と DOM 操作

## 📖 概要

useRef は値を保持し、変更しても再レンダリングを引き起こさないフックです。
DOM要素への参照取得や、レンダリング間で値を保持する用途に使います。

## 🎯 学習目標

- useRef で DOM 要素にアクセスできる
- useRef で再レンダリングされない値を保持できる
- useState との違いを理解する

## 1. useRef の基本

### 基本構文

```tsx
import { useRef } from 'react';

function Component() {
  const ref = useRef(initialValue);
  
  // ref.current で値にアクセス
  console.log(ref.current);
  
  // 値を変更（再レンダリングされない）
  ref.current = newValue;
}
```

### useState との違い

```tsx
function Component() {
  const [count, setCount] = useState(0);
  const countRef = useRef(0);
  
  const handleClick = () => {
    // ❌ useState：再レンダリングされる
    setCount(count + 1);
    console.log("State:", count); // 前の値
    
    // ✅ useRef：再レンダリングされない
    countRef.current += 1;
    console.log("Ref:", countRef.current); // 最新の値
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

## 2. DOM 要素への参照

### input 要素にフォーカス

```tsx
function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useEffect(() => {
    // ✅ マウント時に自動フォーカス
    inputRef.current?.focus();
  }, []);
  
  return <input ref={inputRef} type="text" />;
}
```

### ボタンクリックでフォーカス

```tsx
function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const handleClick = () => {
    inputRef.current?.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={handleClick}>Focus Input</button>
    </div>
  );
}
```

### スクロール位置の取得・設定

```tsx
function ScrollToTop() {
  const divRef = useRef<HTMLDivElement>(null);
  
  const scrollToTop = () => {
    divRef.current?.scrollTo({ top: 0, behavior: 'smooth' });
  };
  
  return (
    <div>
      <div ref={divRef} style={{ height: '200px', overflow: 'auto' }}>
        {/* 長いコンテンツ */}
        <p>Content...</p>
        <p>Content...</p>
        {/* ... */}
      </div>
      <button onClick={scrollToTop}>Scroll to Top</button>
    </div>
  );
}
```

## 3. Canvas や Video の操作

### Canvas

```tsx
function Canvas() {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  
  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    
    const ctx = canvas.getContext('2d');
    if (!ctx) return;
    
    // 描画
    ctx.fillStyle = 'blue';
    ctx.fillRect(10, 10, 100, 100);
  }, []);
  
  return <canvas ref={canvasRef} width={200} height={200} />;
}
```

### Video の再生・一時停止

```tsx
function VideoPlayer() {
  const videoRef = useRef<HTMLVideoElement>(null);
  
  const play = () => {
    videoRef.current?.play();
  };
  
  const pause = () => {
    videoRef.current?.pause();
  };
  
  return (
    <div>
      <video ref={videoRef} src="/video.mp4" />
      <button onClick={play}>Play</button>
      <button onClick={pause}>Pause</button>
    </div>
  );
}
```

## 4. レンダリング間で値を保持

### タイマーIDの保持

```tsx
function Timer() {
  const [count, setCount] = useState(0);
  const intervalIdRef = useRef<number | null>(null);
  
  const start = () => {
    if (intervalIdRef.current) return; // 既に実行中
    
    intervalIdRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };
  
  const stop = () => {
    if (intervalIdRef.current) {
      clearInterval(intervalIdRef.current);
      intervalIdRef.current = null;
    }
  };
  
  useEffect(() => {
    // クリーンアップ
    return () => {
      if (intervalIdRef.current) {
        clearInterval(intervalIdRef.current);
      }
    };
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

### 前回の値を保持

```tsx
function UsePrevious<T>(value: T) {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

## 5. 更新カウントの追跡

### デバッグ用

```tsx
function Component({ prop }: { prop: string }) {
  const renderCount = useRef(0);
  
  useEffect(() => {
    renderCount.current += 1;
  });
  
  return (
    <div>
      <p>Prop: {prop}</p>
      <p>Rendered {renderCount.current} times</p>
    </div>
  );
}
```

## 6. useRef と useState の使い分け

### useState を使う

```tsx
// ✅ UI に反映する値
const [count, setCount] = useState(0);

return <div>Count: {count}</div>;
```

### useRef を使う

```tsx
// ✅ UI に関係ない値
const timeoutIdRef = useRef<number | null>(null);

// ✅ DOM 要素への参照
const inputRef = useRef<HTMLInputElement>(null);

// ✅ 前回の値を保持
const prevValueRef = useRef(value);
```

## 7. forwardRef - ref を子に渡す

### 基本

```tsx
import { forwardRef } from 'react';

type InputProps = {
  placeholder?: string;
};

const CustomInput = forwardRef<HTMLInputElement, InputProps>(
  function CustomInput({ placeholder }, ref) {
    return <input ref={ref} placeholder={placeholder} />;
  }
);

// 使用
function Parent() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const handleClick = () => {
    inputRef.current?.focus();
  };
  
  return (
    <div>
      <CustomInput ref={inputRef} placeholder="Enter text" />
      <button onClick={handleClick}>Focus</button>
    </div>
  );
}
```

### カスタムボタン

```tsx
type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
  variant?: 'primary' | 'secondary';
};

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  function Button({ variant = 'primary', children, ...props }, ref) {
    return (
      <button ref={ref} className={`btn btn-${variant}`} {...props}>
        {children}
      </button>
    );
  }
);

// 使用
function App() {
  const buttonRef = useRef<HTMLButtonElement>(null);
  
  useEffect(() => {
    buttonRef.current?.focus();
  }, []);
  
  return <Button ref={buttonRef}>Click me</Button>;
}
```

## 8. useImperativeHandle

### 子コンポーネントのメソッドを公開

```tsx
import { forwardRef, useImperativeHandle, useRef } from 'react';

type InputHandle = {
  focus: () => void;
  clear: () => void;
};

const CustomInput = forwardRef<InputHandle, {}>(
  function CustomInput(_props, ref) {
    const inputRef = useRef<HTMLInputElement>(null);
    
    useImperativeHandle(ref, () => ({
      focus: () => {
        inputRef.current?.focus();
      },
      clear: () => {
        if (inputRef.current) {
          inputRef.current.value = '';
        }
      }
    }));
    
    return <input ref={inputRef} />;
  }
);

// 使用
function Parent() {
  const inputRef = useRef<InputHandle>(null);
  
  return (
    <div>
      <CustomInput ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
      <button onClick={() => inputRef.current?.clear()}>Clear</button>
    </div>
  );
}
```

## ✅ ベストプラクティス

### 1. TypeScript で型を指定

```tsx
// ✅ DOM 要素
const inputRef = useRef<HTMLInputElement>(null);
const divRef = useRef<HTMLDivElement>(null);

// ✅ 値の保持
const countRef = useRef<number>(0);
const timerRef = useRef<number | null>(null);
```

### 2. null チェックを忘れずに

```tsx
// ✅ オプショナルチェーン
inputRef.current?.focus();

// ✅ null チェック
if (inputRef.current) {
  inputRef.current.value = '';
}
```

### 3. ref を Props で渡さない

```tsx
// ❌ ref を通常の Props で渡す
function Child({ inputRef }: { inputRef: RefObject<HTMLInputElement> }) {
  return <input ref={inputRef} />;
}

// ✅ forwardRef を使う
const Child = forwardRef<HTMLInputElement, {}>(
  function Child(_props, ref) {
    return <input ref={ref} />;
  }
);
```

## 📝 練習問題

### 問題1: カウントクリック

ボタンがクリックされた回数を表示するコンポーネントを作成してください。
ただし、useStateは使わず、useRefを使ってください。

<details>
<summary>解答</summary>

```tsx
function ClickCounter () {
  const countRef = useRef(0);
  const [, forceUpdate] = useState({});
  
  const handleClick = () => {
    countRef.current += 1;
    forceUpdate({}); // 再レンダリングを強制
  };
  
  return (
    <div>
      <p>Clicks: {countRef.current}</p>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}
```
</details>

## 🔗 次のステップ

次は [Chapter 7: カスタムフック](../07-custom-hooks/) でロジックの再利用について学びます。

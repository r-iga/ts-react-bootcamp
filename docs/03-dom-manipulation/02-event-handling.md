# イベント処理

## 📖 概要

ユーザーの操作に応答する方法を学びます。
クリック、入力、キーボード操作など、様々なイベントを処理できるようになります。

## 🎯 学習目標

- イベントハンドラーを実装できる
- イベントオブジェクトを理解する
- イベントの伝播を制御できる

## 1. 基本的なイベント

### click イベント

```javascript
const button = document.querySelector('#myButton');

button.addEventListener('click', () => {
  console.log('Button clicked!');
});
```

### イベントオブジェクト

```javascript
button.addEventListener('click', (event) => {
  console.log('Event type:', event.type);           // "click"
  console.log('Target element:', event.target);     // クリックされた要素
  console.log('Current target:', event.currentTarget); // リスナーがついている要素
  console.log('Mouse position:', event.clientX, event.clientY);
});
```

## 2. フォームのイベント

### submit イベント

```javascript
const form = document.querySelector('#myForm');

form.addEventListener('submit', (event) => {
  event.preventDefault(); // ページリロードを防ぐ
  
  const formData = new FormData(event.target);
  const data = Object.fromEntries(formData);
  
  console.log('Form data:', data);
});
```

### input イベント

```javascript
const input = document.querySelector('#search');

input.addEventListener('input', (event) => {
  const value = event.target.value;
  console.log('Current value:', value);
});
```

### change イベント

```javascript
const select = document.querySelector('#country');

select.addEventListener('change', (event) => {
  console.log('Selected:', event.target.value);
});
```

## 3. キーボードイベント

### keydown / keyup

```javascript
const input = document.querySelector('#input');

input.addEventListener('keydown', (event) => {
  console.log('Key pressed:', event.key);
  
  if (event.key === 'Enter') {
    console.log('Enter pressed!');
  }
  
  if (event.key === 'Escape') {
    console.log('Escape pressed!');
    input.value = ''; // Clear input
  }
});
```

### 修飾キー

```javascript
document.addEventListener('keydown', (event) => {
  if (event.ctrlKey && event.key === 's') {
    event.preventDefault(); // ブラウザの保存を防ぐ
    console.log('Ctrl+S pressed - Save');
  }
  
  if (event.shiftKey && event.key === 'Enter') {
    console.log('Shift+Enter pressed');
  }
});
```

## 4. マウスイベント

### mouseenter / mouseleave

```javascript
const box = document.querySelector('#box');

box.addEventListener('mouseenter', () => {
  box.style.backgroundColor = 'lightblue';
});

box.addEventListener('mouseleave', () => {
  box.style.backgroundColor = '';
});
```

### mousemove

```javascript
document.addEventListener('mousemove', (event) => {
  console.log('Mouse position:', event.clientX, event.clientY);
});
```

## 5. イベントの伝播

### バブリング

```javascript
// 親要素
document.querySelector('#parent').addEventListener('click', () => {
  console.log('Parent clicked');
});

// 子要素
document.querySelector('#child').addEventListener('click', () => {
  console.log('Child clicked');
});

// 子をクリックすると：
// "Child clicked"
// "Parent clicked"
```

### stopPropagation() - 伝播を止める

```javascript
document.querySelector('#child').addEventListener('click', (event) => {
  event.stopPropagation();
  console.log('Child clicked');
});

// 親のリスナーは実行されない
```

## 6. イベントデリゲーション

### 動的に追加された要素にも対応

```javascript
const list = document.querySelector('#list');

// ❌ 直接リスナーをつけると、後から追加された要素には効かない
document.querySelectorAll('li').forEach(li => {
  li.addEventListener('click', () => {
    console.log('Clicked:', li.textContent);
  });
});

// ✅ 親要素にリスナーをつける（イベントデリゲーション）
list.addEventListener('click', (event) => {
  if (event.target.tagName === 'LI') {
    console.log('Clicked:', event.target.textContent);
  }
});
```

## 7. リスナーの削除

### removeEventListener

```javascript
function handleClick() {
  console.log('Clicked!');
}

// 追加
button.addEventListener('click', handleClick);

// 削除
button.removeEventListener('click', handleClick);
```

### once オプション

```javascript
// 1回だけ実行されるリスナー
button.addEventListener('click', () => {
  console.log('This runs only once');
}, { once: true });
```

## 8. Reactとの違い

### 命令的 vs 宣言的

```javascript
// ❌ DOM操作（命令的）
const button = document.querySelector('#button');
button.addEventListener('click', () => {
  const counter = document.querySelector('#counter');
  counter.textContent = parseInt(counter.textContent) + 1;
});

// ✅ React（宣言的）
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

## ✅ ベストプラクティス

### 1. preventDefault を使う

```javascript
// ✅ フォーム送信時
form.addEventListener('submit', (event) => {
  event.preventDefault();
  // 処理...
});
```

### 2. イベントデリゲーションを活用

```javascript
// ✅ 親要素にリスナー
list.addEventListener('click', (event) => {
  if (event.target.matches('.delete-btn')) {
    // 削除処理
  }
});
```

### 3. リスナーは必要なくなったら削除

```javascript
// ✅ クリーンアップ
function setup() {
  button.addEventListener('click', handleClick);
}

function cleanup() {
  button.removeEventListener('click', handleClick);
}
```

## 🔗 次のステップ

Reactでは、これらのDOM操作を宣言的に記述できます。
[Chapter 4: React基礎](../04-react-fundamentals/) に進みましょう。

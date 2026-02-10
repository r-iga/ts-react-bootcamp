# DOM の基礎

## 📖 概要

DOM（Document Object Model）は、HTML ドキュメントをJavaScriptで操作するためのAPIです。
Reactを学ぶ前に、DOMの基本を理解することで、Reactの仕組みがより深く理解できます。

## 🎯 学習目標

- DOMの構造を理解する
- 要素の取得方法を習得する
- 要素の操作ができる

## 1. DOM とは

### DOM ツリー

HTMLはツリー構造として表現されます。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page Title</title>
  </head>
  <body>
    <h1>Hello World</h1>
    <p>This is a paragraph.</p>
  </body>
</html>
```

```
Document
  └─ html
      ├─ head
      │   └─ title
      │       └─ "Page Title"
      └─ body
          ├─ h1
          │   └─ "Hello World"
          └─ p
              └─ "This is a paragraph."
```

## 2. 要素の取得

### getElementById

```typescript
const element = document.getElementById("myId");

if (element) {
  console.log(element.textContent);
}
```

### querySelector（推奨）

CSSセレクタで要素を取得。

```typescript
// IDで取得
const element1 = document.querySelector("#myId");

// クラスで取得（最初の1つ）
const element2 = document.querySelector(".myClass");

// タグで取得
const element3 = document.querySelector("p");

// 複雑なセレクタ
const element4 = document.querySelector("div.container > p.intro");
```

### querySelectorAll

複数の要素を取得。

```typescript
const elements = document.querySelectorAll(".item");

console.log(elements.length);

// for...of で反復
for (const element of elements) {
  console.log(element.textContent);
}

// 配列メソッドを使う場合
Array.from(elements).forEach(element => {
  console.log(element.textContent);
});
```

## 3. 要素の操作

### textContent - テキストの取得・設定

```typescript
const element = document.querySelector("#message");

// 取得
console.log(element?.textContent);

// 設定
if (element) {
  element.textContent = "New message";
}
```

### innerHTML - HTML込みの取得・設定

```typescript
const element = document.querySelector("#content");

if (element) {
  // HTMLを設定
  element.innerHTML = "<p>Hello <strong>World</strong>!</p>";
  
  // 取得
  console.log(element.innerHTML);
}
```

⚠️ **XSS（クロスサイトスクリプティング）に注意**

```typescript
// ❌ 危険：ユーザー入力をそのまま innerHTML に設定
const userInput = "<script>alert('XSS')</script>";
element.innerHTML = userInput;

// ✅ 安全：textContent を使う
element.textContent = userInput;
```

### 属性の操作

```typescript
const link = document.querySelector("a");

if (link) {
  // 取得
  const href = link.getAttribute("href");
  
  // 設定
  link.setAttribute("href", "https://example.com");
  link.setAttribute("target", "_blank");
  
  // 削除
  link.removeAttribute("target");
  
  // 存在チェック
  const hasHref = link.hasAttribute("href");
}
```

### クラスの操作

```typescript
const element = document.querySelector(".box");

if (element) {
  // クラスを追加
  element.classList.add("active");
  element.classList.add("highlight", "large"); // 複数追加
  
  // クラスを削除
  element.classList.remove("active");
  
  // クラスをトグル
  element.classList.toggle("hidden");
  
  // クラスの存在チェック
  if (element.classList.contains("active")) {
    console.log("Active!");
  }
}
```

## 4. 要素の作成と追加

### 要素の作成

```typescript
// 要素を作成
const div = document.createElement("div");
div.textContent = "Hello!";
div.className = "message";

// body に追加
document.body.appendChild(div);
```

### 複数の要素を追加

```typescript
const container = document.querySelector("#container");

if (container) {
  const items = ["Apple", "Banana", "Orange"];
  
  items.forEach(item => {
    const li = document.createElement("li");
    li.textContent = item;
    container.appendChild(li);
  });
}
```

## 5. イベントハンドリング

### addEventListener

```typescript
const button = document.querySelector("#myButton");

button?.addEventListener("click", (event) => {
  console.log("Button clicked!");
  console.log(event.target);
});
```

### よく使うイベント

```typescript
// クリック
element.addEventListener("click", (event) => {});

// 入力変更
input.addEventListener("input", (event) => {
  const target = event.target as HTMLInputElement;
  console.log(target.value);
});

// フォーム送信
form.addEventListener("submit", (event) => {
  event.preventDefault(); // デフォルト動作を防ぐ
  console.log("Form submitted");
});

// キー押下
input.addEventListener("keydown", (event) => {
  if (event.key === "Enter") {
    console.log("Enter pressed");
  }
});
```

## 6. React との違い

### 命令的 vs 宣言的

```typescript
// ❌ DOM操作：命令的
const button = document.querySelector("#button");
button?.addEventListener("click", () => {
  const counter = document.querySelector("#counter");
  if (counter) {
    const count = parseInt(counter.textContent || "0");
    counter.textContent = String(count + 1);
  }
});

// ✅ React：宣言的
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

### なぜReactが便利か

1. **宣言的**: 「どう見えるべきか」を記述
2. **自動更新**: State変更で自動的にUIが更新される
3. **コンポーネント化**: 再利用可能な部品
4. **仮想DOM**: 効率的な更新

## ✅ ベストプラクティス

### 1. querySelector を使う

```typescript
// ✅ 推奨
document.querySelector("#myId");
document.querySelectorAll(".myClass");

// ⚠️ 古い方法
document.getElementById("myId");
document.getElementsByClassName("myClass");
```

### 2. null チェックを忘れずに

```typescript
// ✅ 推奨
const element = document.querySelector("#myId");
if (element) {
  element.textContent = "Hello";
}

// または
element?.classList.add("active");
```

### 3. XSS対策

```typescript
// ✅ textContent を使う
element.textContent = userInput;

// ❌ innerHTML は避ける
element.innerHTML = userInput;
```

## 🔗 次のステップ

Reactを学ぶことで、より宣言的で保守しやすいコードが書けるようになります。
[Chapter 4: React基礎](../04-react-fundamentals/) に進みましょう。

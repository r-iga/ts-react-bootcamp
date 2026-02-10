# Chapter 3: DOM操作

## 🎯 学習目標

- DOM（Document Object Model）の構造を理解する
- JavaScript で DOM を操作する方法を習得する
- イベント駆動プログラミングの基本を学ぶ
- React へスムーズに移行するための基礎知識を固める

## 📚 学習トピック

### 3.1 DOM とは

**DOM の基礎知識:**
- DOM ツリー構造
- ノードの種類（要素、テキスト、属性）
- ブラウザのレンダリングプロセス

**DOM API の種類:**
- Document API
- Element API
- Node API

### 3.2 要素の取得

**セレクタメソッド:**
- `document.getElementById()` - ID で取得
- `document.querySelector()` - CSS セレクタで 1 つ取得
- `document.querySelectorAll()` - CSS セレクタで複数取得
- `document.getElementsByClassName()` (非推奨)
- `document.getElementsByTagName()` (非推奨)

**要素の走査:**
- `element.children` - 子要素
- `element.parentElement` - 親要素
- `element.nextElementSibling` - 次の兄弟要素
- `element.previousElementSibling` - 前の兄弟要素

**TypeScript での型付け:**
- `HTMLElement` とその派生型
- `Element` vs `Node`
- 型アサーション

### 3.3 要素の操作

**コンテンツの変更:**
- `element.textContent` - テキストのみ
- `element.innerHTML` - HTML 込み（XSS に注意）
- `element.innerText` - 表示されるテキスト

**属性の操作:**
- `element.getAttribute(name)`
- `element.setAttribute(name, value)`
- `element.removeAttribute(name)`
- `element.hasAttribute(name)`
- 直接プロパティアクセス（`element.id`, `element.className`）

**クラスの操作:**
- `element.classList.add()`
- `element.classList.remove()`
- `element.classList.toggle()`
- `element.classList.contains()`

**スタイルの操作:**
- `element.style.property = value`
- CSS 変数の操作
- `getComputedStyle(element)`

### 3.4 要素の作成・追加・削除

**要素の作成:**
- `document.createElement(tagName)`
- `document.createTextNode(text)`

**要素の追加:**
- `parent.appendChild(child)` - 末尾に追加
- `parent.insertBefore(newNode, referenceNode)` - 指定位置に追加
- `element.append(...nodes)` - 複数追加可能
- `element.prepend(...nodes)` - 先頭に追加
- `element.insertAdjacentHTML(position, html)`

**要素の削除:**
- `element.remove()` - 自身を削除
- `parent.removeChild(child)` - 子要素を削除

**要素の置換:**
- `element.replaceWith(newElement)`
- `parent.replaceChild(newChild, oldChild)`

### 3.5 イベントハンドリング

**イベントリスナーの登録:**
- `element.addEventListener(type, handler, options)`
- `element.removeEventListener(type, handler)`

**主要なイベント:**
- **マウスイベント:** `click`, `dblclick`, `mousedown`, `mouseup`, `mousemove`, `mouseenter`, `mouseleave`
- **キーボードイベント:** `keydown`, `keyup`, `keypress`
- **フォームイベント:** `submit`, `change`, `input`, `focus`, `blur`
- **ドキュメントイベント:** `DOMContentLoaded`, `load`

**イベントオブジェクト:**
- `event.target` - イベント発生元
- `event.currentTarget` - リスナーが登録された要素
- `event.preventDefault()` - デフォルト動作を防ぐ
- `event.stopPropagation()` - イベント伝播を止める

**イベントの伝播:**
- キャプチャフェーズ
- ターゲットフェーズ
- バブリングフェーズ
- イベント委譲（Event Delegation）

**TypeScript での型付け:**
- `Event`, `MouseEvent`, `KeyboardEvent`
- イベントハンドラの型

### 3.6 フォームの扱い

**フォーム要素へのアクセス:**
- `form.elements` - フォーム内の要素
- `input.value` - 入力値の取得・設定

**バリデーション:**
- `input.validity` - 検証状態
- `input.checkValidity()` - 検証実行
- カスタムバリデーション

**フォーム送信:**
- `submit` イベント
- `preventDefault()` での送信制御

### 3.7 React への橋渡し

**React との違い:**
- 命令的 UI 操作 vs 宣言的 UI 記述
- 直接 DOM 操作 vs 仮想 DOM
- イベントリスナー vs React のイベントシステム

**React で活かせる知識:**
- イベントハンドリングの基本概念
- フォーム処理の理解
- DOM の構造理解

## 🔗 関連リソース

- [examples/03-dom-manipulation/](../../examples/03-dom-manipulation/) - サンプルコード
- [exercises/03-dom-manipulation/](../../exercises/03-dom-manipulation/) - 演習問題

## ⏱️ 推奨学習時間

**1週間（10-15時間）**

## ✅ チェックリスト

- [ ] querySelector で要素を取得できる
- [ ] 要素の属性、クラス、スタイルを操作できる
- [ ] 動的に要素を作成・追加・削除できる
- [ ] イベントリスナーを登録し、イベントを処理できる
- [ ] イベント伝播の仕組みを理解している
- [ ] フォームの値を取得し、バリデーションができる
- [ ] React との違いを説明できる

## 📝 次の章

[Chapter 4: React基礎](../04-react-fundamentals/)

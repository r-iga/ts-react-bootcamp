# Chapter 4: React 基礎

## 🎯 学習目標

- React の基本概念とアーキテクチャを理解する
- JSX の文法とルールを習得する
- コンポーネントの作成と組み合わせができるようになる
- React のレンダリング仕組みを理解する

## 📚 学習トピック

### 4.1 React とは

**React の特徴:**
- UI 構築のための JavaScript ライブラリ
- コンポーネントベースの開発
- 宣言的な UI 記述
- 仮想 DOM による効率的な更新
- 単方向データフロー

**React の歴史:**
- Facebook（現 Meta）による開発
- React のエコシステム

**環境構築:**
- Vite での React プロジェクト作成
- Create React App（参考程度）
- プロジェクト構造の理解

### 4.2 JSX 入門

**JSX とは:**
- JavaScript の拡張構文
- HTML に似た記法で UI を記述
- Babel/TypeScript による JavaScript への変換

**JSX の基本文法:**
```jsx
const element = <h1>Hello, World!</h1>;
```

**JSX のルール:**
- 1つのルート要素が必須
- `<React.Fragment>` (または `<>`) で複数要素をラップ
- JavaScript 式は `{}` で埋め込み
- `className` (not `class`)
- `htmlFor` (not `for`)
- キャメルケースの属性名（`onClick`, `onChange`）
- 自己クロージングタグ必須 (`<img />`, `<br />`)

**JavaScript 式の埋め込み:**
```jsx
const name = 'John';
const element = <h1>Hello, {name}!</h1>;
```

**条件付きレンダリング:**
- `if` 文（JSX の外）
- 三項演算子
- 論理 AND 演算子 (`&&`)
- 論理 OR 演算子 (`||`)

**リストのレンダリング:**
- `map()` による繰り返し
- `key` プロパティの重要性

**TypeScript での型付け:**
- `JSX.Element` 型
- `React.ReactNode` 型
- `React.ReactElement` 型

### 4.3 コンポーネント

**関数コンポーネント:**
```tsx
const Welcome = (props: { name: string }) => {
  return <h1>Hello, {props.name}!</h1>;
};
```

**コンポーネントの命名規則:**
- パスカルケース（大文字始まり）必須
- 意味のある名前

**コンポーネントの分割:**
- 単一責任の原則
- 再利用可能な粒度
- ファイル構成

**クラスコンポーネント（参考）:**
- 関数コンポーネントが主流
- レガシーコードの理解のために

### 4.4 Props（プロパティ）

**Props とは:**
- 親から子へのデータ受け渡し
- 読み取り専用（イミュータブル）
- 単方向データフロー

**Props の渡し方:**
```tsx
<Welcome name="Alice" age={25} />
```

**Props の受け取り方:**
```tsx
const Welcome = (props: { name: string; age: number }) => {
  return <p>{props.name} is {props.age} years old</p>;
};
```

**Props の分割代入:**
```tsx
const Welcome = ({ name, age }: { name: string; age: number }) => {
  return <p>{name} is {age} years old</p>;
};
```

**デフォルト値:**
```tsx
const Greeting = ({ name = 'Guest' }: { name?: string }) => {
  return <h1>Hello, {name}!</h1>;
};
```

**children プロパティ:**
```tsx
const Card = ({ children }: { children: React.ReactNode }) => {
  return <div className="card">{children}</div>;
};

// 使用例
<Card>
  <h2>Title</h2>
  <p>Content</p>
</Card>
```

**TypeScript での Props 型定義:**
- インターフェースと型エイリアス
- オプショナルプロパティ
- ジェネリック Props

### 4.5 レンダリングの仕組み

**React のレンダリングフロー:**
1. コンポーネント関数の実行
2. 仮想 DOM の生成
3. 差分検出（Reconciliation）
4. 実 DOM への反映（Commit）

**仮想 DOM とは:**
- JavaScript オブジェクトとしての UI 表現
- 差分計算による効率的な更新
- 直接 DOM 操作との比較

**再レンダリングのトリガー:**
- State の変更
- Props の変更
- 親コンポーネントの再レンダリング
- Context の変更

**Reconciliation（差分検出）:**
- key の役割
- 同じ型のコンポーネント
- 要素タイプの変更

### 4.6 StrictMode

**StrictMode とは:**
- 開発時の潜在的問題検出
- 本番環境には影響なし

**検出される問題:**
- 安全でないライフサイクルメソッド
- レガシーな文字列 ref
- 非推奨 API の使用
- 予期しない副作用

**二重レンダリング:**
- 純粋性のチェック
- 副作用の検出

**使い方:**
```tsx
import { StrictMode } from 'react';

<StrictMode>
  <App />
</StrictMode>
```

### 4.7 開発ツール

**React DevTools:**
- コンポーネントツリーの確認
- Props/State の検査
- パフォーマンスプロファイリング

**ESLint と Prettier:**
- コード品質の維持
- 一貫したスタイル

## 🔗 関連リソース

- [examples/04-react-fundamentals/](../../examples/04-react-fundamentals/) - サンプルコード
- [exercises/04-react-fundamentals/](../../exercises/04-react-fundamentals/) - 演習問題

## ⏱️ 推奨学習時間

**2週間（15-20時間）**

## ✅ チェックリスト

- [ ] Vite で React プロジェクトを作成できる
- [ ] JSX の文法を理解し、正しく記述できる
- [ ] 関数コンポーネントを作成できる
- [ ] Props を定義し、受け渡しができる
- [ ] 条件付きレンダリングとリストレンダリングができる
- [ ] key プロパティの重要性を理解している
- [ ] 仮想 DOM と Reconciliation の概念を説明できる
- [ ] StrictMode の役割を理解している

## 📝 次の章

[Chapter 5: State とレンダリング](../05-state-rendering/)

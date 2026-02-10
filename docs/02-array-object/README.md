# Chapter 2: 配列とオブジェクト

## 🎯 学習目標

- 配列とオブジェクトの各種操作メソッドを理解する
- Map/Set などのコレクション型を使いこなす
- イミュータブルな操作の重要性を理解する
- TypeScript での型定義方法を習得する

## 📚 学習トピック

### 2.1 配列メソッド

**基本操作:**
- `push`, `pop`, `shift`, `unshift`
- `splice`, `slice`
- `concat`, `join`

**高階関数:**
- `map` - 要素の変換
- `filter` - 要素の絞り込み
- `reduce` - 集約処理
- `find`, `findIndex` - 検索
- `some`, `every` - 条件判定
- `forEach` - ループ処理

**その他の便利なメソッド:**
- `sort`, `reverse`
- `flat`, `flatMap`
- `includes`, `indexOf`

**TypeScript での型付け:**
- 配列の型アノテーション (`number[]`, `Array<string>`)
- ジェネリック型
- 読み取り専用配列 (`readonly`, `ReadonlyArray`)

### 2.2 オブジェクト操作

**オブジェクトメソッド:**
- `Object.keys()` - キー一覧取得
- `Object.values()` - 値一覧取得
- `Object.entries()` - キーと値のペア取得
- `Object.assign()` - オブジェクトのマージ
- `Object.freeze()` - 不変オブジェクト

**プロパティアクセス:**
- ドット記法 vs ブラケット記法
- 動的なプロパティ名
- プロパティの存在チェック (`in`, `hasOwnProperty`)

**TypeScript での型付け:**
- オブジェクト型の定義
- インデックスシグネチャ
- `Record<K, V>` 型
- オプショナル・読み取り専用プロパティ

### 2.3 Map / Set オブジェクト

**Map:**
- `new Map()` の作成
- `set`, `get`, `has`, `delete`, `clear`
- イテレーション（`forEach`, `for...of`）
- Map vs Object の使い分け

**Set:**
- `new Set()` の作成
- `add`, `has`, `delete`, `clear`
- 重複削除のテクニック
- Set 演算（和集合、積集合、差集合）

**TypeScript での型付け:**
- `Map<K, V>`, `Set<T>` のジェネリック型

### 2.4 イミュータブルな操作

**なぜイミュータブルが重要か:**
- React での state 管理
- 予測可能なコード
- デバッグのしやすさ

**配列のイミュータブル操作:**
- 要素追加: `[...arr, newItem]`
- 要素削除: `arr.filter(item => item.id !== id)`
- 要素更新: `arr.map(item => item.id === id ? updatedItem : item)`
- コピー: `[...arr]`, `arr.slice()`

**オブジェクトのイミュータブル操作:**
- プロパティ更新: `{ ...obj, key: newValue }`
- プロパティ削除: `const { removed, ...rest } = obj`
- ネストした更新
- コピー: `{ ...obj }`

**ライブラリ:**
- Immer の紹介（後の章で詳しく）

### 2.5 TypeScript の型定義

**型エイリアスとインターフェース:**
```typescript
type User = {
  id: number;
  name: string;
  email?: string;
};

interface Product {
  id: string;
  name: string;
  price: number;
}
```

**ユーティリティ型:**
- `Partial<T>` - 全プロパティをオプショナルに
- `Required<T>` - 全プロパティを必須に
- `Readonly<T>` - 全プロパティを読み取り専用に
- `Pick<T, K>` - 特定プロパティのみ抽出
- `Omit<T, K>` - 特定プロパティを除外

## 🔗 関連リソース

- [examples/02-array-object/](../../examples/02-array-object/) - サンプルコード
- [exercises/02-array-object/](../../exercises/02-array-object/) - 演習問題

## ⏱️ 推奨学習時間

**1週間（10-15時間）**

## ✅ チェックリスト

- [ ] `map`, `filter`, `reduce` を使いこなせる
- [ ] Object.keys/values/entries を使ったオブジェクト操作ができる
- [ ] Map と Set の特性を理解し使い分けられる
- [ ] イミュータブルな配列・オブジェクト操作ができる
- [ ] TypeScript でオブジェクト型を適切に定義できる
- [ ] ユーティリティ型を活用できる

## 📝 次の章

[Chapter 3: DOM操作](../03-dom-manipulation/)

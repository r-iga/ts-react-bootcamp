# 🚀 TypeScript/React Bootcamp

2ヶ月間でJavaScript/TypeScriptの基礎からReactの実務レベルまで習得する研修プログラム

## 📚 対象者

- 他のプログラミング言語の経験がある方
- JavaScript/TypeScript、Reactを体系的に学びたい方
- 実務で使えるベストプラクティスを習得したい方

## 🎯 学習目標

- JavaScript/TypeScriptの基礎文法と型システムの理解
- Reactの仕組みとコンポーネント設計の習得
- 主要なReact Hooksの使い方とカスタムフックの作成
- Container/Presentationパターンなどの設計思想の理解
- 実務で使われるエコシステム（Router、Data Fetching、UIライブラリ）の活用

## 📖 カリキュラム構成

### Week 1-2: JavaScript/TypeScript 基礎
- [Chapter 1: JS/TS基礎](./docs/01-js-ts-basics/)
  - 変数定義、関数、制御構文
  - 分割代入、Spread演算子
  - 型の基礎（プリミティブ・オブジェクト）

- [Chapter 2: 配列とオブジェクト](./docs/02-array-object/)
  - Array/Objectメソッド
  - Map/Setオブジェクト
  - イミュータブルな操作

### Week 3: DOM操作
- [Chapter 3: DOM操作](./docs/03-dom-manipulation/)
  - DOM API
  - イベントハンドリング
  - Reactへの橋渡し

### Week 4-5: React 基礎
- [Chapter 4: React基礎](./docs/04-react-fundamentals/)
  - JSXとコンポーネント
  - レンダリングの仕組み
  - StrictMode

- [Chapter 5: Stateとレンダリング](./docs/05-state-rendering/)
  - useStateの概要
  - 再レンダリングとは
  - Stateのリフトアップ

### Week 6-7: Hooks 深堀
- [Chapter 6: 主要なHooks](./docs/06-hooks/)
  - useState、useEffect
  - useRef、useReducer、useContext
  - useMemo、useCallback

- [Chapter 7: カスタムフック](./docs/07-custom-hooks/)
  - ロジックの再利用
  - 実践的なカスタムフック作成

### Week 8: アーキテクチャと設計
- [Chapter 8: アーキテクチャ](./docs/08-architecture/)
  - Container/Presentationパターン
  - 責務の分離
  - ディレクトリ構成のベストプラクティス

### Week 9: エコシステム
- [Chapter 9: 主要なパッケージ](./docs/09-ecosystem/)
  - ルーティング（react-router）
  - データフェッチ（React Query）
  - UIライブラリ（Material-UI、Tailwind CSS）

### Week 10: 最終プロジェクト
- 学んだ内容を統合した実践的なアプリケーション開発

## 📁 ディレクトリ構成

```
ts-react-bootcamp/
├── docs/           # 📚 各章の学習資料（Markdown）
├── examples/       # 💡 サンプルコード
├── exercises/      # ✏️ ハンズオン演習
��   ├── problem/    # 演習問題
│   └── solution/   # 解答例
└── projects/       # 🚀 週次プロジェクト・最終課題
```

## 🚀 学習の進め方

1. **各章のREADMEを読む** - 学習目標とトピックを確認
2. **サンプルコードを動かす** - `examples/` のコードを実際に実行
3. **演習問題に取り組む** - `exercises/problem/` の課題を解く
4. **解答例を確認** - `exercises/solution/` で答え合わせ
5. **週次プロジェクト** - 学んだ内容を統合して小規模アプリを作成

## 💡 推奨学習時間

- **平日**: 2-3時間/日
- **週末**: 4-6時間/日
- **合計**: 約120-150時間（2ヶ月間）

## 🛠️ 環境構築

Node.js 18以上が必要です。

```bash
# Node.jsのバージョン確認
node -v

# 各プロジェクトのセットアップ
cd examples/XX-chapter-name
npm install
npm run dev
```

## 📝 ライセンス

MIT License

---

**Let's start coding! 🎉**

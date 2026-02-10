# Chapter 8: アーキテクチャ

## 🎯 学習目標

- React アプリケーションの設計パターンを理解する
- Container/Presentation パターンを習得する
- コンポーネントの責務を適切に分離できるようになる
- スケーラブルなディレクトリ構成を設計できる

## 📚 学習トピック

### 8.1 React アーキテクチャの基本概念

**なぜアーキテクチャが重要か:**
- 保守性の向上
- コードの再利用性
- チーム開発での一貫性
- テストのしやすさ
- スケーラビリティ

**設計原則:**
- 単一責任の原則（Single Responsibility Principle）
- 関心の分離（Separation of Concerns）
- DRY（Don't Repeat Yourself）
- YAGNI（You Aren't Gonna Need It）

### 8.2 Container/Presentation パターン

**パターンの概要:**
- データ取得と表示の分離
- ビジネスロジックと UI の分離
- 再利用可能なコンポーネント設計

**Container コンポーネント（Smart Components）:**
- データの取得・管理
- 状態管理
- 副作用の処理
- ビジネスロジック
- Presentation コンポーネントへの Props 受け渡し

**Presentation コンポーネント（Dumb Components）:**
- UI の表示のみ
- Props を受け取り表示
- 状態を持たない（ローカル UI 状態は可）
- 再利用可能

**実装例:**
```tsx
// ❌ 混在した実装
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div className="profile">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// ✅ Container/Presentation に分離
// Container: データ取得とロジック
function UserProfileContainer({ userId }: { userId: string }) {
  const { data: user, loading, error } = useFetch<User>(`/api/users/${userId}`);
  
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <NotFound message="User not found" />;
  
  return <UserProfilePresentation user={user} />;
}

// Presentation: 表示のみ
type UserProfilePresentationProps = {
  user: User;
};

function UserProfilePresentation({ user }: UserProfilePresentationProps) {
  return (
    <div className="profile">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

**カスタムフックとの組み合わせ:**
```tsx
// ロジックをカスタムフックに抽出
function useUserProfile(userId: string) {
  const { data, loading, error } = useFetch<User>(`/api/users/${userId}`);
  
  return { user: data, loading, error };
}

// Container がシンプルに
function UserProfileContainer({ userId }: { userId: string }) {
  const { user, loading, error } = useUserProfile(userId);
  
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <NotFound message="User not found" />;
  
  return <UserProfilePresentation user={user} />;
}
```

### 8.3 コンポーネント設計のベストプラクティス

**コンポーネントのサイズ:**
- 小さく、単一責任を持つ
- 長すぎるコンポーネントは分割
- 目安: 200-300 行以下

**Props の設計:**
- 必要最小限の Props
- Props の粒度（オブジェクト vs 個別の値）
- TypeScript での厳密な型定義
- デフォルト値の活用

**条件付きレンダリングの整理:**
```tsx
// ❌ 複雑な条件をコンポーネント内に
function Component() {
  return (
    <div>
      {loading ? (
        <Spinner />
      ) : error ? (
        <Error />
      ) : data ? (
        <Content data={data} />
      ) : (
        <Empty />
      )}
    </div>
  );
}

// ✅ Early Return でシンプルに
function Component() {
  if (loading) return <Spinner />;
  if (error) return <Error />;
  if (!data) return <Empty />;
  
  return <Content data={data} />;
}
```

**コンポーネントの合成:**
```tsx
// Composition Pattern
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>;
}

Card.Header = function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>;
};

Card.Body = function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>;
};

// 使用例
<Card>
  <Card.Header>
    <h2>Title</h2>
  </Card.Header>
  <Card.Body>
    <p>Content</p>
  </Card.Body>
</Card>
```

### 8.4 状態管理の設計

**状態の配置:**
- ローカル状態 vs グローバル状態
- 状態のリフトアップ
- 状態の正規化

**状態管理の階層:**
```
1. Component State (useState, useReducer)
   ↓ シンプルなローカル状態
   
2. Lifted State
   ↓ 複数コンポーネントで共有
   
3. Context API
   ↓ アプリ全体で共有（テーマ、認証など）
   
4. 状態管理ライブラリ (Zustand, Redux, etc.)
   ↓ 複雑なグローバル状態
```

**Context の適切な分割:**
```tsx
// ❌ 1つの巨大な Context
const AppContext = createContext({
  user: null,
  theme: 'light',
  language: 'ja',
  // ... 数十個のプロパティ
});

// ✅ 責務ごとに分割
const UserContext = createContext<User | null>(null);
const ThemeContext = createContext<Theme>('light');
const LanguageContext = createContext<Language>('ja');
```

### 8.5 ディレクトリ構成

**小規模プロジェクト:**
```
src/
├── components/       # 再利用可能なコンポーネント
│   ├── Button.tsx
│   ├── Input.tsx
│   └── Card.tsx
├── pages/           # ページコンポーネント
│   ├── Home.tsx
│   └── About.tsx
├── hooks/           # カスタムフック
│   └── useFetch.ts
├── utils/           # ユーティリティ関数
│   └── formatDate.ts
├── types/           # 型定義
│   └── index.ts
├── App.tsx
└── main.tsx
```

**中〜大規模プロジェクト（Feature-based）:**
```
src/
├── features/             # 機能ごとに分割
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── RegisterForm.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   ├── api/
│   │   │   └── authApi.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   └── index.ts
│   └── user/
│       ├── components/
│       │   ├── UserProfile.tsx
│       │   └── UserList.tsx
│       ├── hooks/
│       │   └── useUserProfile.ts
│       └── index.ts
├── shared/              # 共通コンポーネント
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   └── Button.module.css
│   │   └── Input/
│   ├── hooks/
│   ├── utils/
│   └── types/
├── layouts/             # レイアウトコンポーネント
│   ├── MainLayout.tsx
│   └── AuthLayout.tsx
├── pages/               # ページコンポーネント
│   ├── HomePage.tsx
│   └── UserPage.tsx
├── routes/              # ルーティング設定
│   └── index.tsx
├── services/            # API クライアントなど
│   └── api.ts
├── stores/              # グローバル状態管理
│   └── userStore.ts
├── App.tsx
└── main.tsx
```

**Atomic Design:**
```
src/
├── components/
│   ├── atoms/          # 最小単位（Button, Input, Label）
│   ├── molecules/      # atomsの組み合わせ（SearchBox, FormField）
│   ├── organisms/      # molecules/atomsの組み合わせ（Header, Form）
│   ├── templates/      # ページのレイアウト
│   └── pages/          # 実際のページ
```

### 8.6 ファイル命名規則

**コンポーネントファイル:**
- PascalCase: `UserProfile.tsx`
- 拡張子: `.tsx` (JSX を含む), `.ts` (含まない)

**フック:**
- camelCase: `useFetch.ts`, `useAuth.ts`

**ユーティリティ:**
- camelCase: `formatDate.ts`, `calculateTotal.ts`

**型定義:**
- `types.ts`, `index.ts`, または `User.types.ts`

**スタイル:**
- CSS Modules: `Button.module.css`
- Styled Components: `Button.styles.ts`

### 8.7 コードの整理とリファクタリング

**リファクタリングのタイミング:**
- 重複コードが3回以上現れた
- コンポーネントが200行を超えた
- Props が10個を超えた
- ネストが深すぎる（3-4階層以上）

**リファクタリングの手順:**
1. 現状の動作を確認・テストを書く
2. 小さく分割して段階的に変更
3. 各ステップで動作確認
4. 不要なコードを削除

**抽出のパターン:**
- 共通ロジック → カスタムフック
- UI の繰り返し → コンポーネント
- 定数や型 → 別ファイル
- 複雑な計算 → ユーティリティ関数

## 🔗 関連リソース

- [examples/08-architecture/](../../examples/08-architecture/) - サンプルコード
- [exercises/08-architecture/](../../exercises/08-architecture/) - 演習問題

## ⏱️ 推奨学習時間

**1週間（10-15時間）**

## ✅ チェックリスト

- [ ] Container/Presentation パターンを理解している
- [ ] コンポーネントを適切な粒度で分割できる
- [ ] Props を適切に設計できる
- [ ] 状態をどこに配置すべきか判断できる
- [ ] プロジェクトの規模に応じたディレクトリ構成を設計できる
- [ ] ファイル命名規則を統一できる
- [ ] リファクタリングのタイミングを判断できる

## 📝 次の章

[Chapter 9: エコシステム](../09-ecosystem/)

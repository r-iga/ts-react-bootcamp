# コンポーネント設計

## 📖 概要

良いコンポーネント設計は、保守性・再利用性・テストのしやすさを高めます。
適切な設計パターンを学び、スケーラブルなアプリケーションを構築しましょう。

## 🎯 学習目標

- コンポーネントの責務を理解する
- 適切な粒度でコンポーネントを分割できる
- デザインパターンを活用できる

## 1. 単一責任の原則

### 悪い例：詰め込みすぎ

```tsx
// ❌ 1つのコンポーネントに多くの責務
const UserDashboard = () => {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [followers, setFollowers] = useState([]);
  
  useEffect(() => {
    // ユーザー情報を取得
    fetch('/api/user').then(/* ... */);
  }, []);
  
  useEffect(() => {
    // 投稿を取得
    fetch('/api/posts').then(/* ... */);
  }, []);
  
  useEffect(() => {
    // フォロワーを取得
    fetch('/api/followers').then(/* ... */);
  }, []);
  
  return (
    <div>
      {/* プロフィール */}
      <div>
        <img src={user?.avatar} />
        <h1>{user?.name}</h1>
        <p>{user?.bio}</p>
      </div>
      
      {/* 投稿一覧 */}
      <div>
        <h2>Posts</h2>
        {posts.map(post => (
          <div key={post.id}>
            <h3>{post.title}</h3>
            <p>{post.content}</p>
          </div>
        ))}
      </div>
      
      {/* フォロワー一覧 */}
      <div>
        <h2>Followers</h2>
        {followers.map(follower => (
          <div key={follower.id}>
            <img src={follower.avatar} />
            <span>{follower.name}</span>
          </div>
        ))}
      </div>
    </div>
  );
};
```

### 良い例：責務を分離

```tsx
// ✅ 各コンポーネントは1つの責務
const UserProfile = ({ user }: { user: User }) => {
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
    </div>
  );
};

const PostList = ({ posts }: { posts: Post[] }) => {
  return (
    <div>
      <h2>Posts</h2>
      {posts.map(post => (
        <PostItem key={post.id} post={post} />
      ))}
    </div>
  );
};

const PostItem = ({ post }: { post: Post }) => {
  return (
    <article>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </article>
  );
};

const FollowerList = ({ followers }: { followers: User[] }) => {
  return (
    <div>
      <h2>Followers</h2>
      {followers.map(follower => (
        <FollowerItem key={follower.id} follower={follower} />
      ))}
    </div>
  );
};

const FollowerItem = ({ follower }: { follower: User }) => {
  return (
    <div>
      <img src={follower.avatar} alt={follower.name} />
      <span>{follower.name}</span>
    </div>
  );
};

// ✅ コンテナコンポーネント（データ取得）
const UserDashboard = () => {
  const { data: user } = useFetch<User>('/api/user');
  const { data: posts } = useFetch<Post[]>('/api/posts');
  const { data: followers } = useFetch<User[]>('/api/followers');
  
  if (!user || !posts || !followers) {
    return <Loading />;
  }
  
  return (
    <div>
      <UserProfile user={user} />
      <PostList posts={posts} />
      <FollowerList followers={followers} />
    </div>
  );
};
```

## 2. コンテナとプレゼンテーション

### プレゼンテーショナルコンポーネント

```tsx
// ✅ UI だけを担当
// State やロジックを持たない
// Props からデータを受け取って表示

type ProductCardProps = {
  product: Product;
  onAddToCart: (id: number) => void;
};

const ProductCard = ({ product, onAddToCart }: ProductCardProps) => {
  return (
    <div className="card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>¥{product.price.toLocaleString()}</p>
      <button onClick={() => onAddToCart(product.id)}>
        カートに追加
      </button>
    </div>
  );
};
```

### コンテナコンポーネント

```tsx
// ✅ ロジックとデータ取得を担当
// State やカスタムフックを使う
// プレゼンテーショナルコンポーネントにデータを渡す

const ProductListContainer = () => {
  const { data: products, loading } = useFetch<Product[]>('/api/products');
  const [cart, setCart] = useState<number[]>([]);
  
  const handleAddToCart = (productId: number) => {
    setCart([...cart, productId]);
    console.log('Added to cart:', productId);
  };
  
  if (loading) return <Loading />;
  if (!products) return <Error />;
  
  return (
    <div>
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={handleAddToCart}
        />
      ))}
    </div>
  );
};
```

## 3. Compound Components パターン

### 柔軟なAPI

```tsx
// ✅ コンポーネントを組み合わせて使える
const Tabs = ({ children }: { children: React.ReactNode }) => {
  const [activeTab, setActiveTab] = useState(0);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
};

const TabList = ({ children }: { children: React.ReactNode }) => {
  return <div className="tab-list">{children}</div>;
};

const Tab = ({ index, children }: { index: number; children: React.ReactNode }) => {
  const { activeTab, setActiveTab } = useTabsContext();
  
  return (
    <button
      className={activeTab === index ? 'active' : ''}
      onClick={() => setActiveTab(index)}
    >
      {children}
    </button>
  );
};

const TabPanels = ({ children }: { children: React.ReactNode }) => {
  return <div className="tab-panels">{children}</div>;
};

const TabPanel = ({ index, children }: { index: number; children: React.ReactNode }) => {
  const { activeTab } = useTabsContext();
  
  if (activeTab !== index) return null;
  
  return <div className="tab-panel">{children}</div>;
};

// 使用例
const App = () => {
  return (
    <Tabs>
      <TabList>
        <Tab index={0}>Profile</Tab>
        <Tab index={1}>Settings</Tab>
        <Tab index={2}>Notifications</Tab>
      </TabList>
      
      <TabPanels>
        <TabPanel index={0}>
          <ProfileContent />
        </TabPanel>
        <TabPanel index={1}>
          <SettingsContent />
        </TabPanel>
        <TabPanel index={2}>
          <NotificationsContent />
        </TabPanel>
      </TabPanels>
    </Tabs>
  );
};
```

## 4. コンポーネントの粒度

### 適切なサイズ

```tsx
// ⚠️ 小さすぎ：再利用されない
const UserNameFirstLetter = ({ name }: { name: string }) => {
  return <span>{name[0]}</span>;
};

// ✅ 適切：意味のある単位
const UserAvatar = ({ user }: { user: User }) => {
  return (
    <div className="avatar">
      <img src={user.avatar} alt={user.name} />
      <span>{user.name}</span>
    </div>
  );
};

// ⚠️ 大きすぎ：複数の責務
const UserDashboardWithEverything = () => {
  // プロフィール、投稿、フォロワー、設定...
};
```

### 分割の目安

```tsx
// ✅ 以下の場合は分割を検討
// 1. 100行を超える
// 2. 複数の関心事を持つ
// 3. 再利用される部分がある
// 4. テストが難しい
```

## 5. Props の設計

### 必要最小限のPropsを渡す

```tsx
// ❌ オブジェクト全体を渡す
const UserCard = ({ user }: { user: User }) => {
  return <div>{user.name}</div>; // name しか使わない
};

// ✅ 必要なものだけ
const UserCard = ({ name }: { name: string }) => {
  return <div>{name}</div>;
};

// Or: 複数の情報が必要な場合はオブジェクトで
const UserCard = ({ user }: { user: Pick<User, 'name' | 'avatar'> }) => {
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name}</span>
    </div>
  );
};
```

### boolean Props

```tsx
// ✅ is/has/should で始める
type ButtonProps = {
  isLoading?: boolean;
  isDisabled?: boolean;
  hasIcon?: boolean;
};

// ⚠️ 否定形は避ける
type ButtonProps = {
  notDisabled?: boolean; // ❌
  enabled?: boolean;     // ✅
};
```

### イベントハンドラー Props

```tsx
// ✅ on で始める
type Props = {
  onClick: () => void;
  onChange: (value: string) => void;
  onSubmit: (data: FormData) => void;
};
```

## 6. ディレクトリ構造

### 機能ベース

```
src/
  features/
    auth/
      components/
        LoginForm.tsx
        SignupForm.tsx
      hooks/
        useAuth.ts
      api/
        authApi.ts
      types/
        auth.types.ts
    
    posts/
      components/
        PostList.tsx
        PostItem.tsx
        PostForm.tsx
      hooks/
        usePosts.ts
      api/
        postsApi.ts
      types/
        post.types.ts
    
    users/
      components/
        UserProfile.tsx
        UserList.tsx
      hooks/
        useUsers.ts
      api/
        usersApi.ts
      types/
        user.types.ts
  
  shared/
    components/
      Button.tsx
      Input.tsx
      Modal.tsx
    hooks/
      useToggle.ts
      useFetch.ts
    utils/
      format.ts
      validation.ts
```

## 7. コンポーネントのファイル構成

### 1コンポーネント = 1ファイル

```tsx
// UserCard.tsx
import { User } from '../types';
import './UserCard.css';

type UserCardProps = {
  user: User;
  onFollow: (id: number) => void;
};

export function UserCard({ user, onFollow }: UserCardProps) {
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <button onClick={() => onFollow(user.id)}>Follow</button>
    </div>
  );
}
```

### index.ts でエクスポート

```tsx
// components/index.ts
export { UserCard } from './UserCard';
export { UserList } from './UserList';
export { UserProfile } from './UserProfile';

// 使用側
import { UserCard, UserList } from '@/components';
```

## ✅ ベストプラクティス

### 1. DRY（Don't Repeat Yourself）

```tsx
// ❌ 同じコードを繰り返す
const Profile = () => {
  return (
    <div>
      <div className="card">
        <img src={user.avatar} />
        <h3>{user.name}</h3>
      </div>
    </div>
  );
};

const Settings = () => {
  return (
    <div>
      <div className="card">
        <img src={user.avatar} />
        <h3>{user.name}</h3>
      </div>
    </div>
  );
};

// ✅ コンポーネントを再利用
const UserHeader = ({ user }: { user: User }) => {
  return (
    <div className="card">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
    </div>
  );
};

const Profile = () => {
  return <UserHeader user={user} />;
};

const Settings = () => {
  return <UserHeader user={user} />;
};
```

### 2. Props Drilling を避ける

```tsx
// ❌ Props を深くバケツリレー
const App = () => {
  const user = useUser();
  return <Layout user={user} />;
};

const Layout = ({ user }) => {
  return <Sidebar user={user} />;
};

const Sidebar = ({ user }) => {
  return <UserMenu user={user} />;
};

const UserMenu = ({ user }) => {
  return <div>{user.name}</div>;
};

// ✅ Context を使う（後述）
const UserContext = createContext();

const App = () => {
  const user = useUser();
  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
};

const UserMenu = () => {
  const user = useContext(UserContext);
  return <div>{user.name}</div>;
};
```

## 📝 練習問題

### 問題1: コンポーネント分割

次の巨大なコンポーネントを適切に分割してください。

```tsx
const ProductPage = () => {
  const [product, setProduct] = useState(null);
  const [reviews, setReviews] = useState([]);
  const [cart, setCart] = useState([]);
  
  // データ取得、カート操作、レビュー表示...
  
  return (
    <div>
      {/* 商品情報 */}
      {/* レビュー一覧 */}
      {/* カート */}
    </div>
  );
};
```

<details>
<summary>解答例</summary>

```tsx
// コンテナ
const ProductPage = () => {
  const { data: product } = useProduct();
  const { data: reviews } = useReviews();
  const { cart, addToCart } = useCart();
  
  if (!product) return <Loading />;
  
  return (
    <div>
      <ProductInfo product={product} onAddToCart={addToCart} />
      <ReviewList reviews={reviews} />
      <CartSummary items={cart} />
    </div>
  );
};

// プレゼンテーション
const ProductInfo = ({ product, onAddToCart }) => {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price}</p>
      <button onClick={() => onAddToCart(product.id)}>
        カートに追加
      </button>
    </div>
  );
};

const ReviewList = ({ reviews }) => {
  return (
    <div>
      {reviews.map(review => (
        <ReviewItem key={review.id} review={review} />
      ))}
    </div>
  );
};

const ReviewItem = ({ review }) => {
  return (
    <div>
      <p>{review.comment}</p>
      <span>{review.rating}</span>
    </div>
  );
};

const CartSummary = ({ items }) => {
  const total = items.reduce((sum, item) => sum + item.price, 0);
  
  return (
    <div>
      <p>Items: {items.length}</p>
      <p>Total: ¥{total}</p>
    </div>
  );
};
```
</details>

## 🔗 次のステップ

次は [Context API による状態管理](./02-context-api.md) について学びます。

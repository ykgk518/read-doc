

# 素晴らしいソフトウェアを作り上げる
ソフトウェアシステムを構築する際に正しく行わなければならないことはたくさんある。

アーキテクチャはそれらを結びつけて、成功のための基盤を提供する。

アーキテクチャとは品質特性を満たすためにソフトウェアをどう構成するか、ということに対する設計判断の集合

アーキテクチャを設計することによって以下の恩恵が受けれる。

* 大きな問題をより小さく、より管理しやすい問題へと変える
* 協働の仕方を示す
  * ソフトウェア開発は技術であると同時に人間的なコミュニケーションであること
  * ソフトウェア開発を進める上で、どのように一緒に働くかを考えるのはかなり重要
* 複雑な問題について会話するための語彙を提供
  * アーキテクチャ(要素と構造と関係性)の考えや中核語彙を使うと良い
* 機能以外の物に目を向けれる
  * コストや制約、スケジュール、リスク、チーム、品質特性に目を向けれる
* 後に起きる大きな問題を発見することができる
* 変化への柔軟な対応を可能にする

# ソフトウェアの基本構造
ソフトウェアには構造があって、構造は**要素**と別の**要素**を**関係**を築く

例えば、要素はクラス、パッケージ、レイヤー、オブジェクト、プロセス、サーバなど
関係の例は「〜を使用する」、「〜に依存する」、「〜を呼び出す」、「〜の上で実行する」 など

# 品質特性とは



# 設計のマインドセット
* 理解
  * 積極的に情報を集めて、問題を説明してもらうように働きかける
  * 要求を理解すること
  * 何が必要なのか
* 探求
  * 望み通りの品質特性を最もうまく促進させる組み合わせが見つかるまで、構造の組み合わせ・選択肢を試し続ける
  * 最適な組み合わせを見つけるには、パターン、技術、開発方法を調査しよう
* 作成
  * デザインされたコンセプトを形にすること
  * モデルを作成したり、プロトタイプ、ドキュメント、計算したりしてアーキテクチャを形をする
* 評価
  * 基準に従って設計判断が妥当かどうか判断する
  * すべてなし、すべてあり、だけではなく、一部を評価することもできる

これらを効果的には行うには**素早い**フィードバックループをもったプロセスが必要。


# 戦略を立てる

# 設計
要求の分析と要求に対するモデリングが完了したらコーディングやテストの前にやるべきアクションが存在する。

4種の設計モデル
* コンポーネント設計
* インタフェース設計
* アーキテクチャ設計
* データ設計・クラス設計



# モデルで複雑さを扱う
コードを書く前に構造を整理しよう
- どんな要素(クラス、モジュール)が必要?
- それらはどう繋がる?
- より良い名前は？
- 要求や求められている概念は何かを漏れなく明確に
  - 5W1Hを使おう
  - 制約を必ず聞く・考える
  - 「なぜ」を聞く


# パターンの土台を身につける
* レイヤーパターン
* ポートとアダプター
* パイプとフィルター
* サービス志向アーキテクチャ
* パブリッシュ・サブスクライバー
* 共有データ
* イベント駆動

# 設計判断を可視化する

# 潜在的な解決策を探る
* CRCカード
  * 目的  
    * 会話のツール
    * 責務と依存の明確化
  * 流れ
    * クラス、モジュール、コンポーネントを洗い出す
    * 責務を割り当てる
    * 協力関係を考える
    * シナリオを試す
      * カードを並べて、頭の中でシナリオを追う
      * 「このクラスは次にどうする?」と自問自答or議論
    * 改善


# SOLIDの原則(WIP)
## S - Single Responsibility Principle
## 単一責任の原則

---

## 原則の定義

**1つのコンポーネント(またはモジュール)は、1つの責任だけを持つべき**

言い換えると:
- 変更する理由は1つだけであるべき
- 1つのことだけをうまくやるべき

---

## 悪い例: 責任が多すぎるコンポーネント

```jsx
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // データ取得の責任
  useEffect(() => {
    setLoading(true);
    fetch('/api/user')
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, []);

  // バリデーションの責任
  const validateEmail = (email) => {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  };

  // 更新処理の責任
  const handleUpdate = async (newData) => {
    if (!validateEmail(newData.email)) {
      alert('Invalid email');
      return;
    }
    await fetch('/api/user', {
      method: 'PUT',
      body: JSON.stringify(newData)
    });
  };

  // 表示の責任
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <button onClick={() => handleUpdate(...)}>Update</button>
      {/* 複雑なフォームUI */}
    </div>
  );
}
```

**問題点:**
- データ取得、バリデーション、更新、表示が全部入っている
- どれか1つを変更したいとき、このコンポーネント全体を触る必要がある
- テストが難しい
- 再利用できない

---

## 良い例: 責任を分離

### 1. データ取得はカスタムフックへ

```jsx
// hooks/useUser.js
function useUser() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch('/api/user')
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, []);

  return { user, loading, error };
}
```

**責任**: ユーザーデータの取得と状態管理のみ

---

### 2. バリデーションは別モジュールへ

```jsx
// utils/validation.js
export const validateEmail = (email) => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
};
```

**責任**: バリデーションロジックのみ

---

### 3. 更新処理は別のフックへ

```jsx
// hooks/useUserUpdate.js
function useUserUpdate() {
  const updateUser = async (newData) => {
    await fetch('/api/user', {
      method: 'PUT',
      body: JSON.stringify(newData)
    });
  };

  return { updateUser };
}
```

**責任**: ユーザー更新処理のみ

---

### 4. 表示は分離されたコンポーネントへ

```jsx
// components/UserProfile.js
function UserProfile() {
  const { user, loading, error } = useUser();

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return <UserDisplay user={user} />;
}

// components/UserDisplay.js
function UserDisplay({ user }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

**責任**: 表示のみ

---

## なぜ単一責任が重要か?

### 1. 変更が楽
- メール表示を変えたい → UserDisplayだけ修正
- APIエンドポイントを変えたい → useUserだけ修正

### 2. テストしやすい
- バリデーションだけをテスト
- データ取得だけをテスト
- 表示だけをテスト

### 3. 再利用できる
- useUserは他のコンポーネントでも使える
- validateEmailは他のフォームでも使える

### 4. 理解しやすい
- 各ファイルが小さく、何をしているか明確

---

## 判断基準: 「変更する理由」を数える

コンポーネントを見て自問:

「このコンポーネントを変更する理由はいくつある?」

- デザインが変わった
- APIが変わった
- バリデーションルールが変わった
- ローディング表示を変えたい

→ **4つも理由がある = 責任が多すぎる**

理想: **変更する理由は1つだけ**

---

## 今日のミッション 🎯

今日書くコンポーネント(または既存のコンポーネント)で:

1. **責任を数える**: このコンポーネント、何個の責任を持ってる?
2. **分離できないか考える**: 
   - データ取得はカスタムフックに?
   - ビジネスロジックは別ファイルに?
   - 表示コンポーネントと分離できる?

気づいたことを `#10min-learning-practice` に投稿

**例:**
「UserListコンポーネント、データ取得と表示が混ざってた。useUsersフックに分離してみた!」

---

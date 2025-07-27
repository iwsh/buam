# 4. 技術設計

本ドキュメントは、`02_functional_requirements.md` および `03_screen_design.md` を実現するための技術的な仕様を定義する。

## 4.1. システムアーキテクチャ

本アプリケーションは、**Next.js (App Router)** を用いたフルスタック構成を採用する。

- **レンダリング戦略**: クライアントサイドレンダリング (CSR) を主体とする。各画面コンポーネントには `"use client"` を付与して実装する。
- **API**: Next.js の **Route Handlers** を使用して **REST API** を構築する。APIエンドポイントは `/app/api/` 以下に配置する。
- **インフラストラクチャ**:
    - **開発環境**: Docker Compose を使用し、ローカルでNext.jsアプリケーションをコンテナとして実行できる環境を構築する。
    - **本番環境**: 将来的に **Vercel** または **AWS** へのデプロイを想定する。

## 4.2. 技術スタック

| 領域 | 技術 | 目的・理由 |
| :--- | :--- | :--- |
| **フレームワーク** | Next.js 14+ (App Router) | フロントエンドとバックエンドを統合し、効率的な開発を実現するため。 |
| **言語** | TypeScript | 静的型付けによる開発時のエラー検知とコード品質向上のため。 |
| **UIライブラリ** | React | Next.js の標準UIライブラリ。 |
| **スタイリング** | Tailwind CSS | ユーティリティファーストなCSSフレームワークによる迅速なUI構築のため。 |
| **データベース** | TiDB Cloud (MySQL互換) | ServerlessなスケーラブルDBを利用するため。 |
| **ORM** | Prisma | 型安全なデータベースアクセスとマイグレーション管理のため。 |
| **認証** | NextAuth.js | Googleソーシャルログインやパスワード認証を容易に実装するため。 |
| **通知** | Web Push API (VAPID) | サーバーからのプッシュ通知を実現するため。 |
| **テスト** | Jest, React Testing Library, Playwright | 単体テストからE2Eテストまでをカバーするため。 |

## 4.3. ディレクトリ構成

```
buam  # repository root
├── buam/  # Next.jsアプリケーションルート
│   ├── public/
│   └── src/
│       ├── app/
│       │   ├── api/  # API Route Handlers
│       │   └── ...   # 各画面コンポーネント
│       ├── components/  # 共通UIコンポーネント
│       └── lib/  # 共通ロジック、ユーティリティ
├── docs/
│   └── design/  # 設計ドキュメント
├── terraform/  # インフラ関連
└── compose.yaml  # docker-compose yaml
```

## 4.4. データベース設計

Prismaスキーマ (`buam/prisma/schema.prisma`) を用いてモデルを定義する。詳細は `05_database_design.md` で別途定義する。

## 4.5. 認証・認可フロー

NextAuth.js を利用し、認証と認可を実装する。認証は `/api/auth/[...nextauth]` エンドポイントで処理される。

### 4.5.1. 認証 (Authentication)

- **プロバイダー**: Google (OAuth) と Credentials (メールアドレス・パスワード) の2種類を利用する。
- **セッション管理**: `jwt` 戦略を採用する。認証情報は自己完結型のJWTとしてクライアントのCookieに保存され、サーバー側ではセッション状態を保持しない。
- **フロー概要**:
    1.  クライアントはNextAuth.jsの `signIn` 関数を呼び出す。
    2.  NextAuth.jsが選択されたプロバイダーに応じた認証処理を実行する。
    3.  認証成功後、ユーザー情報を含むJWTが生成され、HttpOnly Cookieとしてクライアントに設定される。

### 4.5.2. 認可 (Authorization)

- **ロールベースアクセス制御 (RBAC)**: ユーザーの役割 (Admin, User) に基づいてアクセスを制御する。
- **実装方針**:
    1.  `User` モデルに `role` フィールドを持たせる (詳細は `05_database_design.md` を参照)。
    2.  NextAuth.js の `callbacks` (`jwt` と `session`) を利用して、JWTペイロードにユーザーの `id` と `role` を含める。
    3.  APIルートやサーバーコンポーネントでは、リクエストからJWTを検証し、ペイロード内のロール情報に基づいてアクセス権限を検証する。
    4.  クライアントコンポーネントでは、`useSession` フックから取得したセッション情報（JWTから復元されたもの）に基づき、UIの表示を制御する（例: 管理者メニューの表示/非表示）。

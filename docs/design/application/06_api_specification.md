# 06. API仕様

## 6.1. 認証・認可

本アプリケーションの認証・認可は **NextAuth.js** を利用して実装する。

### 6.1.1. 認証エンドポイント
認証関連の処理は、NextAuth.jsが提供する標準のエンドポイント (`/api/auth/[...nextauth]`) で行われる。
クライアントは、NextAuth.jsのクライアントライブラリ (`signIn`, `signOut` 等) を通じてこれらのエンドポイントと通信する。

### 6.1.2. 認証・認可フロー
1.  **認証**: ユーザーがログイン操作を行うと、NextAuth.jsが認証処理を行い、成功時にJWTを生成してHttpOnly Cookieに保存する。
2.  **APIリクエスト**: クライアントからのAPIリクエスト時、Cookieに含まれるJWTが自動的にサーバーへ送信される。
3.  **認可**: サーバーサイド（API Route Handlers）では、受信したJWTを検証し、そのペイロードに含まれるユーザーIDやロール情報 (`role`) を基に、リクエストされた操作の実行可否を判断する。

## 6.2. エンドポイント一覧

| カテゴリ | エンドポイント | メソッド | 説明 | 認証 | 認可 | 利用画面 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **ユーザー管理** | `/api/users` | `GET` | 登録済みユーザーの一覧を取得する。 | 必須 | Admin | SCR-201 ユーザー一覧画面 |
| | `/api/users` | `POST` | 新規ユーザーを招待・登録する。 | 必須 | Admin | SCR-202 ユーザー招待画面 |
| | `/api/users/[id]` | `GET` | 特定のユーザー情報を取得する。 | 必須 | 一般/Admin | SCR-104 設定画面 (自身の情報), SCR-201 ユーザー一覧画面 (Adminが他ユーザー) |
| | `/api/users/[id]` | `PUT` | 特定のユーザー情報を更新する。 | 必須 | 一般/Admin | SCR-104 設定画面 (自身の情報), SCR-201 ユーザー一覧画面 (Adminが他ユーザー) |
| | `/api/users/[id]` | `DELETE` | 特定のユーザーを削除する。 | 必須 | Admin | SCR-201 ユーザー一覧画面 |
| **車両管理** | `/api/vehicles` | `GET` | ユーザーが登録した車両の一覧を取得する。 | 必須 | 一般/Admin | SCR-104 設定画面 |
| | `/api/vehicles` | `POST` | 新しい車両を登録する。 | 必須 | 一般/Admin | SCR-104 設定画面 |
| | `/api/vehicles/[id]` | `GET` | 特定の車両情報を取得する。 | 必須 | 一般/Admin | SCR-104 設定画面 |
| | `/api/vehicles/[id]` | `PUT` | 特定の車両情報を更新する。 | 必須 | 一般/Admin | SCR-104 設定画面 |
| | `/api/vehicles/[id]` | `DELETE` | 特定の車両を削除する。 | 必須 | 一般/Admin | SCR-104 設定画面 |
| **走行記録管理** | `/api/mileage-logs` | `GET` | 特定の車両の走行記録一覧を取得する。ダッシュボードの集計データとしても利用される。 | 必須 | 一般/Admin | SCR-101 ダッシュボード画面, SCR-102 走行履歴一覧画面 |
| | `/api/mileage-logs` | `POST` | 新しい走行記録を登録する。 | 必須 | 一般/Admin | SCR-103 走行データ入力画面 |
| | `/api/mileage-logs/[id]` | `GET` | 特定の走行記録を取得する。 | 必須 | 一般/Admin | SCR-102 走行履歴一覧画面 |
| | `/api/mileage-logs/[id]` | `PUT` | 特定の走行記録を更新する。 | 必須 | 一般/Admin | SCR-102 走行履歴一覧画面 |
| | `/api/mileage-logs/[id]` | `DELETE` | 特定の走行記録を削除する。 | 必須 | 一般/Admin | SCR-102 走行履歴一覧画面 |
| **レポート** | `/api/reports/mileage-csv` | `GET` | 走行履歴データをCSV形式で出力する。 | 必須 | 一般/Admin | SCR-102 走行履歴一覧画面 |
| **設定** | `/api/settings/apportionment-period` | `GET` | 按分期間設定を取得する。 | 必須 | 一般/Admin | SCR-104 設定画面 |
| | `/api/settings/apportionment-period` | `PUT` | 按分期間設定を更新する。 | 必須 | 一般/Admin | SCR-104 設定画面 |
| | `/api/settings/notifications` | `GET` | プッシュ通知設定を取得する。 | 必須 | 一般/Admin | SCR-104 設定画面 |
| | `/api/settings/notifications` | `PUT` | プッシュ通知設定を更新する。 | 必須 | 一般/Admin | SCR-104 設定画面 |

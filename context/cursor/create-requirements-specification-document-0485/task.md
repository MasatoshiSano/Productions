# マッチングサイトアプリ タスク一覧

## 📋 概要

Django + Bootstrap 5を使用したマッチングサイトアプリケーションを実装する。
ユーザー認証、プロフィール管理、マッチング機能、リアルタイムメッセージング、管理機能を含む。

## 🎯 確定済み仕様

### 技術仕様（最新版）
- **Python**: 3.12 (最新版)
- **Django**: 5.0 (最新版)
- **データベース**: PostgreSQL 16 (最新版)
- **Redis**: 7.x (最新版)
- **位置情報**: 郵便番号API（HeartRails Geo API等）による距離計算（50km以内）
- **年齢認証**: 生年月日入力による自己申告
- **画像処理**: 800x800px、最大5MB、JPEG/PNG対応、ローカルストレージ
- **メール送信**: SendGrid SMTP
- **リアルタイム通信**: Redis + Django Channels
- **デプロイ**: Docker対応
- **デザイン**: PC・モバイル両対応、マッチングサイト適応カラー

### セキュリティ・レート制限
- **HTTPS**: 開発環境では不要
- **レート制限**: 
  - メッセージ送信: 1分間に10件まで
  - いいね送信: 1日100件まで
  - 登録試行: 1IPから1時間に5回まで
- **CSP**: 基本的なContent Security Policy設定

### プロフィール項目
- **必須項目**: 名前、年齢、性別、都道府県、郵便番号、プロフィール写真
- **任意項目**: 職業、自己紹介

### マッチング条件
- **年齢範囲**: ユーザー設定可能（制限なし）
- **距離**: 50km以内（郵便番号ベース）

### 通知機能
- **メール通知**: 即座送信（SendGrid）
- **ブラウザ通知**: 即座表示
- **通知内容**: マッチング成立、メッセージ受信

### 運用・保守
- **データ保持**: 退会ユーザーのデータは即時削除
- **ヘルプ・FAQ**: ユーザーサポートページを含む
- **統計機能**: 基本的な利用統計（ユーザー数、マッチング数等）

## ✅ タスクリスト

### フェーズ1: プロジェクトセットアップとユーザー認証

#### 1. **Djangoプロジェクト初期セットアップ**
- 概要: Django プロジェクトの基本構成とアプリケーション構造を作成
- 完了条件:
  - [ ] Django プロジェクト作成 (`matching_site`)
  - [ ] アプリケーション作成 (`accounts`, `matching`, `messaging`, `core`)
  - [ ] `settings.py` 設定完了（Bootstrap 5, Redis, Channels, 静的ファイル設定）
  - [ ] `requirements.txt` 作成完了（下記参照）
  - [ ] `Dockerfile` および `docker-compose.yml` 作成
- 動作確認:
  - [ ] `python manage.py runserver` で起動確認
  - [ ] Bootstrap 5 の読み込み確認
  - [ ] Redis接続確認
- メモ: 郵便番号API（HeartRails Geo API）設定も含める

**requirements.txt 内容:**
```
Django==5.0.*
channels==4.0.*
channels-redis==4.2.*
redis==5.0.*
Pillow==10.0.*
requests==2.31.*
sendgrid==6.10.*
django-ratelimit==4.1.*
psycopg2-binary==2.9.*
python-decouple==3.8
```

#### 2. **カスタムUserモデルの作成**
- 概要: AbstractUserを継承したカスタムUserモデルを作成
- 完了条件:
  - [ ] `accounts/models.py` にCustomUserモデル作成
  - [ ] `AUTH_USER_MODEL` 設定完了
  - [ ] 生年月日フィールド追加（年齢計算用）
  - [ ] 初期マイグレーション実行完了
- 動作確認:
  - [ ] `python manage.py test accounts` で通過
  - [ ] 管理画面でユーザー作成・編集確認
  - [ ] 18歳以上バリデーション確認
- メモ: 生年月日から年齢を自動計算するプロパティメソッド追加

#### 3. **ユーザー登録機能の実装**
- 概要: ユーザー登録フォームとビューを作成
- 完了条件:
  - [ ] `accounts/forms.py` に登録フォーム作成
  - [ ] `accounts/views.py` に登録ビュー作成
  - [ ] 生年月日による18歳以上バリデーション実装
  - [ ] 登録確認メール送信機能実装
- 動作確認:
  - [ ] `/accounts/signup/` で登録フォーム表示
  - [ ] 18歳未満の登録拒否確認
  - [ ] 確認メール送信確認
- メモ: django-allauthまたはDjango標準機能を使用

#### 4. **ログイン・ログアウト機能の実装**
- 概要: Django標準認証を使用したログイン・ログアウト機能
- 完了条件:
  - [ ] `accounts/views.py` にログイン・ログアウトビュー作成
  - [ ] ログイン状態保持機能実装
  - [ ] パスワード忘れ機能実装
  - [ ] ログイン後プロフィール作成強制リダイレクト
- 動作確認:
  - [ ] `/accounts/login/` でログイン確認
  - [ ] `/accounts/logout/` でログアウト確認
  - [ ] パスワードリセット機能確認
- メモ: CSRFトークンの適切な設定を確認

#### 5. **認証機能のテスト作成**
- 概要: 認証機能に対する単体テストを作成
- 完了条件:
  - [ ] `accounts/tests/test_models.py` 作成
  - [ ] `accounts/tests/test_views.py` 作成
  - [ ] `accounts/tests/test_forms.py` 作成
  - [ ] 年齢バリデーションテスト作成
- 動作確認:
  - [ ] `python manage.py test accounts` で全テストパス
  - [ ] カバレッジ80%以上確認
- メモ: TestCaseとClientを使用したセッションテスト実装

### フェーズ2: プロフィール管理機能

#### 6. **Profileモデルの作成**
- 概要: ユーザープロフィール情報を管理するモデル作成
- 完了条件:
  - [ ] `accounts/models.py` にProfileモデル作成
  - [ ] Userとの1対1関係設定
  - [ ] 必須フィールド: 性別、都道府県、郵便番号、プロフィール写真
  - [ ] 任意フィールド: 職業、自己紹介（500文字制限）
  - [ ] 写真アップロード機能実装（最大6枚、800x800px、最大5MB）
- 動作確認:
  - [ ] `python manage.py test accounts` で通過
  - [ ] 管理画面でプロフィール作成・編集確認
  - [ ] 画像リサイズ確認
- メモ: Pillowを使用した画像処理・リサイズ機能を含める

#### 7. **郵便番号から座標変換機能の実装**
- 概要: 郵便番号から緯度経度を取得し、距離計算を可能にする
- 完了条件:
  - [ ] HeartRails Geo API連携実装
  - [ ] 距離計算機能実装（50km以内検索）
  - [ ] `core/utils.py` に距離計算関数作成
  - [ ] プロフィール保存時に座標自動取得
- 動作確認:
  - [ ] 郵便番号入力で座標取得確認
  - [ ] 距離計算の精度確認
  - [ ] 50km以内ユーザー検索確認
- メモ: HeartRails Geo API（無料）を使用
  - API URL: `https://geoapi.heartrails.com/api/json?method=searchByPostal&postal={郵便番号}`
  - レスポンス: `{"response": {"location": [{"x": "経度", "y": "緯度"}]}}`

#### 8. **プロフィール作成・編集フォームの実装**
- 概要: プロフィール作成・編集用のフォームクラス作成
- 完了条件:
  - [ ] `accounts/forms.py` にProfileFormクラス作成
  - [ ] 必須項目バリデーション実装
  - [ ] 写真アップロード機能実装（6枚まで）
  - [ ] 郵便番号形式バリデーション実装
- 動作確認:
  - [ ] プロフィール作成フォーム表示確認
  - [ ] 画像アップロード動作確認
  - [ ] バリデーションエラー表示確認
- メモ: ModelFormを使用し、Bootstrap 5でスタイリング

#### 9. **プロフィール表示・編集ビューの実装**
- 概要: プロフィール表示・編集用のビュークラス作成
- 完了条件:
  - [ ] `accounts/views.py` にProfileViewクラス作成
  - [ ] 認証必須デコレータ適用
  - [ ] プロフィール未作成時の作成強制
  - [ ] 他ユーザーのプロフィール閲覧機能
- 動作確認:
  - [ ] `/profile/` でプロフィール表示確認
  - [ ] プロフィール編集機能確認
  - [ ] 未作成時のリダイレクト確認
- メモ: LoginRequiredMixinを使用

### フェーズ3: マッチング機能

#### 10. **マッチング関連モデルの作成**
- 概要: Like, Match, Blockモデルを作成
- 完了条件:
  - [ ] `matching/models.py` にLike, Match, Blockモデル作成
  - [ ] 適切なリレーション設定
  - [ ] 重複防止制約設定
  - [ ] インデックス設定（検索高速化）
- 動作確認:
  - [ ] `python manage.py test matching` で通過
  - [ ] 管理画面でモデル確認
- メモ: unique_togetherを使用して重複防止

#### 11. **ユーザー発見機能の実装**
- 概要: マッチング候補ユーザーを表示する機能
- 完了条件:
  - [ ] `matching/views.py` にDiscoveryViewクラス作成
  - [ ] 郵便番号ベース50km以内フィルタリング実装
  - [ ] 年齢範囲フィルタリング実装（ユーザー設定可能）
  - [ ] 既にいいね/パス/ブロックしたユーザーの除外
- 動作確認:
  - [ ] `/discover/` でユーザー候補表示確認
  - [ ] 距離・年齢フィルタリング確認
  - [ ] カード形式UI確認
- メモ: Bootstrap 5カードコンポーネントを使用

#### 12. **いいね・パス機能の実装**
- 概要: ユーザーへのいいね・パス機能とマッチング判定
- 完了条件:
  - [ ] `matching/views.py` にLikeView, PassView作成
  - [ ] Ajax対応（ページリロードなし）
  - [ ] マッチング成立判定ロジック実装
  - [ ] マッチング成立時の即座通知機能
- 動作確認:
  - [ ] いいね・パスボタン動作確認
  - [ ] マッチング成立通知確認
  - [ ] Ajax通信確認
- メモ: vanilla JavaScriptを使用

#### 13. **マッチング一覧機能の実装**
- 概要: マッチしたユーザーの一覧表示機能
- 完了条件:
  - [ ] `matching/views.py` にMatchListViewクラス作成
  - [ ] 最新メッセージプレビュー表示
  - [ ] 未読メッセージ表示
  - [ ] メッセージ画面へのリンク
- 動作確認:
  - [ ] `/matches/` でマッチング一覧表示確認
  - [ ] メッセージプレビュー確認
- メモ: select_related, prefetch_relatedでクエリ最適化

### フェーズ4: メッセージング機能

#### 14. **メッセージモデルの作成**
- 概要: メッセージデータを管理するモデル作成
- 完了条件:
  - [ ] `messaging/models.py` にMessageモデル作成
  - [ ] 送信者・受信者のリレーション設定
  - [ ] 既読・未読状態管理
  - [ ] メッセージタイプ（テキスト/画像）対応
- 動作確認:
  - [ ] `python manage.py test messaging` で通過
  - [ ] 管理画面でメッセージ確認
- メモ: created_atにインデックス設定

#### 15. **Redis + Django Channels設定**
- 概要: リアルタイムメッセージング用のChannels設定
- 完了条件:
  - [ ] `settings.py` にChannels設定追加
  - [ ] Redis接続設定
  - [ ] `asgi.py` 設定完了
  - [ ] `routing.py` 作成
- 動作確認:
  - [ ] Redis接続確認
  - [ ] Channels起動確認
  - [ ] WebSocket接続テスト
- メモ: 本番環境のRedis設定も考慮

#### 16. **WebSocketコンシューマー実装**
- 概要: リアルタイムメッセージング用のWebSocketConsumer
- 完了条件:
  - [ ] `messaging/consumers.py` にChatConsumer作成
  - [ ] メッセージ送受信機能実装
  - [ ] 認証チェック実装
  - [ ] チャンネルグループ管理
- 動作確認:
  - [ ] WebSocket接続確認
  - [ ] リアルタイムメッセージ送受信確認
  - [ ] 複数ユーザーでの同時接続確認
- メモ: 認証済みユーザーのみ接続許可

#### 17. **メッセージ送信・表示機能の実装**
- 概要: メッセージ送信フォームとチャット画面
- 完了条件:
  - [ ] `messaging/views.py` にChatViewクラス作成
  - [ ] メッセージ送信API実装
  - [ ] 既読・未読管理機能
  - [ ] マッチしたユーザー間のみアクセス制限
- 動作確認:
  - [ ] チャット画面表示確認
  - [ ] メッセージ送信確認
  - [ ] 既読機能確認
- メモ: ページネーション実装

### フェーズ5: 通知・管理機能

#### 18. **通知システムの実装**
- 概要: メール・ブラウザ通知機能
- 完了条件:
  - [ ] `core/notification.py` に通知システム作成
  - [ ] SendGrid設定完了
  - [ ] マッチング成立時の即座メール送信
  - [ ] メッセージ受信時の即座メール送信
  - [ ] ブラウザ通知API実装
  - [ ] レート制限実装（1分間10件、1日100件）
- 動作確認:
  - [ ] 各種通知機能確認
  - [ ] 通知タイミング確認
  - [ ] レート制限確認
- メモ: 通知の重複送信防止機能、SendGrid API Key設定

**SendGrid設定:**
```python
# settings.py
EMAIL_BACKEND = 'sendgrid_backend.SendgridBackend'
SENDGRID_API_KEY = os.environ.get('SENDGRID_API_KEY')
DEFAULT_FROM_EMAIL = 'noreply@matchingsite.com'
```

#### 19. **ブロック・報告機能の実装**
- 概要: ユーザーブロック・不適切ユーザー報告機能
- 完了条件:
  - [ ] `matching/models.py` にReportモデル作成
  - [ ] ブロック・報告フォーム作成
  - [ ] ブロックユーザーの表示除外機能
  - [ ] 管理者への報告通知機能
- 動作確認:
  - [ ] ブロック機能確認
  - [ ] 報告機能確認
  - [ ] ブロックユーザー除外確認
- メモ: 報告理由の分類機能を含める

#### 20. **管理者機能の実装**
- 概要: 管理者用のカスタム機能
- 完了条件:
  - [ ] Django admin拡張
  - [ ] ユーザーアカウント削除機能
  - [ ] 不適切コンテンツ削除機能
  - [ ] ユーザーブロック/凍結機能
  - [ ] 統計情報ダッシュボード
- 動作確認:
  - [ ] 管理画面アクセス確認
  - [ ] 各種管理機能確認
- メモ: 管理者権限の適切な設定

### フェーズ6: UI/UX仕上げ

#### 21. **Bootstrap 5による UI実装**
- 概要: レスポンシブデザインとマッチングサイト適応デザイン
- 完了条件:
  - [ ] 全画面のBootstrap 5対応
  - [ ] PC・モバイル両対応レスポンシブデザイン
  - [ ] マッチングサイト適応カラーテーマ
  - [ ] Bootstrap Iconsの活用
- 動作確認:
  - [ ] モバイル表示確認
  - [ ] タブレット表示確認
  - [ ] デスクトップ表示確認
- メモ: 有名マッチングサイトのUIを参考

#### 22. **パフォーマンス最適化**
- 概要: データベースクエリ最適化と画像最適化
- 完了条件:
  - [ ] データベースクエリ最適化
  - [ ] 画像自動リサイズ・圧縮
  - [ ] 静的ファイル圧縮
  - [ ] キャッシュ戦略実装
- 動作確認:
  - [ ] ページ読み込み速度確認
  - [ ] データベースクエリ数確認
- メモ: Django Debug Toolbarで確認

#### 23. **ヘルプ・FAQページの実装**
- 概要: ユーザーサポートページの作成
- 完了条件:
  - [ ] `core/views.py` にヘルプ・FAQビュー作成
  - [ ] よくある質問の整理・作成
  - [ ] 使い方ガイドの作成
  - [ ] 問い合わせフォームの実装
- 動作確認:
  - [ ] `/help/` でヘルプページ表示確認
  - [ ] `/faq/` でFAQページ表示確認
  - [ ] 問い合わせフォーム動作確認
- メモ: 利用規約・プライバシーポリシーも含める

#### 24. **統合テスト・E2Eテスト**
- 概要: 全機能の統合テストとエンドツーエンドテスト
- 完了条件:
  - [ ] 全機能統合テスト作成
  - [ ] ユーザーシナリオテスト
  - [ ] パフォーマンステスト
  - [ ] セキュリティテスト
- 動作確認:
  - [ ] 全テストパス確認
  - [ ] 実際のユーザーシナリオテスト
- メモ: テストデータの作成も含める

---

## 📊 進捗管理

### 現在の状況
- [ ] フェーズ1: プロジェクトセットアップとユーザー認証 (0/5)
- [ ] フェーズ2: プロフィール管理機能 (0/4)
- [ ] フェーズ3: マッチング機能 (0/4)
- [ ] フェーズ4: メッセージング機能 (0/4)
- [ ] フェーズ5: 通知・管理機能 (0/3)
- [ ] フェーズ6: UI/UX仕上げ (0/4)

### 全体進捗: 0/24 タスク完了

### 実装順序の依存関係
```
フェーズ1 → フェーズ2 → フェーズ3 → フェーズ4 → フェーズ5 → フェーズ6
     ↓        ↓        ↓        ↓        ↓        ↓
   必須基盤   必須基盤   コア機能   コア機能   付加機能   仕上げ
```

**注意**: 各フェーズの完了後、必ず `python manage.py test` と `flake8` でのコード品質チェックを実行してください。

---

## � **実装準備情報**

### 環境変数設定（.env ファイル）
```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/matching_site
POSTGRES_DB=matching_site
POSTGRES_USER=user
POSTGRES_PASSWORD=password

# Django
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# Redis
REDIS_URL=redis://localhost:6379/0

# SendGrid
SENDGRID_API_KEY=your-sendgrid-api-key

# Media Files
MEDIA_ROOT=/app/media/
MEDIA_URL=/media/

# Security
CSRF_COOKIE_SECURE=False
SESSION_COOKIE_SECURE=False
```

### Docker設定（docker-compose.yml）
```yaml
version: '3.8'
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: matching_site
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/matching_site
      - REDIS_URL=redis://redis:6379/0
    volumes:
      - ./media:/app/media

volumes:
  postgres_data:
```

---

## �🔗 関連ドキュメント

- 実装要件: [specification.md](./specification.md)
- 設計方針: [design-doc.md](../../../design-doc.md)
- タスク分割指針: [make-task.md](../../../plan/make-task.md)
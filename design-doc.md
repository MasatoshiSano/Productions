# マッチングサイトアプリ 設計方針書

## 📋 概要

マッチングサイトアプリの実装方針を定義します。Django + Bootstrap 5を使用し、ユーザー同士の出会いやマッチングを促進するWebアプリケーションの設計方針を示します。外部API・CDN使用禁止、Reactなどのフロントエンドフレームワーク使用禁止という制約下で、セキュアで使いやすいプラットフォームを構築します。

## 🔍 要件の分析

### 主要機能ブロック
1. **認証システム**: ユーザー登録・ログイン・パスワード管理
2. **プロフィール管理**: 作成・編集・写真アップロード
3. **マッチングエンジン**: ユーザー発見・いいね機能・マッチング成立
4. **メッセージシステム**: リアルタイムチャット・通知
5. **管理・セキュリティ**: ブロック・報告・管理者機能

### 技術的制約
- Django + Bootstrap 5 + SQLite
- 外部API使用禁止
- フロントエンドフレームワーク使用禁止
- セキュリティ対策必須（CSRF、XSS、SQLインジェクション）

## 🛠 実装方針

### アーキテクチャ選択

**選択**: Django MVT（Model-View-Template）パターン + 部分的SPA
- **Model**: Django ORM使用、リレーショナルデータベース設計
- **View**: Django CBV（Class-Based Views）中心
- **Template**: Bootstrap 5 + Django Templates + vanilla JavaScript
- **リアルタイム**: Django Channels（WebSocket）

**選択理由**: 
- Django標準アーキテクチャで開発効率を最大化
- 外部フレームワーク禁止制約に対応
- WebSocketで必要な部分のみリアルタイム化

### コンポーネント設計

#### 1. 認証・ユーザー管理層
- **Django Auth System**: 標準認証機能
- **Django Allauth**: ソーシャル認証・メール認証
- **Custom User Profile**: 拡張プロフィール情報

#### 2. マッチング処理層
- **MatchingEngine**: いいね・マッチング判定ロジック
- **DiscoveryService**: ユーザー発見・フィルタリング
- **GeoLocation**: 地理的距離計算

#### 3. メッセージング層
- **Django Channels**: WebSocket通信
- **MessageHandler**: リアルタイムメッセージ処理
- **NotificationService**: 通知管理

#### 4. ファイル管理層
- **PhotoUploadHandler**: 画像アップロード・リサイズ
- **MediaStorage**: ローカルストレージ管理
- **SecurityValidator**: アップロードファイル検証

### データフロー

#### マッチング処理フロー
1. ユーザー発見リクエスト → DiscoveryService
2. フィルタリング・ソート → マッチング候補表示
3. いいね/パス → MatchingEngine
4. 相互いいね判定 → マッチング成立
5. 通知送信 → NotificationService

#### リアルタイムメッセージフロー
1. WebSocket接続確立 → Django Channels
2. メッセージ送信 → MessageHandler
3. 相手側への配信 → WebSocket
4. データベース永続化 → Message Model
5. 未読管理 → 既読状態更新

## 🔄 実装方法の選択肢と決定

### 選択肢1: 従来型MPA（Multi-Page Application）
- **長所**: シンプル、SEO対応、開発速度
- **短所**: UX劣化、リアルタイム性低い
- **評価**: 基本機能には適しているが、マッチング体験が劣る

### 選択肢2: 完全SPA（Single Page Application）
- **長所**: 優れたUX、リアルタイム性高い
- **短所**: フロントエンドフレームワーク必要、制約違反
- **評価**: 制約により選択不可

### 選択肢3: ハイブリッド型（MPA + 部分的Ajax/WebSocket）
- **長所**: 制約内で最適なUX、段階的実装可能
- **短所**: 複雑性やや高い
- **評価**: 制約とUXのバランスが最適

### 決定: ハイブリッド型（MPA + 部分的Ajax/WebSocket）

**選択理由**:
- 技術制約内で最高のユーザー体験を実現
- マッチング・メッセージ部分のみリアルタイム化
- 段階的開発・テストが可能
- 保守性と拡張性を確保

## 📊 技術的制約と考慮事項

### セキュリティ対策
- **CSRF Protection**: Django標準機能 + カスタムトークン
- **XSS Prevention**: テンプレートエスケープ + CSP Header
- **SQL Injection**: Django ORM使用（Raw SQL禁止）
- **File Upload Security**: MIMEタイプ検証 + サイズ制限

### パフォーマンス考慮
- **Database Optimization**: インデックス設計、クエリ最適化
- **Static File Handling**: Django静的ファイル配信最適化
- **Image Processing**: Pillow使用、リサイズ・圧縮
- **Caching Strategy**: Django Cache Framework活用

### 拡張性設計
- **Modular Architecture**: Django Apps分割
- **API Ready**: REST API対応準備
- **Horizontal Scaling**: データベース分離設計
- **Feature Flags**: 機能別有効/無効切り替え

## 🔧 実装コンポーネント詳細

### 1. 認証コンポーネント
```
apps/accounts/
├── models.py (Profile, UserSettings)
├── views.py (CBV: SignUpView, LoginView, ProfileView)
├── forms.py (CustomUserCreationForm, ProfileForm)
├── urls.py (認証関連URL)
└── templates/accounts/ (認証画面テンプレート)
```

### 2. マッチングコンポーネント
```
apps/matching/
├── models.py (Like, Match, Block)
├── views.py (DiscoveryView, MatchingView)
├── services.py (MatchingEngine, DiscoveryService)
├── urls.py (マッチング関連URL)
└── templates/matching/ (マッチング画面テンプレート)
```

### 3. メッセージコンポーネント
```
apps/messaging/
├── models.py (Message, Conversation)
├── views.py (MessageListView, MessageDetailView)
├── consumers.py (WebSocket Consumer)
├── routing.py (WebSocket URL routing)
└── templates/messaging/ (メッセージ画面テンプレート)
```

### 4. 共通コンポーネント
```
apps/core/
├── models.py (共通モデル)
├── utils.py (共通ユーティリティ)
├── validators.py (カスタムバリデータ)
└── middleware.py (カスタムミドルウェア)
```

## 🎨 UI/UX設計方針

### デザインシステム
- **Bootstrap 5**: レスポンシブデザイン基盤
- **カスタムCSS**: ブランドカラー・コンポーネント
- **JavaScript**: vanilla JS（jQuery最小限使用）
- **アイコン**: Bootstrap Icons使用

### インタラクションデザイン
- **カードUI**: Tinder風スワイプ操作（タップ/クリック）
- **モーダル**: プロフィール詳細表示
- **トースト通知**: マッチング・メッセージ通知
- **プログレスバー**: 写真アップロード進捗

### レスポンシブ対応
- **Mobile First**: スマートフォン最適化
- **タブレット**: 中間画面サイズ対応
- **デスクトップ**: 大画面レイアウト調整

## 🚀 開発・デプロイ戦略

### 開発フェーズ
1. **Phase 1**: 基本認証・プロフィール機能
2. **Phase 2**: マッチング・発見機能
3. **Phase 3**: メッセージ・リアルタイム機能
4. **Phase 4**: 管理・セキュリティ機能

### テスト戦略
- **Unit Test**: Django TestCase使用
- **Integration Test**: Selenium WebDriver
- **Performance Test**: Django Debug Toolbar
- **Security Test**: OWASP ZAP使用

### デプロイ構成
- **Development**: SQLite + Django Dev Server
- **Production**: PostgreSQL + Gunicorn + Nginx
- **Static Files**: Django Collectstatic
- **Media Files**: 専用ディレクトリ

## ❓ 解決すべき技術的課題

### 1. 地理的距離計算
**課題**: 位置情報の精度と計算方法
**解決方針**: 
- 都道府県ベース→GPS座標段階的移行
- Haversine formula使用
- 距離計算キャッシュ化

### 2. リアルタイム性能
**課題**: 同時接続ユーザー数対応
**解決方針**:
- Django Channels + Redis
- WebSocket接続プール管理
- メッセージ配信最適化

### 3. 画像処理最適化
**課題**: 写真アップロード・表示性能
**解決方針**:
- Pillow使用、複数サイズ生成
- 遅延読み込み実装
- 画像圧縮・最適化

### 4. セキュリティ強化
**課題**: 年齢認証・本人確認
**解決方針**:
- 段階的認証システム
- 写真による年齢推定
- 管理者承認フロー

### 5. パフォーマンス最適化
**課題**: データベースクエリ最適化
**解決方針**:
- select_related/prefetch_related使用
- インデックス設計最適化
- キャッシュ戦略実装

## 📈 成功指標

### 技術的指標
- **応答時間**: 95%のリクエストが2秒以内
- **可用性**: 99.9%以上のアップタイム
- **セキュリティ**: 脆弱性ゼロ維持
- **パフォーマンス**: 同時接続1000ユーザー対応

### ユーザー体験指標
- **マッチング成功率**: 月間アクティブユーザーの30%以上
- **メッセージ応答率**: マッチング後24時間以内50%以上
- **ユーザー継続率**: 1ヶ月後60%以上維持
- **不具合報告**: 月間10件以下

この設計方針書に基づいて、段階的かつセキュアなマッチングサイトアプリケーションの実装を進めます。
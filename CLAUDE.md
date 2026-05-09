# Memory Pin — CLAUDE.md

## プロジェクト概要

思い出をマップに登録・閲覧できる個人向けWebアプリ「Memory Pin」の開発プロジェクト。

## 開発者について

- 新卒フロントエンドエンジニア（業務では React / TypeScript / MUI / React Hook Form / OpenAPI を使用）
- Svelte / SvelteKit / Tailwind CSS / Supabase はすべて初めて触る
- このプロジェクトは学習目的で進めている

## ⚠️ 最重要ルール: 先生モード

このプロジェクトでは **Claude は先生として振る舞うこと**。

### やるべきこと
- 質問に対して、考え方・アプローチのヒントを出す
- 該当する公式ドキュメントのURLやセクションを教える
- エラーに対して、原因の方向性と調べるべきキーワードを教える
- コードの問題点を指摘し、なぜ問題なのかを解説する
- React での同等概念がある場合は、比較して説明する（開発者が React 経験者のため）

### やってはいけないこと
- 完成したコードをそのまま書いて渡すこと
- コピペで動くソリューションを提示すること
- 質問されていない部分まで先回りして実装すること

### 例外（コードを書いてよいケース）
- 開発者が「コード書いて」「実装して」と明示的に依頼した場合
- 設定ファイルの記述方法（svelte.config.js 等の定型的な設定）
- 5行以下の短いコード例でアプローチを示す場合

## 技術スタック

| レイヤー | 技術 |
|---|---|
| フレームワーク | SvelteKit (Svelte 5) |
| UIコンポーネント | shadcn-svelte (Maia プリセット) |
| ボトムシート | vaul-svelte (shadcn-svelte 内蔵) |
| スタイリング | Tailwind CSS |
| 地図 | sveaflet (Leaflet ラッパー) |
| 場所検索 | Nominatim API (OpenStreetMap) |
| BaaS / DB | Supabase (PostgreSQL + PostGIS) |
| 認証 | Supabase Auth (Google OAuth) |
| 画像保存 | Supabase Storage |
| 画像圧縮 | browser-image-compression |
| ピンクラスタリング | leaflet.markercluster |
| ホスティング | Cloudflare Pages (adapter-static) |

## アーキテクチャ

```
Cloudflare Pages（静的ファイル配信）
      ↓
SvelteKit SPA（adapter-static でビルド）
      ↓
Supabase Client SDK → Supabase (Auth / DB / Storage)
                          ↑
                      RLS でアクセス制御
```

- BEサーバーなし。Supabase SDK でフロントから直接 CRUD
- 認可は Row Level Security (RLS) で `auth.uid()` ベース

## データモデル

### users
- id (uuid, PK) — Supabase Auth の user id
- display_name (varchar) — 表示名
- avatar_url (varchar, nullable) — アバター画像URL
- created_at, updated_at

### memories
- id (uuid, PK)
- user_id (uuid, FK → users)
- title (varchar)
- description (text, nullable)
- date (date, nullable) — 任意
- place_name (varchar)
- latitude (float8)
- longitude (float8)
- created_at, updated_at

### memory_images
- id (uuid, PK)
- memory_id (uuid, FK → memories)
- storage_path (varchar) — Supabase Storage のパス
- display_order (int)
- created_at

### companions（同行者）
- id (uuid, PK)
- memory_id (uuid, FK → memories)
- name (varchar) — フリーテキスト
- user_id (uuid, FK → users, nullable) — 将来の他ユーザー紐付け用

## 画面構成

1. **ログイン画面** — Google OAuth ボタンのみ
2. **初回プロフィール登録** — 名前入力（Google名がデフォルト）
3. **マップ画面（メイン）** — 全画面地図 + 検索バー（上部） + FAB（右下） + アバターアイコン（左上）
4. **登録モーダル（ボトムシート）** — 場所・タイトル・日付・写真・同行者・メモ
5. **ポップアップ** — ピンタップでサムネ・タイトル・日付を表示
6. **詳細モーダル（ボトムシート）** — 画像カルーセル・場所・日付・同行者・メモ
7. **設定画面** — 名前/アバター変更・ログアウト・アカウント削除

## 画面遷移

```
ログイン → (初回) プロフィール登録 → マップ画面
マップ画面:
  ├── 検索バー → 検索結果ドロップダウン → 地図移動
  ├── +ボタン / 地図タップ → 登録モーダル → 保存 → ピン追加
  ├── ピンタップ → ポップアップ → タップ → 詳細モーダル
  └── アバターアイコン → 設定画面
```

## 公式ドキュメント（回答時はここから案内すること）

- SvelteKit: https://svelte.dev/docs/kit
- Svelte 5: https://svelte.dev/docs/svelte
- shadcn-svelte: https://www.shadcn-svelte.com
- sveaflet: https://sveaflet-doc.vercel.app
- Supabase JS: https://supabase.com/docs/reference/javascript
- Supabase Auth: https://supabase.com/docs/guides/auth
- Supabase Storage: https://supabase.com/docs/guides/storage
- Leaflet: https://leafletjs.com/reference.html
- Nominatim API: https://nominatim.org/release-docs/latest/api/Search/
- vaul-svelte: https://github.com/huntabyte/vaul-svelte
- Tailwind CSS: https://tailwindcss.com/docs

## コーディング規約

- TypeScript を使用する
- コンポーネントは機能単位でフォルダ分け
- Svelte 5 の runes ($state, $derived, $effect) を使う
- shadcn-svelte のコンポーネントを積極的に活用
- Tailwind CSS でスタイリング（インラインスタイル禁止）

## レビュー観点

コードレビュー時は以下を確認すること（詳細は .claude/skills/review/SKILL.md 参照）:

1. Svelte のベストプラクティスに沿っているか
2. TypeScript の型が適切か
3. アクセシビリティ（a11y）
4. パフォーマンス（不要な再レンダリング等）
5. セキュリティ（RLS、認証チェック）
6. コードの読みやすさ
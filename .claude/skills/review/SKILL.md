---
name: memory-pin-review
description: Memory Pin プロジェクトのコードレビュースキル。開発者がコードレビューを依頼したとき、コードの変更を確認したとき、PRを見てほしいと言ったとき、または「レビューして」「チェックして」「見て」などの依頼があったときに使用する。レビュー結果は教育的な観点で、なぜ問題なのか・どう改善すべきかの考え方を示す。
---

# Memory Pin コードレビュースキル

## レビューの原則

1. **教育優先**: 問題を指摘するだけでなく、なぜ問題なのかを説明する
2. **React との比較**: 開発者は React 経験者なので、Svelte との違いを比較で示すと理解が早い
3. **公式ドキュメント参照**: 指摘には該当する公式ドキュメントのURLを添える
4. **修正コードは書かない**: 方向性とヒントを示し、自分で修正させる。ただし「直して」と明示された場合は修正コードを提示してよい
5. **良い点も伝える**: 良いコードには具体的に褒める。学習モチベーションの維持が重要

## レビュー観点

### 1. Svelte ベストプラクティス

チェック項目:
- Svelte 5 の runes (`$state`, `$derived`, `$effect`) を正しく使えているか
- `$effect` の乱用がないか（React の useEffect と同じ落とし穴）
- リアクティブな値の更新が適切か（直接代入 vs 不変更新）
- コンポーネントの粒度は適切か（大きすぎ / 小さすぎ）
- `{#each}` ブロックに key を付けているか
- `onMount` / `onDestroy` のライフサイクル管理

参照:
- Svelte 5 runes: https://svelte.dev/docs/svelte/$state
- Reactivity: https://svelte.dev/docs/svelte/reactivity

React との比較ポイント:
- `$state` ≈ `useState` だが、setter 関数不要で直接代入
- `$derived` ≈ `useMemo` だが、依存配列の指定不要
- `$effect` ≈ `useEffect` だが、依存の自動追跡
- Svelte はコンパイラなので、React のように「なぜ再レンダリングされたか」の問題が起きにくい

### 2. TypeScript

チェック項目:
- `any` を使っていないか
- Props の型定義が適切か
- Supabase のレスポンス型を活用しているか
- イベントハンドラの型が正しいか
- null / undefined の安全な処理（optional chaining, nullish coalescing）

参照:
- SvelteKit + TypeScript: https://svelte.dev/docs/kit/types

### 3. shadcn-svelte / UI

チェック項目:
- shadcn-svelte の既存コンポーネントで代替できるのに自前実装していないか
- Tailwind CSS のクラスが冗長でないか
- レスポンシブ対応（モバイルファースト）
- ダークモード対応（CSS変数の使用）
- vaul-svelte の Drawer が適切に使われているか

参照:
- shadcn-svelte コンポーネント: https://www.shadcn-svelte.com/docs
- Tailwind CSS: https://tailwindcss.com/docs

### 4. アクセシビリティ (a11y)

チェック項目:
- インタラクティブ要素に適切な aria 属性があるか
- 画像に alt テキストがあるか
- キーボード操作が可能か
- フォーカス管理（モーダル開閉時）
- 色のコントラスト比

参照:
- Svelte a11y: https://svelte.dev/docs/svelte/accessibility
- Svelte はコンパイル時に a11y 警告を出してくれるので、警告を無視していないか確認

### 5. パフォーマンス

チェック項目:
- 地図関連: ピンの大量描画時にクラスタリングしているか
- 画像: アップロード前に圧縮しているか
- Supabase クエリ: 不要なデータを取得していないか（select で必要カラムのみ）
- `$effect` 内で重い処理をしていないか
- コンポーネントの不要な再生成

### 6. セキュリティ

チェック項目:
- Supabase のクライアントキー（anon key）のみ使用しているか（service_role key は絶対にフロントに置かない）
- RLS ポリシーが適切に設定されているか
- ユーザー入力のサニタイズ（XSS対策）
- 画像アップロードのバリデーション（ファイルタイプ、サイズ）

参照:
- Supabase RLS: https://supabase.com/docs/guides/database/postgres/row-level-security

### 7. コードの読みやすさ

チェック項目:
- 変数名・関数名が意図を表しているか
- コンポーネント名がわかりやすいか
- 複雑なロジックにコメントがあるか
- 1ファイルが長すぎないか（200行超えたら分割を検討）
- マジックナンバーがないか

### 8. SvelteKit 固有

チェック項目:
- ファイルベースルーティングの規約に沿っているか
- `+page.svelte` / `+layout.svelte` の使い分け
- `$app/stores` の適切な使用
- 環境変数の管理（`$env/static/private` vs `$env/static/public`）
- Supabase の初期化が適切な場所にあるか

参照:
- SvelteKit routing: https://svelte.dev/docs/kit/routing
- SvelteKit env: https://svelte.dev/docs/kit/$env-static-public

## レビュー出力フォーマット

```
## レビュー結果

### 🔴 要修正（must fix）
- [観点] 問題の説明
  - なぜ問題か: ...
  - ヒント: ...
  - 参照: URL

### 🟡 改善推奨（should fix）
- [観点] 問題の説明
  - なぜ問題か: ...
  - ヒント: ...
  - 参照: URL

### 🟢 良い点
- 具体的に何が良いか

### 💡 学習ポイント
- このコードに関連する Svelte / SvelteKit の概念で、知っておくと今後役立つこと
```

## レビュー時の注意

- 完璧を求めすぎない。学習段階なので、最も重要な2〜3点に絞って指摘する
- 同じ種類の問題が複数箇所にある場合、1箇所だけ指摘して「他にも同様の箇所があるので探してみて」と促す
- 初めて触る技術なので、基本的な間違いにも丁寧に対応する
- 「React ではこう書くけど、Svelte ではこう」という比較は積極的に使う
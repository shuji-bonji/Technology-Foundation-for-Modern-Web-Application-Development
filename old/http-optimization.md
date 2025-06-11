# HTTP最適化技術

Webパフォーマンスを向上させるためには、HTTP通信に関連する最適化技術を活用することが重要です。以下に代表的な手法を紹介します。

## サーバープリコネクト（Preconnect）

`<link rel="preconnect">` を使うことで、外部ドメインへのDNSルックアップ、TLSハンドシェイク、TCP接続をあらかじめ行うことができます。これにより、リクエスト開始時の待機時間を短縮できます。

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
```

## リンクプリフェッチ（Prefetch）

`<link rel="prefetch">` は、将来使うリソース（次のページなど）をバックグラウンドで事前取得しておく手法です。ユーザーが次にどのページに進むか予測可能な場合に有効です。

```html
<link rel="prefetch" href="/next-page.html">
```

## オブジェクトプリロード（Preload）

`<link rel="preload">` により、CSSやフォント、重要なJavaScriptなど、優先的に読み込みたいリソースを明示的に指定して先に読み込ませることができます。

```html
<link rel="preload" href="/style.css" as="style">
<link rel="preload" href="/font.woff2" as="font" type="font/woff2" crossorigin="anonymous">
```

## 遅延読み込み（Lazy Loading）

`loading="lazy"` 属性を `<img>` や `<iframe>` に付与することで、ビューポート外のリソースはスクロールされるまで読み込まれなくなります。初期読み込みを軽減できます。

```html
<img src="image.jpg" loading="lazy" alt="説明文">
```

## まとめ

これらの技術を適切に組み合わせることで、**初期表示速度の向上**や**帯域の節約**、**ユーザー体験の改善**が期待できます。
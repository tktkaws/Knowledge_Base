# astro:assets — 画像最適化の仕組み

## astro:assets とは

- Astro 組み込みの画像最適化モジュール（`astro:assets`）
- ビルド時（または SSR 時）にリサイズ・フォーマット変換・圧縮を行う
- `<Image />` コンポーネントと `getImage()` 関数が中心
- 追加ライブラリなしで WebP / AVIF 等への変換が可能

> 参照: [Astro — Images](https://docs.astro.build/en/guides/images/)

## src/ の import 画像 vs public/

| 置き場所 | 最適化 | width/height | 向く例 |
|---|---|---|---|
| `src/`（import） | 自動最適化 | import 時に推論可能 | コンポーネントで使う写真、OGP |
| `public/` | パス指定のみ（最適化は限定的） | **手動で必須** | favicon、固定URL、外部参照必須ファイル |

```astro
---
// 良い例：src から import（最適化の恩恵大）
import heroImage from '../assets/hero.jpg';
import { Image } from 'astro:assets';
---
<Image src={heroImage} alt="チームの集合写真" />
```

```astro
---
// public/ の画像（最適化は generate 時のみ、width/height 必須）
import { Image } from 'astro:assets';
---
<Image
  src="/images/logo.png"
  alt="会社ロゴ"
  width={200}
  height={80}
/>
```

- **`src/` + import** が基本推奨
- `public/` は URL が固定である必要がある場合に限定

> 参照: [Astro — Where to store images](https://docs.astro.build/en/guides/images/#where-to-store-images)

## Image コンポーネント

```astro
---
import { Image } from 'astro:assets';
import profile from '../assets/profile.jpg';
---
<Image
  src={profile}
  alt="山田太郎のプロフィール写真"
  width={400}
  height={400}
  format="webp"
  quality={80}
/>
```

- `alt` はアクセシビリティのため必須（装飾のみなら `alt=""`）
- `src/` の画像は `width` / `height` 省略可能（メタデータから推論）
- `format` で webp / avif 等を指定
- `quality` で圧縮率を調整

```astro
---
import { Image } from 'astro:assets';
import banner from '../assets/banner.jpg';
---
<!-- 悪い例：alt 欠落 -->
<Image src={banner} width={800} height={300} />

<!-- 悪い例：public 画像で width/height 未指定 -->
<Image src="/images/photo.jpg" alt="写真" />
```

> 参照: [Astro — Image component](https://docs.astro.build/en/reference/modules/astro-assets/#image-)

## width / height の重要性

- CLS（Cumulative Layout Shift）防止のため、表示サイズのヒントが重要
- `src/` import 画像は Astro が元サイズを読み取れる
- `public/` 画像は Astro がファイルを解析できないため **width / height 必須**
- レスポンシブ時は CSS で `max-width: 100%; height: auto;` 等と組み合わせ

```astro
<Image
  src={photo}
  alt="商品写真"
  widths={[400, 800, 1200]}
  sizes="(max-width: 768px) 100vw, 800px"
/>
```

- `widths` と `sizes` で srcset を生成し、デバイスに応じたサイズ配信

> 参照: [Astro — Responsive images](https://docs.astro.build/en/guides/images/#responsive-image-behavior)

## getImage() — プログラム的な最適化

- `<Image />` 以外の場所（CSS background、カスタム markup）向け
- 最適化済み URL や属性オブジェクトを返す
- サーバー側（ビルド / SSR）でのみ実行

```astro
---
import { getImage } from 'astro:assets';
import background from '../assets/background.png';

const optimized = await getImage({
  src: background,
  format: 'avif',
  width: 1920,
  height: 1080,
});
---
<div
  class="hero"
  style={`background-image: url(${optimized.src});`}
  role="img"
  aria-label="オフィスの風景"
>
  <slot />
</div>
```

```astro
---
// 悪い例：最適化せず public パスを background に直書き
---
<div style="background-image: url('/images/huge-photo.jpg');"></div>
```

- 大きな原寸 JPEG をそのまま配信するより、`getImage()` 経由が望ましい

> 参照: [Astro — getImage()](https://docs.astro.build/en/reference/modules/astro-assets/#getimage)

## フォーマットの選び方

| フォーマット | 特徴 |
|---|---|
| `webp` | 広いブラウザサポート、JPEG/PNG より小さくなりやすい |
| `avif` | さらに高圧縮、古いブラウザでは非対応のことも |
| `png` | 透過が必要な図版 |
| `jpg` | 写真系、透過不要 |

```astro
<Image src={photo} alt="風景" format="webp" />
```

- デフォルトの変換先は設定や Astro バージョンに依存
- 写真主体なら webp / avif、ロゴなら svg や png を検討

## Picture コンポーネント（フォーマットフォールバック）

```astro
---
import { Picture } from 'astro:assets';
import hero from '../assets/hero.jpg';
---
<Picture
  src={hero}
  formats={['avif', 'webp']}
  alt="メインビジュアル"
/>
```

- 複数フォーマットを `<picture>` として出力
- ブラウザが対応する最適フォーマットを選択

> 参照: [Astro — Picture component](https://docs.astro.build/en/reference/modules/astro-assets/#picture-)

## リモート画像

```astro
---
import { Image } from 'astro:assets';
---
<Image
  src="https://example.com/photo.jpg"
  alt="外部画像"
  inferSize
  width={800}
  height={600}
/>
```

- リモート URL は `astro.config.mjs` の `image.domains` または `image.remotePatterns` で許可が必要
- `inferSize` でリモート画像のサイズ推論（サーバーへの追加リクエストあり）

```js
// astro.config.mjs
export default defineConfig({
  image: {
    domains: ['example.com'],
  },
});
```

> 参照: [Astro — Authorizing remote images](https://docs.astro.build/en/guides/images/#authorizing-remote-images)

## Markdown / Content Collections 内の画像

```md
<!-- src/content/blog/post.md -->
![説明テキスト](../assets/diagram.png)
```

- Content Collections 内の相対パス画像も最適化対象
- MDX では `<Image />` コンポーネントを直接使える

## 通常の `<img>` との比較

```astro
---
import rawPhoto from '../assets/photo.jpg';
---
<!-- 悪い例：import したのに img で原寸配信 -->
<img src={rawPhoto.src} alt="写真" />
```

```astro
---
import { Image } from 'astro:assets';
import photo from '../assets/photo.jpg';
---
<!-- 良い例：最適化 + 適切な属性 -->
<Image src={photo} alt="写真" loading="lazy" decoding="async" />
```

- import だけでは最適化されない
- `<Image />` または `getImage()` を使う

## パフォーマンスのチェックリスト

- [ ] コンポーネント用画像は `src/` に置いて import している
- [ ] すべての `<Image />` に意味のある `alt` がある
- [ ] `public/` 画像には `width` / `height` を指定している
- [ ]  Above-the-fold 以外は `loading="lazy"` を検討
- [ ] 写真は webp / avif への変換を検討している
- [ ] リモート画像は domains / remotePatterns を設定している

## まとめ

- `astro:assets` の `<Image />` と `getImage()` が画像最適化の中心
- `src/` import が推奨、`public/` は固定 URL が必要な場合に限定
- `public/` 画像は width / height が必須、import 画像は推論可能
- `format` / `quality` / `widths` でサイズとフォーマットを制御
- 装飾でなければ `alt` 必須、lazy loading で初期表示を軽くする

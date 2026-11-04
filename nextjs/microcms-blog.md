## Next.jsでプロジェクトを作成

`npx create-next-app@latest [プロジェクト名]`

```
Would you like to use TypeScript? ... Yes
Which linter would you like to use? » ESLint
Would you like to use Tailwind CSS? ... Yes
Would you like your code inside a src/ directory? ... Yes
Would you like to use App Router? (recommended) ... Yes
Would you like to use Turbopack? (recommended) ... No
Would you like to customize the import alias (`@/*` by default)? ... No
```

```
  "dependencies": {
    "react": "19.1.0",
    "react-dom": "19.1.0",
    "next": "15.5.2"
  },
```

## microCMSでAPIを作成

参照

[microCMS + Next.jsでJamstackブログを作ってみよう（Next.js 15 ver.）](https://blog.microcms.io/microcms-next15-jamstack-blog/)

APIの型：リスト形式

エンドポイント：blogs

フィールド：title, body, tags

エンドポイント：tags

フィールド：title, slug

[https://xxxxxx.microcms.io/api/v1/blogs](https://frontdispatch.microcms.io/api/v1/blogs)

で以下のような結果が取得できれば問題ない。

エンドポイントをblogsにしてしまったが、URLが/blog/ ならエンドポイントをblogにしたほうが良かった。

管理画面でAPIプレビューできることが画期的だ。

自分で考えるにしてもAIに渡すにしても非常に便利。

```
{
    "contents": [
        {
            "id": "9n9g8ftwx",
            "createdAt": "2025-05-29T03:44:05.796Z",
            "updatedAt": "2025-05-29T03:44:31.284Z",
            "publishedAt": "2025-05-29T03:44:05.796Z",
            "revisedAt": "2025-05-29T03:44:31.284Z",
            "title": "マイノリティの「つながらない権利」――ひとりでも生存できる社会のために",
            "body": "",
            "tags": [
                {
                    "id": "cid_7nmyv98",
                    "createdAt": "2025-01-23T01:56:58.749Z",
                    "updatedAt": "2025-03-24T04:34:11.045Z",
                    "publishedAt": "2025-01-23T01:56:58.749Z",
                    "revisedAt": "2025-03-24T04:34:11.045Z",
                    "title": "Other",
                    "slug": "other"
                }
            ]
        },
      
    ],
    "totalCount": 1,
    "offset": 0,
    "limit": 10
}
```

## envファイルの作成

.env.localファイルを作成

```
MICROCMS_API_KEY=xxxxxxxxxx
MICROCMS_SERVICE_DOMAIN=xxxxxxxxxx
```

**MICROCMS_API_KEY**

microCMS 管理画面の「サービス設定 > API キー」から確認することができます。

**MICROCMS_SERVICE_DOMAIN**

microCMS 管理画面の URL（[https://xxxxxxxx.microcms.io）の](https://xxxxxxxx.microcms.xn--io\)-nc4b/) xxxxxxxx の部分です。

## microcms-js-sdkのインストール

`npm install microcms-js-sdk`

```
// libs/microcms.ts
import { createClient } from 'microcms-js-sdk';

// 環境変数にMICROCMS_SERVICE_DOMAINが設定されていない場合はエラーを投げる
if (!process.env.MICROCMS_SERVICE_DOMAIN) {
  throw new Error('MICROCMS_SERVICE_DOMAIN is required');
}

// 環境変数にMICROCMS_API_KEYが設定されていない場合はエラーを投げる
if (!process.env.MICROCMS_API_KEY) {
  throw new Error('MICROCMS_API_KEY is required');
}

// Client SDKの初期化を行う
export const client = createClient({
  serviceDomain: process.env.MICROCMS_SERVICE_DOMAIN,
  apiKey: process.env.MICROCMS_API_KEY,
});
```

## ブログ記事一覧を表示

```
// app/page.tsx
import Link from 'next/link';
import { client } from '../libs/microcms';

// ブログ記事の型定義
type Props = {
  id: string;
  title: string;
};

// microCMSからブログ記事を取得
async function getBlogPosts(): Promise<Props[]> {
  const data = await client.get({
    endpoint: 'blog', // 'blog'はmicroCMSのエンドポイント名
    queries: {
      fields: 'id,title',  // idとtitleを取得
      limit: 5,  // 最新の5件を取得
    },
  });
  return data.contents;
}

export default async function Home() {
  const posts = await getBlogPosts();

  return (
    <main>
      <h1>ブログ記事一覧</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>
            <Link href={`/blog/${post.id}`}> {/* 記事へのリンクを生成 */}
              {post.title} {/* タイトルを表示 */}
            </Link>
          </li>
        ))}
      </ul>
    </main>
  );
}
```

## ブログ詳細記事を表示

`npm install dayjs`

```
// app/blog/[id]/page.tsx
import { client } from '../../../libs/microcms';
import dayjs from 'dayjs';

// ブログ記事の型定義
type Props = {
  id: string;
  title: string;
  body: string;
  publishedAt: string;
  category: { name: string };
};

// microCMSから特定の記事を取得
async function getBlogPost(id: string): Promise<Props> {
  const data = await client.get({
    endpoint: `blog/${id}`,
  });
  return data;
}

// 記事詳細ページの生成
export default async function BlogPostPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params; // IDを取得
  const post = await getBlogPost(id);

  // dayjsを使ってpublishedAtをYY.MM.DD形式に変換
  const formattedDate = dayjs(post.publishedAt).format('YY.MM.DD');

  return (
    <main>
      <h1>{post.title}</h1> {/* タイトルを表示 */}
      <div>{formattedDate}</div> {/* 日付を表示 */}
      <div>カテゴリー：{post.category && post.category.name}</div> {/* カテゴリーを表示 */}
      <div dangerouslySetInnerHTML={{ __html: post.body }} /> {/* 記事本文を表示 */}
    </main>
  );
}

// 静的パスを生成
export async function generateStaticParams() {
  const contentIds = await client.getAllContentIds({ endpoint: 'blog' });

  return contentIds.map((contentId) => ({
    id: contentId, // 各記事のIDをパラメータとして返す
  }));
}
```

## 404ページの作成

appディレクトリにnot-found.tsxを作成

## Vercelでデプロイ

Environment Variablesに、envファイルに記載していたMICROCMS_API_KEYとMICROCMS_SERVICE_DOMAINを登録

## Webhookの設定

デプロイしたプロジェクトの**Settings**から**Git > Deploy Hooks**にてWebhookを作成

Hookは任意の名前でOK、ブランチはデプロイに使用しているブランチ名を記述

microCMSの管理画面へ移動し、コンテンツ「xxx」のAPI設定 > Webhookから設定microcmsのブログを参照





ダークモード導入

シンタックスハイライト

フィルター機能

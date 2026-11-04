# Turnstile と reCAPTCHA — 問い合わせフォームのスパム対策

## スパム対策とは

- 問い合わせフォームは、ボットによる自動投稿（スパム）の標的になりやすい
- 対策として広く使われてきたのが Google の **reCAPTCHA**（リキャプチャ）
- 近年は Cloudflare の **Turnstile**（ターンスタイル）も選択肢として定着している
- どちらも無料枠があるが、条件と運用負荷は同じではない
- 本記事では、問い合わせフォーム用途での違いと、今後どちらを選ぶかを整理する

## reCAPTCHA と Turnstile の概要

### reCAPTCHA

- Google が提供するボット対策サービス
- 画像選択型（v2）や、ユーザー操作をほぼ求めないスコア型（v3）がある
- WordPress の問い合わせフォームでも長年の定番だった
- 現在は Google Cloud 上のサービスとして提供される — キー管理や課金設定が Cloud 側に寄っている

### Turnstile

- Cloudflare が提供する CAPTCHA の代替サービス
- 多くのケースでパズル操作を求めず、バックグラウンドで検証する
- プライバシー重視の設計を謳っており、広告トラッキング前提ではない
- Free プランでもチャレンジ回数に上限がない — ウィジェット数・ホスト名数には上限がある

> 参照: [Cloudflare Turnstile](https://developers.cloudflare.com/turnstile/) / [Google Cloud reCAPTCHA](https://cloud.google.com/recaptcha)

## reCAPTCHA にかかりはじめた制限

- かつての「キーを発行して埋め込むだけ」の運用から、Google Cloud 前提の運用へ移行が進んだ
- 無料枠は組織あたり月 **10,000** アセスメントまで
  - サイト単位ではなく、Google Cloud の **organization 全体で合算** される
  - 複数サイトを同じ組織で運用していると、枠を早く消費しやすい
- 枠を超えると、課金アカウントが有効でない場合は新しいリクエストがエラーになる
- 有料側はおおよそ次のイメージ
  - 10,001〜100,000 / 月: 定額 **$8**
  - 100,000 超: 1,000 アセスメントあたり **$1**
- 「無料で使える」こと自体は残っているが、上限・課金設定・Cloud 管理の手間が現実的な制約になってきた

> 参照: [reCAPTCHA — Billing information](https://docs.cloud.google.com/recaptcha/docs/billing-information) / [Compare features between reCAPTCHA tiers](https://cloud.google.com/recaptcha-enterprise/docs/compare-tiers)

## Contact Form 7 は Turnstile を推奨している

- Contact Form 7（CF7）は公式に Turnstile 連携モジュールを提供している
- ドキュメント上の方針は明確 — **特別な理由がなければ Turnstile を推奨**、reCAPTCHA は必要な場合のみ
- 理由のひとつとして、Turnstile が無料で使える点を挙げている
- 導入の流れは次のとおり
  1. Cloudflare でウィジェットを作成し、サイトキーとシークレットキーを取得する
  2. WordPress 管理画面の「お問い合わせ → インテグレーション」から Turnstile を設定する
  3. キーを貼り付けると、各フォームにウィジェットが付与される
- 表示位置を変えたい場合は、フォームテンプレートに `[turnstile]` タグを置く
- `theme` / `size` / `language` などのオプションもタグ側で指定できる

> 参照: [Contact Form 7 — Cloudflare Turnstile integration](https://contactform7.com/turnstile-integration/)

## 比較表

| 項目 | reCAPTCHA | Turnstile |
|---|---|---|
| 提供元 | Google | Cloudflare |
| 無料枠 | 月 **10,000** アセスメント（組織合算） | Free プランでチャレンジ回数は無制限 |
| 課金の前提 | Google Cloud と課金設定が絡む | Cloudflare アカウントで始められる |
| UX | v2 は画像選択、v3 はスコア型 | 多くの場合は非干渉・低摩擦 |
| CF7 連携 | あり | **公式推奨**・インテグレーションあり |
| 向いている用途 | Google エコシステム前提、リスクスコア活用など | 問い合わせフォーム中心の一般的なサイト |

> 参照: [Cloudflare Turnstile plans](https://developers.cloudflare.com/turnstile/plans/)

## 今後は Turnstile の方がよいのか

- 問い合わせフォームのスパム対策としては、**Turnstile を第一候補にするのが妥当**
  - CF7 が公式推奨している
  - 無料枠の上限に当たりにくい
  - Google Cloud の課金・組織管理を持ち込まずに済む
- reCAPTCHA を残す合理的な理由もある
  - すでに Google Cloud 側の運用が整っている
  - v3 のスコアを独自のリスク判定に組み込んでいる
  - 社内ポリシーで Google 製品が指定されている
- 「どちらも無料だから同じ」ではない — 制限の形と運用コストが違う
- 新規構築や CF7 利用サイトでは、特別な事情がなければ Turnstile へ寄せる判断でよい

## まとめ

- 問い合わせフォームのボット対策として、reCAPTCHA と Turnstile が代表的
- reCAPTCHA は無料枠が月 **10,000** アセスメントに制限され、Google Cloud 前提の運用になった
- Turnstile は Free プランでもチャレンジ回数に上限がなく、導入のハードルが低い
- Contact Form 7 は Turnstile 連携を提供し、公式に推奨している
- 一般的な問い合わせフォーム用途では、今後は Turnstile を選ぶ方がよい
- reCAPTCHA はスコア活用や Google 側の既存運用がある場合の選択肢として残る

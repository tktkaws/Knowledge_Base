# 一時的・状況的な制約 — 片手操作、騒音、屋外のまぶしさでも同じ設計が効く理由

## 一時的・状況的な制約とは

- Webの使いにくさは、永続的な障害だけから起きるわけではない
- **一時的な制約**: 骨折、手術後、眼鏡を忘れた、薬の副作用などで、しばらく操作・知覚が制限される状態
- **状況的な制約**: 屋外のまぶしさ、電車内の騒音、子どもを抱えた片手操作、低速回線など、環境や状況による制限
- W3C WAIも、一時的障害・状況的制限の例を挙げ、アクセシビリティが障害のない人にも利点があると説明する
- 制約は「特別な人」だけの話ではなく、誰にでも起こり得る利用条件

> 参照:
> - [W3C WAI — Introduction to Web Accessibility](https://www.w3.org/WAI/fundamentals/accessibility-intro/)
> - [W3C WAI — Diverse Abilities and Barriers](https://www.w3.org/WAI/people-use-web/abilities-barriers/)

## 永続・一時・状況はつながっている

Microsoftの Inclusive Design で使われる **Persona Spectrum** の考え方:

| 軸の例 | 永続的 | 一時的 | 状況的 |
|---|---|---|---|
| 片手操作 | 片腕 | 腕の骨折・ギプス | 乳児を抱えている |
| 聞こえにくい | 難聴 | 中耳炎 | 騒がしい駅・工事現場 |
| 見えにくい | ロービジョン | 目の手術後・眼鏡忘れ | 強い日差しの下 |
| 集中しづらい | ADHDなど | 睡眠不足・体調不良 | 移動中・割り込みが多い場 |

- 永続的なニーズ向けに設計すると、一時的・状況的な利用にも効きやすい
- 「障害のある人専用の機能」ではなく、同じUI改善が広い利用条件をカバーする
- 社会モデルとも相性が良い — 問題は個人ではなく、その場の環境・設計とのミスマッチ

> 参照: [Microsoft Inclusive Design — Inclusive 101](https://inclusive.microsoft.design/articles/inclusive-101-guidebook)

## よくある状況と、効く設計

| 状況 | 起きやすいこと | 効く設計（例） |
|---|---|---|
| 片手操作 | 精密なクリック・ドラッグが難しい | 十分なタッチターゲット、ドラッグ必須を避ける |
| 騒音下 | 音声が聞こえない | 字幕・文字起こし、音声のみに依存しない |
| 屋外のまぶしさ | 低コントラストが見えない | 十分なコントラスト、色だけに頼らない |
| 通勤・移動中 | 読み終わる前に画面が変わる | 自動再生の抑制、十分な時間、一時停止 |
| 低速回線 | 動画や重いUIが届かない | テキスト代替、軽量な代替コンテンツ |
| 小さな画面 | 横スクロール・切れが生じる | リフロー、相対単位、レスポンシブ |

- これらはWCAGの達成基準と重なることが多い
- 「便利なUX」と「アクセシビリティ」は別物ではなく、同じ実装で両方を満たせる

## フロントエンドでの具体例

### 1. 片手・タッチでも押せる

```css
/* 悪い例：小さすぎて誤タップしやすい */
.icon-btn {
  width: 20px;
  height: 20px;
}

/* 良い例：指でも押しやすいサイズを確保 */
.icon-btn {
  min-width: 44px;
  min-height: 44px;
}
```

- WCAG 2.2 の 2.5.8 Target Size (Minimum) などと関連
- ドラッグ操作だけに依存せず、タップやボタンでも同じ結果に届ける

> 参照: [WCAG 2.2 — 2.5.8 Target Size (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)

### 2. 騒音下でも情報が届く

```html
<!-- 悪い例：音声だけ -->
<audio src="announce.mp3" controls></audio>

<!-- 良い例：文字でも同じ内容を渡す -->
<video controls>
  <source src="announce.mp4" type="video/mp4" />
  <track kind="captions" src="announce-ja.vtt" srclang="ja" label="日本語字幕" default />
</video>
<p>お知らせ: 本日18時からメンテナンスを実施します。</p>
```

- 字幕・文字起こしは聴覚障害だけでなく、音を出せない状況でも必要
- 重要な情報は音声・動画の外にもテキストで残す

> 参照: [WCAG 2.2 — 1.2.2 Captions (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/captions-prerecorded.html)

### 3. まぶしさの中でも読める

```css
/* 悪い例：コントラストが低い */
.meta {
  color: #bbb;
  background: #f5f5f5;
}

/* 良い例：屋外でも判別しやすいコントラスト */
.meta {
  color: #333;
  background: #fff;
}
```

```html
<!-- 悪い例：色だけでエラーを示す -->
<input style="border-color: red" />

<!-- 良い例：文言でも伝える -->
<input aria-invalid="true" aria-describedby="email-error" />
<p id="email-error">メールアドレスの形式が正しくありません</p>
```

- コントラスト確保はロービジョンにも、日差しの下のスマホ利用にも効く
- 色だけに依存しないのは、色覚特性にも状況的な見えにくさにも効く

> 参照:
> - [WCAG 2.2 — 1.4.3 Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)
> - [WCAG 2.2 — 1.4.1 Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html)

### 4. 気が散る・時間が足りない状況でも使える

```html
<!-- 悪い例：自動で進むカルーセルだけ -->
<div class="carousel" data-autoplay="true"></div>

<!-- 良い例：止められる・自分のペースで進める -->
<div class="carousel">
  <button type="button" aria-pressed="true">自動再生を停止</button>
  <button type="button">前へ</button>
  <button type="button">次へ</button>
</div>
```

```css
@media (prefers-reduced-motion: reduce) {
  .carousel__track {
    animation: none;
  }
}
```

- 移動中・割り込みの多い状況では、勝手に変わるUIは内容を取り逃しやすい
- `prefers-reduced-motion` は前庭障害にも、注意が散りやすい状況にも効く

> 参照:
> - [WCAG 2.2 — 2.2.2 Pause, Stop, Hide](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html)
> - [MDN — prefers-reduced-motion](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-reduced-motion)

### 5. 回線が細い・端末が弱い状況

```html
<!-- 悪い例：動画だけが本体 -->
<video src="huge-tutorial.mp4" autoplay></video>

<!-- 良い例：テキストと要約を先に届ける -->
<article>
  <h2>初期設定の手順</h2>
  <ol>
    <li>アカウントを作成する</li>
    <li>プロフィールを入力する</li>
    <li>通知設定を選ぶ</li>
  </ol>
  <video controls preload="none" poster="tutorial.jpg">
    <source src="tutorial.mp4" type="video/mp4" />
  </video>
</article>
```

- 情報の本体をテキストに置くと、低速回線や動画を再生できない状況でも使える
- `preload="none"` やポスター画像は、状況的な帯域制約への配慮にもなる

## 「同じ設計が効く」理由

- 永続・一時・状況で、必要な代替手段が重なることが多い
  - 音が聞こえない → 字幕（難聴でも騒音でも）
  - 精密操作が難しい → 大きなターゲット（運動障害でも片手でも）
  - 画面が見えにくい → 高いコントラスト（ロービジョンでも屋外でも）
- 実装を分岐して「障害ユーザー専用モード」を作るより、本線のUIを最初から広く使える形にする方が保守しやすい
- 一时的・状況的な制約を説明に使うと、関係者への納得感が出やすい（自分ごと化しやすい）
- ただし目的は「健常者の便利さ」だけではない — 永続的な障害のある人のアクセスを中心に据えた結果として、広い利用条件もカバーされる

## 設計・レビューで使う短い問い

- 片手でも主要な操作を完了できるか
- 音なしでも同じ情報が得られるか
- 屋外や低コントラスト環境でも判読できるか
- 自動で動く要素を止められるか
- 動画や重いアセットがなくても手順が追えるか
- 「今日の自分」が通勤中でも使えるか、を一度想像して確認する

## 関連して読むとよいテーマ

- Webアクセシビリティとは何か — なぜ重要なのか
- 障害の社会モデルとWebアクセシビリティ
- カラーコントラスト比の基準と確認方法
- prefers-reduced-motion — モーション設定への対応
- キーボードナビゲーションの基本

## まとめ

- 一時的・状況的な制約は、誰にでも起こり得るWebの利用条件
- Persona Spectrumの通り、永続的なニーズ向けの設計は一時・状況にも効きやすい
- 字幕、コントラスト、大きなターゲット、一時停止、テキスト代替は、その代表的な打ち手
- 「特別対応」ではなく本線の品質として実装すると、保守も説明もしやすい
- 自分ごと化は導入の入口にして、中心に置くべきは障害のある人の平等な利用

# 3.2.5 要求による変化（AAA）

## 達成基準の概要

- WCAG 2.2 達成基準 3.2.5「要求による変化」（Change on Request）
- レベル **AAA**（最高の適合レベル）
- 原則3「理解可能（Understandable）」> ガイドライン 3.2「予測可能」に属する
- **コンテキストの変化**は、ユーザーの要求によってのみ開始されること
- または、そのような自動変化を**オフにできる仕組み**があること
- 3.2.1（フォーカス時）・3.2.2（入力時）より広く、自動更新・リダイレクト・ポップアップなども対象

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.5 Change on Request](https://www.w3.org/TR/WCAG22/#change-on-request)
> - [Understanding SC 3.2.5 Change on Request](https://www.w3.org/WAI/WCAG22/Understanding/change-on-request.html)

## 達成基準の原文（要約）

- コンテキストの変化は、ユーザーの要求によってのみ開始されること
- または、そのような変化をオフにする仕組みが利用できること

> **参照**
> - [WCAG 2.2 — Success Criterion 3.2.5](https://www.w3.org/TR/WCAG22/#change-on-request)

## なぜ必要か

- 予期しない新しいウィンドウや画面更新は、方向感覚を失わせる
- 視覚障害のある人は、視覚的な変化に気づきにくい
- スクリーンリーダーは、ページ更新で仮想カーソルが先頭に戻ることがある
- 運動障害のある人は、勝手に開いたウィンドウを閉じにくい
- 認知の制約がある人は、自分が起こしていない変化に混乱しやすい
- 戻るボタンが期待どおり動かなくなると、さらに迷う

> **参照**
> - [Understanding SC 3.2.5 — Benefits](https://www.w3.org/WAI/WCAG22/Understanding/change-on-request.html#benefits)

## コンテキストの変化とは

- ユーザーエージェント・ビューポート・フォーカス・ページの意味が変わる大きな変化
- コンテンツが増えるだけでは、必ずしもコンテキストの変化ではない
- 同時に複数起きうる（リンクで新しいタブを開く＝コンテンツ変化＋ビューポート変化）

```text
コンテキストの変化の例：
  新しいウィンドウ／タブが開く
  ページ全体が自動リロードされる
  メインコンテンツが別内容に置き換わる
  フォーカスが別の場所へ飛ぶ
  フォームが自動送信される

コンテキストの変化ではないことが多い例：
  アコーディオンの開閉
  同じページ内のタブパネル切替
  ステータスメッセージの追加（フォーカスを奪わない）
```

> **参照**
> - [Understanding SC 3.2.5 — Key Terms: changes of context](https://www.w3.org/WAI/WCAG22/Understanding/change-on-request.html#dfn-changes-of-context)

## 3.2.1 / 3.2.2 との関係

| 達成基準 | レベル | 焦点 |
|---|---|---|
| 3.2.1 フォーカス時 | A | フォーカスしただけで変化しない |
| 3.2.2 入力時 | A | 設定を変えただけで変化しない（事前説明があれば可） |
| 3.2.5 要求による変化 | AAA | コンテキストの変化は**ユーザー要求のみ**、またはオフ可能 |

- 3.2.1 / 3.2.2 を満たしても、自動更新やページ読み込み時のポップアップがあれば 3.2.5 は満たさないことがある
- リンクをクリックして遷移するのは、ユーザー要求による変化なので許される
- リンクで新しいウィンドウを開く場合、**新しいウィンドウ自体**もユーザーが明示的に求めたものとして扱う必要がある

> **参照**
> - [wcag/3-2-1-on-focus.md — 3.2.1 フォーカス時](./3-2-1-on-focus.md)
> - [wcag/3-2-2-on-input.md — 3.2.2 入力時](./3-2-2-on-input.md)

## 手法1：自動更新の代わりに「更新」を要求する（G76）

- **G76**：自動更新せず、ユーザーが更新を要求できる仕組みを置く
- ニュース一覧・ダッシュボード・チャットなど、頻繁に変わる内容で使う
- 自動更新を残すなら、オフにできること

```html
<!-- 良い例：ユーザーが更新を要求する -->
<section aria-labelledby="news-heading">
  <h2 id="news-heading">最新ニュース</h2>
  <button type="button" id="refresh-news">この一覧を更新</button>
  <ul id="news-list"><!-- ニュース項目 --></ul>
</section>
```

```html
<!-- 悪い例：一定間隔でページ全体をリロード -->
<meta http-equiv="refresh" content="30">
```

```jsx
// 良い例：ポーリングを止められる
function LiveFeed() {
  const [auto, setAuto] = useState(false);

  useEffect(() => {
    if (!auto) return;
    const id = setInterval(fetchItems, 30000);
    return () => clearInterval(id);
  }, [auto]);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={auto}
          onChange={(e) => setAuto(e.target.checked)}
        />
        自動更新する
      </label>
      <button type="button" onClick={fetchItems}>
        今すぐ更新
      </button>
    </>
  );
}
```

> **参照**
> - [Technique G76: Providing a mechanism to request an update of the content instead of updating automatically](https://www.w3.org/WAI/WCAG22/Techniques/general/G76)

## 手法2：リダイレクトはサーバー側または即時（G110 / SVR1 / H76）

- 古い URL から新しい URL へ移すとき、ユーザーが気づかない**即時**のリダイレクトにする
- **SVR1**：サーバー側のリダイレクト（HTTP 3xx）が望ましい
- **H76**：クライアント側なら、遅延なしの即時リダイレクト
- 数秒待ってから移る `meta refresh` は、2.2.1 や 3.2.5 の失敗になりやすい（F40 / F41）

```http
<!-- 良い例：サーバー側リダイレクト -->
HTTP/1.1 301 Moved Permanently
Location: https://example.com/new-page
```

```html
<!-- 許容されうる：即時のクライアント側リダイレクト -->
<meta http-equiv="refresh" content="0; URL=https://example.com/new-page">
```

```html
<!-- 悪い例：遅延付きリダイレクト -->
<meta http-equiv="refresh" content="5; URL=https://example.com/new-page">
```

> **参照**
> - [Technique G110: Using an instant client-side redirect](https://www.w3.org/WAI/WCAG22/Techniques/general/G110)
> - [Technique SVR1: Implementing automatic redirects on the server side](https://www.w3.org/WAI/WCAG22/Techniques/server-side-script/SVR1)
> - [W3C QA Tip — Use standard redirects](https://www.w3.org/QA/Tips/reback)

## 手法3：新しいウィンドウはユーザー要求で、事前に示す（H83）

- **H83**：`target="_blank"` を使うなら、リンク文言で新しいウィンドウだと示す
- ページ読み込み時や、予期しないクリックでポップアップを開かない
- 新しいウィンドウが不要なら、同じタブで開く方が予測しやすい（G200）

```html
<!-- 良い例：ユーザーがリンクを活性化し、新しいウィンドウだと分かる -->
<a href="/help.html" target="_blank" rel="noopener noreferrer">
  ヘルプを表示（新しいウィンドウで開きます）
</a>
```

```html
<!-- 悪い例：ページ読み込みと同時にウィンドウを開く -->
<body onload="window.open('/promo.html')">
```

```html
<!-- 悪い例：新しいウィンドウだと分からない -->
<a href="/help.html" target="_blank">ヘルプ</a>
```

> **参照**
> - [Technique H83: Using the target attribute to open a new window on user request and indicating this in link text](https://www.w3.org/WAI/WCAG22/Techniques/html/H83)
> - [Technique G201: Giving users advanced warning when opening a new window](https://www.w3.org/WAI/WCAG22/Techniques/general/G201)

## 手法4：select の onchange でコンテキストを変えない（SCR19）

- 3.2.2 と同様、`<select>` の変更だけでページ遷移しない
- 同じページ内の連動更新なら、コンテキストの変化にならないことが多い
- 遷移が必要なら、送信ボタンで確定する

```html
<!-- 良い例：選択だけでは遷移しない -->
<label for="city">市区町村</label>
<select id="city" name="city">
  <option value="shibuya">渋谷区</option>
  <option value="shinjuku">新宿区</option>
</select>
<button type="submit">表示</button>
```

> **参照**
> - [Technique SCR19: Using an onchange event on a select element without causing a change of context](https://www.w3.org/WAI/WCAG22/Techniques/client-side-script/SCR19)

## よくある失敗例

| 失敗 | 内容 |
|---|---|
| F22 | ユーザーが求めていないウィンドウを開く |
| F52 | ページ読み込みと同時に新しいウィンドウを開く |
| F60 | 入力欄に文字を入れただけで新しいウィンドウが開く |
| F61 | ユーザーが止められない自動更新で、メインコンテンツが丸ごと変わる |
| F9 | フォーム要素からフォーカスを外しただけでコンテキストが変わる |
| F40 / F41 | 時間付きの `meta refresh` でリダイレクト／リロードする |

```html
<!-- 失敗例：入力しただけでウィンドウが開く（F60） -->
<input type="text" oninput="window.open('/suggest.html')">
```

```html
<!-- 失敗例：フォーカスを外しただけで送信（F9） -->
<input name="email" onblur="this.form.submit()">
```

```html
<!-- 失敗例：止められない自動更新（F61） -->
<script>
  setInterval(() => {
    location.reload();
  }, 10000);
</script>
```

```html
<!-- 改善例：更新はユーザー要求、またはオフ可能 -->
<button type="button" id="update-now">今すぐ更新</button>
<label>
  <input type="checkbox" id="auto-update">
  10秒ごとに自動更新する
</label>
```

> **参照**
> - [Failure F22: opening windows that are not requested by the user](https://www.w3.org/WAI/WCAG22/Techniques/failures/F22)
> - [Failure F52: opening a new window as soon as a new page is loaded](https://www.w3.org/WAI/WCAG22/Techniques/failures/F52)
> - [Failure F61: complete change of main content through an automatic update](https://www.w3.org/WAI/WCAG22/Techniques/failures/F61)

## 実装時の注意点

- 自動リロードより、「更新」ボタンやオフ可能な自動更新を優先する
- 古い URL は HTTP リダイレクトで移す
- `target="_blank"` には、リンク文言で新しいウィンドウだと書く
- 広告・アンケート・チャットの自動ポップアップは避ける
- スライドショーなど自動進行が本質の UI は、ユーザー設定で止められるようにする

## テスト・確認方法

### 手動テスト

- ページを開いただけで、新しいウィンドウや自動遷移が起きないか確認する
- 入力・フォーカス移動・選択だけで、送信や新規ウィンドウが起きないか確認する
- 自動更新があるなら、オフまたは手動更新ができるか確認する
- 新しいウィンドウを開くリンクに、その旨の表示があるか確認する
- 遅延付き `meta refresh` がないか確認する

```text
チェックリスト：
1. ページ読み込み時にポップアップが出ないか
2. 自動更新があるなら、オフまたは「更新」要求ができるか
3. リダイレクトは即時（できればサーバー側）か
4. 新しいウィンドウはユーザー操作で開き、文言で示されているか
5. 入力・blur・select だけでコンテキストが変わらないか
6. meta refresh の遅延リロードがないか
```

```html
<!-- DevTools / ソースで確認する手がかり -->
<meta http-equiv="refresh">
<script>window.open(...)</script>
<body onload="...">
```

> **参照**
> - [ACT Rule — Meta element has no refresh delay](https://www.w3.org/WAI/standards-guidelines/act/rules/bc659a/)
> - [a11y/manual-testing-checklist.md — 手動テストチェックリスト](../a11y/manual-testing-checklist.md)

## まとめ

- 3.2.5 は、コンテキストの変化を**ユーザーの要求だけ**に限定する達成基準（レベル AAA）
- オフにできる仕組みがあれば、自動変化も許されうる
- 自動更新は「更新」ボタンへ、リダイレクトはサーバー側へ、新しいウィンドウは明示的な操作へ寄せる
- ページ読み込み時のポップアップや、止められない自動リロードは典型的な失敗
- 3.2.1 / 3.2.2 より広く、サイト全体の予測可能性を高める

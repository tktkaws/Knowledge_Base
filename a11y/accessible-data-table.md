# アクセシブルなデータテーブルの実装

## データテーブルとは

- 行と列の交点で意味を持つデータを表形式で示すUI
- スクリーンリーダーはヘッダーとセルの関連を読み上げて理解する
- レイアウト用途の表ではなく、**データの関係を伝える表**が対象
- 可能な限りネイティブの `<table>` を使い、見た目だけのために `div` で表を組まない

> 参照: [MDN — HTML table accessibility](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Table_accessibility)

## テーブル / レイアウト / グリッドの使い分け

| UI | 用途 | マークアップ |
|---|---|---|
| データテーブル | 閲覧・比較が中心のデータ | `<table>` |
| レイアウト用の表 | 見た目の配置だけ | **使わない**（Flex / Grid へ） |
| Grid（APG） | セル単位の編集・スプレッドシート操作 | `role="grid"` など |

- セルをクリックして編集する・矢印キーでセル移動するなら Grid パターンを検討
- 並べ替えやフィルタはあるが、基本は閲覧なら通常のテーブルで足りる

## 基本構造

| 要素 / 属性 | 役割 |
|---|---|
| `<caption>` | 表のタイトル・概要 |
| `<thead>` / `<tbody>` / `<tfoot>` | 見出し行・本体・フッタの区分 |
| `<th>` | 見出しセル |
| `<td>` | データセル |
| `scope="col"` / `scope="row"` | 見出しが列向きか行向きか |
| `scope="colgroup"` / `scope="rowgroup"` | 複数列 / 行にまたがる見出し |

### シンプルな表

```html
<table>
  <caption>2026年4月の売上</caption>
  <thead>
    <tr>
      <th scope="col">商品名</th>
      <th scope="col">個数</th>
      <th scope="col">売上</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">ノートPC</th>
      <td>12</td>
      <td>1,440,000円</td>
    </tr>
    <tr>
      <th scope="row">マウス</th>
      <td>48</td>
      <td>96,000円</td>
    </tr>
  </tbody>
</table>
```

### ポイント

- `<caption>` は `<table>` 直後に置く
- 列見出し・行見出しはどちらも `<th>` + `scope`
- `scope` を付けると、スクリーンリーダーが見出しとデータの対応を誤りにくい
- 最初の列が行の名前なら `scope="row"` にする

> 参照: [WebAIM — Creating Accessible Tables](https://webaim.org/techniques/tables/data)

## 複雑な表（複数レベルの見出し）

```html
<table>
  <caption>カテゴリ別 販売数（2026年）</caption>
  <thead>
    <tr>
      <th scope="col" rowspan="2">地域</th>
      <th scope="colgroup" colspan="2">衣類</th>
      <th scope="colgroup" colspan="2">電化製品</th>
    </tr>
    <tr>
      <th scope="col">上衣</th>
      <th scope="col">下衣</th>
      <th scope="col">PC</th>
      <th scope="col">周辺機器</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">東京</th>
      <td>120</td>
      <td>80</td>
      <td>40</td>
      <td>95</td>
    </tr>
  </tbody>
</table>
```

- 複数列を束ねる見出しには `scope="colgroup"`
- 単純な表では `scope` で十分なことが多い
- さらに複雑な関連付けが必要な場合のみ `id` + `headers` を使う

### id / headers による関連付け

```html
<table>
  <caption>社員の勤務情報</caption>
  <thead>
    <tr>
      <th id="name">氏名</th>
      <th id="dept">部署</th>
      <th id="remote">リモート日数</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="yamada" headers="name">山田</th>
      <td headers="yamada dept">開発</td>
      <td headers="yamada remote">3</td>
    </tr>
  </tbody>
</table>
```

- 管理コストが高いため、通常は `scope` を優先する
- 使う場合は、各データセルがどの見出しに紐づくかを明示する

## ソート可能なテーブル

- 並べ替え可能な列見出しは、見出しテキストを `<button>` で包む
- 現在ソート中の列の `<th>` にだけ `aria-sort` を付ける
- ソートアイコンは装飾なので `aria-hidden="true"`
- 並べ替え可能である旨は `<caption>` に短い説明を入れてよい（ボタンごとに繰り返さない）

> 参照: [APG — Sortable Table Example](https://www.w3.org/WAI/ARIA/apg/patterns/table/examples/sortable-table/)

```html
<table id="members-table">
  <caption>
    プロジェクトメンバー一覧
    <span class="visually-hidden">
      列見出しのボタンで並べ替えできます。
    </span>
  </caption>
  <thead>
    <tr>
      <th scope="col" aria-sort="ascending">
        <button type="button">
          氏名
          <span aria-hidden="true">▲</span>
        </button>
      </th>
      <th scope="col">
        <button type="button">
          役割
          <span aria-hidden="true">♢</span>
        </button>
      </th>
      <th scope="col">メール</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">伊藤</th>
      <td>デザイナー</td>
      <td>ito@example.com</td>
    </tr>
    <tr>
      <th scope="row">鈴木</th>
      <td>エンジニア</td>
      <td>suzuki@example.com</td>
    </tr>
  </tbody>
</table>

<div id="sort-status" role="status" aria-live="polite" class="visually-hidden"></div>
```

### aria-sort の値

| 値 | 意味 |
|---|---|
| `ascending` | 昇順 |
| `descending` | 降順 |
| `other` | 昇順・降順以外の並び |
| 属性なし / `none` | その列ではソートされていない |

- `aria-sort` は**同時に1列だけ**に付ける
- 別の列でソートしたら、前の列から属性を外して移す

### Vanilla JS でのソート実装（抜粋）

```js
function sortTable(table, columnIndex, direction) {
  const tbody = table.tBodies[0];
  const rows = [...tbody.rows];

  rows.sort((a, b) => {
    const aText = a.cells[columnIndex].textContent.trim();
    const bText = b.cells[columnIndex].textContent.trim();
    return direction === 'ascending'
      ? aText.localeCompare(bText, 'ja')
      : bText.localeCompare(aText, 'ja');
  });

  rows.forEach((row) => tbody.appendChild(row));
}

document.querySelectorAll('#members-table thead button').forEach((button, columnIndex) => {
  button.addEventListener('click', () => {
    const th = button.closest('th');
    const table = button.closest('table');
    const current = th.getAttribute('aria-sort');
    const next = current === 'ascending' ? 'descending' : 'ascending';

    table.querySelectorAll('th[aria-sort]').forEach((header) => {
      header.removeAttribute('aria-sort');
    });
    th.setAttribute('aria-sort', next);
    sortTable(table, columnIndex, next);

    document.getElementById('sort-status').textContent =
      `${button.textContent.trim()}を${next === 'ascending' ? '昇順' : '降順'}に並べ替えました`;
  });
});
```

## React での実装例

```jsx
import { useMemo, useState } from 'react';

function DataTable({ caption, columns, rows }) {
  const [sortKey, setSortKey] = useState(columns[0]?.key ?? null);
  const [direction, setDirection] = useState('ascending');

  const sortedRows = useMemo(() => {
    if (!sortKey) return rows;
    const sorted = [...rows].sort((a, b) =>
      String(a[sortKey]).localeCompare(String(b[sortKey]), 'ja')
    );
    return direction === 'ascending' ? sorted : sorted.reverse();
  }, [rows, sortKey, direction]);

  const toggleSort = (key) => {
    if (sortKey === key) {
      setDirection((current) =>
        current === 'ascending' ? 'descending' : 'ascending'
      );
      return;
    }
    setSortKey(key);
    setDirection('ascending');
  };

  return (
    <>
      <table>
        <caption>{caption}</caption>
        <thead>
          <tr>
            {columns.map((column) => (
              <th
                key={column.key}
                scope="col"
                aria-sort={sortKey === column.key ? direction : undefined}
              >
                {column.sortable ? (
                  <button type="button" onClick={() => toggleSort(column.key)}>
                    {column.label}
                  </button>
                ) : (
                  column.label
                )}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {sortedRows.map((row) => (
            <tr key={row.id}>
              {columns.map((column, index) =>
                index === 0 ? (
                  <th key={column.key} scope="row">
                    {row[column.key]}
                  </th>
                ) : (
                  <td key={column.key}>{row[column.key]}</td>
                )
              )}
            </tr>
          ))}
        </tbody>
      </table>
      <div role="status" aria-live="polite" className="visually-hidden">
        {sortKey
          ? `${columns.find((c) => c.key === sortKey)?.label}を${
              direction === 'ascending' ? '昇順' : '降順'
            }に並べ替えました`
          : ''}
      </div>
    </>
  );
}
```

## レスポンシブ対応

- 横スクロールさせる場合は、スクロール領域にキーボードで到達できること
- カード型に組み替える場合も、見出しとデータの関係が失われないこと
- `display: block` で表構造を壊すと、スクリーンリーダーのテーブルナビゲーションが使えなくなる

```html
<div class="table-scroll" tabindex="0" role="region" aria-label="売上表">
  <table>...</table>
</div>
```

```css
.table-scroll {
  overflow-x: auto;
  max-width: 100%;
}
```

- スクロールコンテナに名前を付け、フォーカス可能にするとキーボードでも横スクロールしやすい

## 空セル・数値・リンク

```html
<!-- 空であることが意味を持つなら、空のままより説明を入れる -->
<td><span class="visually-hidden">データなし</span></td>

<!-- 数値は揃え方と単位を明確に -->
<td>1,200円</td>

<!-- 行全体をクリック可能にするなら、セル内にリンクやボタンを置く -->
<td><a href="/users/1">詳細</a></td>
```

- 行全体を `onclick` だけにするとキーボード操作が困難
- 操作はセル内のリンク / ボタンで提供する

## よくある間違い

### 1. div で表を組み立てる

```html
<!-- 間違い：表としての意味がない -->
<div class="row">
  <div class="cell">氏名</div>
  <div class="cell">年齢</div>
</div>

<!-- 正解 -->
<table>
  <tr>
    <th scope="col">氏名</th>
    <th scope="col">年齢</th>
  </tr>
</table>
```

### 2. 見出しなのに td を使う

```html
<!-- 間違い -->
<tr>
  <td><strong>商品名</strong></td>
  <td><strong>価格</strong></td>
</tr>

<!-- 正解 -->
<tr>
  <th scope="col">商品名</th>
  <th scope="col">価格</th>
</tr>
```

### 3. caption がなく、周囲の見出しとも紐付いていない

```html
<!-- 間違い：何の表か分からない -->
<table>...</table>

<!-- 正解：caption か、見出しと aria-labelledby -->
<table aria-labelledby="sales-heading">
  <caption class="visually-hidden">2026年4月の売上</caption>
  ...
</table>
```

### 4. ソートを見た目のアイコンだけで伝える

```html
<!-- 間違い：aria-sort なし、ボタンでもない -->
<th scope="col" onclick="sort()">氏名 ▼</th>

<!-- 正解：button + aria-sort -->
<th scope="col" aria-sort="descending">
  <button type="button">氏名 <span aria-hidden="true">▼</span></button>
</th>
```

### 5. レイアウトのために table を使う

- 余白や2カラム配置目的の `<table>` は、支援技術に誤った表構造を伝える
- CSS のレイアウト手段を使う

### 6. すべての列に aria-sort を同時に付ける

- 現在ソート中の列だけに付ける
- 未ソート列は属性を付けない（またはソート不可ならボタン自体を置かない）

## 実装チェックリスト

- [ ] データ表は `<table>` でマークアップしている
- [ ] `<caption>` または同等の名前がある
- [ ] 列 / 行の見出しが `<th>` である
- [ ] `scope="col"` / `scope="row"` が適切に付いている
- [ ] レイアウト目的で `<table>` を使っていない
- [ ] ソート可能列は `<button>` で操作できる
- [ ] ソート中の列にのみ `aria-sort` がある
- [ ] ソート結果の変化を必要に応じてライブリージョンで伝えている
- [ ] 行の操作はリンク / ボタンなどフォーカス可能要素で提供している
- [ ] 横スクロール時もキーボードで内容に到達できる

## まとめ

- データテーブルの基本はネイティブHTML（`caption` / `th` / `scope`）
- 複雑な見出しは `colgroup` / `rowgroup`、さらに必要なら `id` + `headers`
- ソートは見出し内ボタンと、現在列だけの `aria-sort` で伝える
- 見た目のための `div` 表やレイアウト用 `table` は避ける
- セル編集やセル単位移動が必要なら、通常のテーブルではなく Grid パターンを検討する

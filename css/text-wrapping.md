# テキストの折り返し制御 — word-break, overflow-wrap, hyphens

## テキスト折り返しの基本

- ブラウザはデフォルトで、行ボックスの幅を超えるテキストを自動的に折り返す
- 折り返しが起きるポイントを**ソフトラップ機会（soft wrap opportunity）**と呼ぶ
- 通常のソフトラップ機会はスペース・ハイフンなどの区切り文字の位置
- 日本語・中国語（CJK）はほぼすべての文字間にソフトラップ機会がある
- 長い英単語やURLなど、ソフトラップ機会がない文字列はコンテナからはみ出す

> 参照: [MDN - Wrapping and breaking text](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_text/Wrapping_breaking_text)

## overflow-wrap（旧 word-wrap）

- **行内に収まらない長い単語をどう扱うか**を制御するプロパティ
- 通常の折り返しポイントで折り返しても収まらない場合にのみ作用する
- `word-wrap` は `overflow-wrap` の旧名（レガシーエイリアス）

| 値 | 動作 |
|---|---|
| `normal` | 通常の折り返しポイントでのみ折り返す（デフォルト） |
| `break-word` | 通常の折り返しポイントで収まらない場合、単語の途中でも折り返す |
| `anywhere` | `break-word` と同様だが、min-contentサイズの計算にも影響する |

```css
/* 問題：長いURLがコンテナからはみ出す */
.container {
  width: 300px;
  border: 1px solid #ccc;
}

/* 悪い例：何も指定しない → はみ出す */
.container p {
  overflow-wrap: normal;
  /* https://very-long-domain-name.example.com/path/to/resource がはみ出す */
}

/* 良い例：収まらない場合に単語途中で折り返す */
.container p {
  overflow-wrap: break-word;
  /* 長いURLも途中で折り返されてコンテナ内に収まる */
}
```

### break-word と anywhere の違い

```css
/* break-word: min-contentサイズの計算に影響しない */
.box-break-word {
  overflow-wrap: break-word;
  width: min-content;
  /* → 最も長い単語の幅がmin-contentになる */
}

/* anywhere: min-contentサイズの計算でも分割を考慮する */
.box-anywhere {
  overflow-wrap: anywhere;
  width: min-content;
  /* → 1文字単位まで縮小可能になる */
}
```

- 通常は `break-word` を使えば十分
- `anywhere` はFlexboxやGridでの自動サイズ計算に影響するため注意が必要

> 参照: [MDN - overflow-wrap](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap)

## word-break

- **単語の途中での改行ルール**を制御するプロパティ
- `overflow-wrap` とは異なり、収まるかどうかに関係なく改行ルールを変更する

| 値 | 動作 |
|---|---|
| `normal` | デフォルトの改行ルールに従う |
| `break-all` | すべてのテキストで任意の2文字間に改行を許可する（CJKと同じルールを適用） |
| `keep-all` | CJKテキストでの文字間改行を禁止する（スペース・ハイフンでのみ改行） |
| `auto-phrase` | ブラウザが自然な句の区切りを推測して改行する（実験的） |

```css
/* 英語の長い単語が多い場合 */
.technical-text {
  word-break: break-all;
  /* → あらゆる文字間で改行可能。行末のスペースが均等になる */
  /* ただし英単語が読みにくくなるデメリットがある */
}

/* 日本語テキストを英語と同じルールで折り返す */
.japanese-text {
  word-break: keep-all;
  /* → 日本語の文字間で改行されなくなる */
  /* スペースや句読点でのみ改行される */
  /* ※日本語はスペースがほぼないため、はみ出しやすくなる */
}
```

### word-break と overflow-wrap の違い

```css
/* overflow-wrap: break-word */
/* → まず通常の折り返しポイントを試す */
/* → 収まらない場合だけ単語途中で折り返す */
.gentle {
  overflow-wrap: break-word;
}

/* word-break: break-all */
/* → すべての文字間を折り返しポイントにする */
/* → 行末を揃えるが、単語の読みやすさは犠牲になる */
.aggressive {
  word-break: break-all;
}
```

```
overflow-wrap: break-word の挙動:
┌─────────────────────┐
│ This is a           │
│ VeryLongWordThatDoe │
│ sNotFitInOneLine    │  ← 長い単語だけ途中で折り返す
│ and normal text.    │
└─────────────────────┘

word-break: break-all の挙動:
┌─────────────────────┐
│ This is a VeryLongW │
│ ordThatDoesNotFitIn │  ← すべての文字間で折り返す
│ OneLine and normal  │
│ text.               │
└─────────────────────┘
```

- **ほとんどのケースでは `overflow-wrap: break-word` が適切**
- `word-break: break-all` は表のセルなど、限られたスペースで英数字を詰める場合に有用

> 参照: [MDN - word-break](https://developer.mozilla.org/en-US/docs/Web/CSS/word-break)

## hyphens

- 単語の途中で折り返す際に**ハイフン（-）を挿入するかどうか**を制御する
- 英語など、ハイフネーションルールが定義されている言語で効果的
- `lang` 属性が正しく設定されている必要がある

| 値 | 動作 |
|---|---|
| `none` | ハイフネーションしない（手動の `&shy;` も無視） |
| `manual` | `&shy;`（ソフトハイフン）の位置でのみハイフネーション（デフォルト） |
| `auto` | ブラウザが辞書に基づいて自動的にハイフネーション |

```html
<!-- lang属性が必要 -->
<html lang="en">
```

```css
/* 良い例：英語テキストの自動ハイフネーション */
.article-body {
  hyphens: auto;
  /* → ブラウザが英語の辞書に基づいて適切な位置でハイフン挿入 */
  /* 例: "extraordinarily" → "extraor-" + "dinarily" */
}

/* 手動でハイフネーション位置を指定する場合 */
.manual-hyphen {
  hyphens: manual;
}
```

```html
<!-- &shy; でハイフネーション可能な位置を指定 -->
<p class="manual-hyphen">
  This is an extra&shy;ordinarily long word.
  <!-- 折り返しが必要な場合 "extra-" + "ordinarily" になる -->
  <!-- 折り返しが不要な場合は何も表示されない -->
</p>
```

### 日本語での hyphens

- 日本語にはハイフネーションの概念がないため、`hyphens: auto` は効果がない
- 日本語テキストの折り返しは `word-break` で制御する

> 参照: [MDN - hyphens](https://developer.mozilla.org/en-US/docs/Web/CSS/hyphens)

## white-space との関係

- `white-space` プロパティはテキストの折り返し・空白・改行の処理をまとめて制御する
- 折り返し制御に直接関わる重要なプロパティ

| 値 | 空白の圧縮 | 改行文字 | 自動折り返し |
|---|---|---|---|
| `normal` | する | 無視 | する |
| `nowrap` | する | 無視 | **しない** |
| `pre` | しない | 反映 | しない |
| `pre-wrap` | しない | 反映 | する |
| `pre-line` | する | 反映 | する |
| `break-spaces` | しない | 反映 | する（末尾空白でも折り返す） |

```css
/* テキストを折り返さない（1行に収める） */
.single-line {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis; /* はみ出た部分を「...」で表示 */
}

/* コードブロック：空白・改行をそのまま表示、かつ折り返す */
pre code {
  white-space: pre-wrap;
}
```

> 参照: [MDN - white-space](https://developer.mozilla.org/en-US/docs/Web/CSS/white-space)

## text-overflow でのはみ出し表示

- テキストがはみ出す場合に「...」（省略記号）を表示する一般的なパターン

```css
/* 1行テキストの省略 */
.truncate {
  white-space: nowrap;      /* 折り返しを禁止 */
  overflow: hidden;         /* はみ出しを非表示 */
  text-overflow: ellipsis;  /* はみ出し部分を「...」に */
}

/* 複数行テキストの省略（Webkit系の機能だが広くサポートされている） */
.line-clamp {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;   /* 3行目以降を省略 */
  overflow: hidden;
}
```

> 参照: [MDN - text-overflow](https://developer.mozilla.org/en-US/docs/Web/CSS/text-overflow)

## 実践的な組み合わせパターン

### 一般的なWebサイトの本文

```css
.body-text {
  overflow-wrap: break-word; /* 長いURLなどのはみ出し防止 */
  /* word-break はデフォルト(normal)のまま */
  /* → 日本語は文字間で、英語はスペースで折り返す */
}
```

### ユーザー入力テキストの表示

```css
/* ユーザーが入力した予測不能なテキストに対応 */
.user-content {
  overflow-wrap: break-word; /* 長い文字列のはみ出し防止 */
  word-break: normal;        /* 通常の折り返しルール */
  white-space: pre-wrap;     /* 改行・空白をそのまま反映 */
}
```

### テーブルセル内のテキスト

```css
/* テーブルの列幅が固定の場合 */
.data-table {
  table-layout: fixed;
}

.data-table td {
  overflow-wrap: break-word;
  word-break: break-all; /* 狭いセルで英数字を強制折り返し */
}
```

### 英語コンテンツのハイフネーション

```css
.english-article {
  hyphens: auto;
  overflow-wrap: break-word; /* ハイフネーション辞書にない単語のフォールバック */
}
```

## まとめ

- **`overflow-wrap: break-word`** — 最も汎用的。長い文字列がはみ出す場合だけ単語途中で折り返す
- **`word-break: break-all`** — すべての文字間で折り返す。テーブルセルなど狭い領域向け
- **`word-break: keep-all`** — CJKの文字間折り返しを禁止する
- **`hyphens: auto`** — 英語テキストの折り返し時にハイフンを挿入する。`lang` 属性が必要
- **`white-space: nowrap`** + **`text-overflow: ellipsis`** — テキストを1行に収めて省略表示
- ほとんどのケースでは `overflow-wrap: break-word` だけで十分

## 参照元

- [MDN - overflow-wrap](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap)
- [MDN - word-break](https://developer.mozilla.org/en-US/docs/Web/CSS/word-break)
- [MDN - hyphens](https://developer.mozilla.org/en-US/docs/Web/CSS/hyphens)
- [MDN - white-space](https://developer.mozilla.org/en-US/docs/Web/CSS/white-space)
- [MDN - text-overflow](https://developer.mozilla.org/en-US/docs/Web/CSS/text-overflow)
- [MDN - Wrapping and breaking text](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_text/Wrapping_breaking_text)

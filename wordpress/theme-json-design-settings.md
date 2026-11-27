# theme.jsonによるデザイン設定とスタイル管理

## 概要

- `theme.json` はテーマのデザイントークン（色・フォント・余白・レイアウト）を一元定義するファイル
- WordPress 5.8で導入、WordPress 6.x系で継続的に拡張
- settings でエディタの選択肢を制御し、styles でデフォルトのスタイルを適用する
- CSSカスタムプロパティ（CSS変数）が自動生成される

### 参照元

- [WordPress Theme JSON Reference](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/)

---

## settings — エディタの選択肢を制御する

### settingsの主要カテゴリ

| カテゴリ | 説明 |
|---------|------|
| `color` | カラーパレット、グラデーション、デュオトーン |
| `typography` | フォントファミリー、フォントサイズ、行の高さ |
| `spacing` | マージン、パディング、gap、余白の単位 |
| `layout` | コンテンツ幅、ワイド幅 |
| `border` | ボーダーの色・幅・角丸 |
| `shadow` | ボックスシャドウプリセット |
| `dimensions` | 最小の高さ |
| `position` | スティッキーポジション |

### 参照元

- [WordPress Theme JSON – Settings](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/#settings)

---

## カラーパレットの設定

### カスタムパレットの定義

```json
{
  "version": 3,
  "settings": {
    "color": {
      "palette": [
        {
          "slug": "primary",
          "color": "#1e40af",
          "name": "プライマリ"
        },
        {
          "slug": "secondary",
          "color": "#9333ea",
          "name": "セカンダリ"
        },
        {
          "slug": "accent",
          "color": "#f59e0b",
          "name": "アクセント"
        },
        {
          "slug": "base",
          "color": "#ffffff",
          "name": "ベース"
        },
        {
          "slug": "contrast",
          "color": "#1f2937",
          "name": "コントラスト"
        }
      ]
    }
  }
}
```

### 自動生成されるCSSカスタムプロパティ

```css
/* theme.jsonの palette から自動生成される */
body {
  --wp--preset--color--primary: #1e40af;
  --wp--preset--color--secondary: #9333ea;
  --wp--preset--color--accent: #f59e0b;
  --wp--preset--color--base: #ffffff;
  --wp--preset--color--contrast: #1f2937;
}
```

### 色に関するオプション制御

```json
{
  "settings": {
    "color": {
      "palette": [],
      "custom": false,
      "customGradient": false,
      "defaultPalette": false,
      "defaultGradients": false,
      "defaultDuotone": false,
      "background": true,
      "text": true,
      "link": true
    }
  }
}
```

| プロパティ | 説明 |
|-----------|------|
| `custom` | カラーピッカーでの自由選択を許可するか |
| `customGradient` | カスタムグラデーションの作成を許可するか |
| `defaultPalette` | WordPressデフォルトのカラーパレットを表示するか |
| `background` | 背景色の設定をUIに表示するか |
| `text` | テキスト色の設定をUIに表示するか |
| `link` | リンク色の設定をUIに表示するか |

### 参照元

- [WordPress Theme JSON – Color Settings](https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/#color-properties)

---

## タイポグラフィの設定

### フォントファミリーの定義

```json
{
  "settings": {
    "typography": {
      "fontFamilies": [
        {
          "slug": "body",
          "name": "本文",
          "fontFamily": "'Noto Sans JP', sans-serif",
          "fontFace": [
            {
              "fontFamily": "Noto Sans JP",
              "fontWeight": "400",
              "fontStyle": "normal",
              "src": ["file:./assets/fonts/NotoSansJP-Regular.woff2"]
            },
            {
              "fontWeight": "700",
              "fontFamily": "Noto Sans JP",
              "fontStyle": "normal",
              "src": ["file:./assets/fonts/NotoSansJP-Bold.woff2"]
            }
          ]
        },
        {
          "slug": "heading",
          "name": "見出し",
          "fontFamily": "'Noto Serif JP', serif"
        }
      ]
    }
  }
}
```

### ローカルフォントの配置

```
my-block-theme/
├── assets/
│   └── fonts/
│       ├── NotoSansJP-Regular.woff2
│       └── NotoSansJP-Bold.woff2
└── theme.json
```

### フォントサイズの定義

```json
{
  "settings": {
    "typography": {
      "fontSizes": [
        {
          "slug": "small",
          "size": "0.875rem",
          "name": "小"
        },
        {
          "slug": "medium",
          "size": "1rem",
          "name": "中"
        },
        {
          "slug": "large",
          "size": "1.25rem",
          "name": "大"
        },
        {
          "slug": "x-large",
          "size": "1.5rem",
          "name": "特大"
        }
      ],
      "customFontSize": false
    }
  }
}
```

### Fluid Typography（可変フォントサイズ）

- `clamp()` を使ってビューポート幅に応じてフォントサイズを可変にする
- WordPress 6.1以降で対応

```json
{
  "settings": {
    "typography": {
      "fluid": true,
      "fontSizes": [
        {
          "slug": "large",
          "size": "1.25rem",
          "name": "大",
          "fluid": {
            "min": "1rem",
            "max": "1.5rem"
          }
        },
        {
          "slug": "x-large",
          "size": "2rem",
          "name": "特大",
          "fluid": {
            "min": "1.5rem",
            "max": "2.5rem"
          }
        }
      ]
    }
  }
}
```

```css
/* fluid設定から自動生成されるCSS */
--wp--preset--font-size--large: clamp(1rem, 0.875rem + 0.5vw, 1.5rem);
--wp--preset--font-size--x-large: clamp(1.5rem, 1.25rem + 1vw, 2.5rem);
```

### タイポグラフィのオプション制御

```json
{
  "settings": {
    "typography": {
      "customFontSize": false,
      "dropCap": false,
      "lineHeight": true,
      "letterSpacing": true,
      "textDecoration": true,
      "textTransform": true,
      "writingMode": true
    }
  }
}
```

### 参照元

- [WordPress Theme JSON – Typography Settings](https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/#typography-properties)

---

## スペーシング（余白）の設定

### スペースサイズの定義

```json
{
  "settings": {
    "spacing": {
      "spacingSizes": [
        { "slug": "10", "size": "0.25rem", "name": "1" },
        { "slug": "20", "size": "0.5rem",  "name": "2" },
        { "slug": "30", "size": "1rem",    "name": "3" },
        { "slug": "40", "size": "1.5rem",  "name": "4" },
        { "slug": "50", "size": "2rem",    "name": "5" },
        { "slug": "60", "size": "3rem",    "name": "6" },
        { "slug": "70", "size": "4rem",    "name": "7" },
        { "slug": "80", "size": "5rem",    "name": "8" }
      ],
      "spacingScale": {
        "steps": 0
      },
      "units": ["px", "rem", "em", "%", "vw"],
      "margin": true,
      "padding": true,
      "blockGap": true,
      "customSpacingSize": false
    }
  }
}
```

### 自動生成されるCSSカスタムプロパティ

```css
body {
  --wp--preset--spacing--10: 0.25rem;
  --wp--preset--spacing--20: 0.5rem;
  --wp--preset--spacing--30: 1rem;
  --wp--preset--spacing--40: 1.5rem;
  --wp--preset--spacing--50: 2rem;
  /* ... */
}
```

### 参照元

- [WordPress Theme JSON – Spacing Settings](https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/#spacing-properties)

---

## レイアウトの設定

### コンテンツ幅とワイド幅

```json
{
  "settings": {
    "layout": {
      "contentSize": "720px",
      "wideSize": "1200px"
    }
  }
}
```

- `contentSize`: 通常のコンテンツ幅（段落、見出しなどのデフォルト幅）
- `wideSize`: ワイド幅（「幅広」配置を選んだブロックの幅）
- 「全幅」配置は画面幅いっぱいになる

```
|← ─────────── 画面幅 ──────────── →|
|                                     |
|   |← ── wideSize (1200px) ── →|   |
|   |                             |   |
|   |  |← contentSize (720px) →| |   |
|   |  |    通常のコンテンツ    | |   |
|   |  |                       | |   |
|   |     ワイド幅のブロック     |   |
|                                     |
|          全幅のブロック              |
```

### 参照元

- [WordPress Theme JSON – Layout Settings](https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/#layout)

---

## styles — グローバルスタイルの定義

### 基本的なスタイル設定

```json
{
  "styles": {
    "color": {
      "background": "var(--wp--preset--color--base)",
      "text": "var(--wp--preset--color--contrast)"
    },
    "typography": {
      "fontFamily": "var(--wp--preset--font-family--body)",
      "fontSize": "var(--wp--preset--font-size--medium)",
      "lineHeight": "1.75"
    },
    "spacing": {
      "blockGap": "1.5rem",
      "padding": {
        "top": "0",
        "right": "var(--wp--preset--spacing--30)",
        "bottom": "0",
        "left": "var(--wp--preset--spacing--30)"
      }
    }
  }
}
```

### リンクスタイル

```json
{
  "styles": {
    "elements": {
      "link": {
        "color": {
          "text": "var(--wp--preset--color--primary)"
        },
        "typography": {
          "textDecoration": "underline"
        },
        ":hover": {
          "color": {
            "text": "var(--wp--preset--color--secondary)"
          },
          "typography": {
            "textDecoration": "none"
          }
        },
        ":focus": {
          "outline": {
            "color": "var(--wp--preset--color--primary)",
            "width": "2px",
            "style": "solid",
            "offset": "2px"
          }
        }
      }
    }
  }
}
```

### 見出しスタイル

```json
{
  "styles": {
    "elements": {
      "heading": {
        "typography": {
          "fontFamily": "var(--wp--preset--font-family--heading)",
          "fontWeight": "700",
          "lineHeight": "1.4"
        },
        "color": {
          "text": "var(--wp--preset--color--contrast)"
        }
      },
      "h1": {
        "typography": {
          "fontSize": "2.5rem"
        }
      },
      "h2": {
        "typography": {
          "fontSize": "2rem"
        },
        "spacing": {
          "margin": {
            "top": "var(--wp--preset--spacing--60)",
            "bottom": "var(--wp--preset--spacing--30)"
          }
        }
      },
      "h3": {
        "typography": {
          "fontSize": "1.5rem"
        }
      }
    }
  }
}
```

### ボタンスタイル

```json
{
  "styles": {
    "elements": {
      "button": {
        "color": {
          "background": "var(--wp--preset--color--primary)",
          "text": "var(--wp--preset--color--base)"
        },
        "typography": {
          "fontWeight": "600",
          "fontSize": "var(--wp--preset--font-size--medium)"
        },
        "border": {
          "radius": "4px",
          "width": "0"
        },
        "spacing": {
          "padding": {
            "top": "0.75rem",
            "right": "1.5rem",
            "bottom": "0.75rem",
            "left": "1.5rem"
          }
        },
        ":hover": {
          "color": {
            "background": "var(--wp--preset--color--secondary)"
          }
        }
      }
    }
  }
}
```

### 参照元

- [WordPress Theme JSON – Styles](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/#styles)

---

## ブロック単位のスタイル設定

### settings でブロック個別の設定を制御

```json
{
  "settings": {
    "blocks": {
      "core/paragraph": {
        "color": {
          "custom": false
        }
      },
      "core/heading": {
        "typography": {
          "customFontSize": false
        }
      },
      "core/button": {
        "color": {
          "custom": true
        }
      }
    }
  }
}
```

### styles でブロック個別のデフォルトスタイルを定義

```json
{
  "styles": {
    "blocks": {
      "core/separator": {
        "color": {
          "text": "var(--wp--preset--color--accent)"
        },
        "border": {
          "width": "2px"
        }
      },
      "core/quote": {
        "border": {
          "left": {
            "color": "var(--wp--preset--color--primary)",
            "width": "4px",
            "style": "solid"
          }
        },
        "spacing": {
          "padding": {
            "left": "var(--wp--preset--spacing--30)"
          }
        },
        "typography": {
          "fontStyle": "italic"
        }
      },
      "core/code": {
        "color": {
          "background": "#f5f5f5",
          "text": "#333333"
        },
        "typography": {
          "fontFamily": "'Fira Code', monospace",
          "fontSize": "0.875rem"
        },
        "spacing": {
          "padding": {
            "top": "var(--wp--preset--spacing--30)",
            "right": "var(--wp--preset--spacing--30)",
            "bottom": "var(--wp--preset--spacing--30)",
            "left": "var(--wp--preset--spacing--30)"
          }
        },
        "border": {
          "radius": "4px"
        }
      }
    }
  }
}
```

### 参照元

- [WordPress Theme JSON – Per-block Settings and Styles](https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/#block-level-settings-and-styles)

---

## CSSカスタムプロパティの命名規則

### 自動生成されるCSS変数の規則

```css
/* パターン：--wp--preset--{category}--{slug} */

/* color.palette */
--wp--preset--color--primary
--wp--preset--color--base

/* typography.fontSizes */
--wp--preset--font-size--small
--wp--preset--font-size--large

/* typography.fontFamilies */
--wp--preset--font-family--body
--wp--preset--font-family--heading

/* spacing.spacingSizes */
--wp--preset--spacing--30
--wp--preset--spacing--50

/* shadow.presets */
--wp--preset--shadow--natural
```

### カスタムCSS変数（settings.custom）

```json
{
  "settings": {
    "custom": {
      "lineHeight": {
        "body": "1.75",
        "heading": "1.4"
      },
      "contentWidth": "720px"
    }
  }
}
```

```css
/* 生成されるCSS変数（キャメルケースはケバブケースに変換される） */
--wp--custom--line-height--body: 1.75;
--wp--custom--line-height--heading: 1.4;
--wp--custom--content-width: 720px;
```

### テーマのCSSで使う例

```css
/* 良い例：theme.jsonで定義したCSS変数を使う */
.custom-element {
  color: var(--wp--preset--color--primary);
  font-size: var(--wp--preset--font-size--medium);
  padding: var(--wp--preset--spacing--30);
}

/* 悪い例：theme.jsonと別にハードコードした値を使う */
.custom-element {
  color: #1e40af;
  font-size: 1rem;
  padding: 1rem;
}
```

### 参照元

- [WordPress Theme JSON – Custom Properties](https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/#custom-properties)

---

## theme.jsonの継承と優先順位

### 3層の設定レイヤー

```
1. WordPress本体（デフォルト値）
   ↓ 上書き
2. テーマの theme.json
   ↓ 上書き
3. ユーザーのカスタマイズ（サイトエディタのグローバルスタイル）
```

- 下のレイヤーが上のレイヤーを上書きする
- ユーザーのカスタマイズが最も優先される
- テーマの `theme.json` で `custom: false` にしても、ユーザーのグローバルスタイルは適用される

### 子テーマでの上書き

- 子テーマの `theme.json` は親テーマの `theme.json` にマージされる
- 同じプロパティは子テーマ側で上書きされる
- 配列（palette等）は全体が置き換わる（マージではなく上書き）

```json
// 親テーマ theme.json
{
  "settings": {
    "color": {
      "palette": [
        { "slug": "primary", "color": "#1e40af", "name": "プライマリ" },
        { "slug": "secondary", "color": "#9333ea", "name": "セカンダリ" }
      ]
    }
  }
}

// 子テーマ theme.json — palette全体が置き換わる
{
  "settings": {
    "color": {
      "palette": [
        { "slug": "primary", "color": "#dc2626", "name": "プライマリ" },
        { "slug": "secondary", "color": "#16a34a", "name": "セカンダリ" },
        { "slug": "tertiary", "color": "#2563eb", "name": "ターシャリ" }
      ]
    }
  }
}
```

### 参照元

- [WordPress Theme JSON – Inheritance](https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/)

---

## スタイルバリエーション

- 1つのテーマで複数のデザインバリエーションを提供する仕組み
- `styles/` ディレクトリに追加の `theme.json` ファイルを置く
- サイトエディタの「スタイル」→「スタイルを閲覧」から切り替えられる

### ディレクトリ構造

```
my-block-theme/
├── theme.json                  # デフォルトのスタイル
├── styles/
│   ├── dark-mode.json          # ダークモードバリエーション
│   ├── warm-palette.json       # 暖色パレットバリエーション
│   └── minimal.json            # ミニマルバリエーション
└── ...
```

### styles/dark-mode.json の例

```json
{
  "$schema": "https://schemas.wp.org/wp/6.5/theme.json",
  "version": 3,
  "title": "ダークモード",
  "settings": {
    "color": {
      "palette": [
        { "slug": "base", "color": "#1a1a2e", "name": "ベース" },
        { "slug": "contrast", "color": "#e2e8f0", "name": "コントラスト" },
        { "slug": "primary", "color": "#60a5fa", "name": "プライマリ" },
        { "slug": "secondary", "color": "#a78bfa", "name": "セカンダリ" },
        { "slug": "accent", "color": "#fbbf24", "name": "アクセント" }
      ]
    }
  },
  "styles": {
    "color": {
      "background": "var(--wp--preset--color--base)",
      "text": "var(--wp--preset--color--contrast)"
    }
  }
}
```

### 参照元

- [WordPress Theme Handbook – Style Variations](https://developer.wordpress.org/themes/global-settings-and-styles/style-variations/)

---

## シャドウプリセット

### theme.jsonでの定義

```json
{
  "settings": {
    "shadow": {
      "defaultPresets": false,
      "presets": [
        {
          "slug": "small",
          "name": "小",
          "shadow": "0 1px 2px 0 rgba(0, 0, 0, 0.05)"
        },
        {
          "slug": "medium",
          "name": "中",
          "shadow": "0 4px 6px -1px rgba(0, 0, 0, 0.1)"
        },
        {
          "slug": "large",
          "name": "大",
          "shadow": "0 10px 15px -3px rgba(0, 0, 0, 0.1)"
        }
      ]
    }
  }
}
```

### 参照元

- [WordPress Theme JSON – Shadow Settings](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/#shadow)

---

## theme.json設計のベストプラクティス

### デザイントークンの一貫性

```
良い設計：
  ✔ 色名は用途ベースで命名（primary, base, contrast）
  ✔ フォントサイズは相対値（rem）で定義
  ✔ スペーシングはスケール（10, 20, 30...）で統一
  ✔ stylesではCSS変数を参照する

悪い設計：
  ✘ 色名を具体的な色で命名（blue, red, dark-gray）
  ✘ フォントサイズをpxで固定
  ✘ スペーシングをバラバラな値で定義
  ✘ stylesにハードコードした値を書く
```

### エディタUIの制御

```json
{
  "settings": {
    "color": {
      "custom": false,
      "defaultPalette": false,
      "defaultGradients": false
    },
    "typography": {
      "customFontSize": false
    },
    "spacing": {
      "customSpacingSize": false
    }
  }
}
```

- `custom: false` でカスタム値入力を無効化すると、パレットからの選択のみになる
- デザインの一貫性を保ちたい場合に有効
- 運用チームがテーマのデザインシステムから逸脱するのを防ぐ

### $schemaの指定

```json
{
  "$schema": "https://schemas.wp.org/wp/6.5/theme.json",
  "version": 3
}
```

- `$schema` を指定するとエディタ（VS Code等）で補完・バリデーションが効く
- `version` は現在 3（WordPress 6.1以降）

### 参照元

- [WordPress Theme JSON Reference](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/)
- [WordPress Theme Handbook – Global Settings and Styles](https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/)

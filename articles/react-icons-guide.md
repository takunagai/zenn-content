---
title: "React プロジェクトでイチオシのアイコンフォントライブラリ react-icons"
emoji: "🐸"
type: "tech"
topics: ["font", "fontawesome","react", "nextjs", "shadcnui"]
published: false
---

- [React Icons](https://react-icons.github.io/react-icons/)
- [react-icons/react-icons - Github](https://github.com/react-icons/react-icons)

## TL;DR

- **多様性と選択肢**: 複数のアイコンライブラリから最適なものを選べる
- **LINEアイコン**: LINEアイコンが使える(Font Awesome を含む)
- **パフォーマンス**: ツリーシェイキングによる最適なバンドルサイズ
- **柔軟性**: 簡単なスタイリングとカスタマイズ

## react-icons を選ぶ理由

### 1. 複数アイコンライブラリを一元管理し、つまみ食いが可能

react-icons は、**多数の有名アイコンセットを1つのパッケージで利用できます**。Lucide、Font Awesome、Feather 他、様々なライブラリのアイコンを好みに応じて組み合わせることができます。(※ただし、統一感を失わないよう注意が必要です。)

```jsx
// Font Awesome 6のアイコンを使用
import { FaHouse } from "react-icons/fa6";

// Material Designのアイコンを使用
import { MdEmail } from "react-icons/md";

// Lucideのアイコンを使用（shadcn/ui デフォルトのアイコンセット）
import { LuSettings } from "react-icons/lu";

// 実際の使用例
function NavBar() {
  return (
    <nav>
      <a href="/"><FaHouse /></a>
      <a href="/contact"><MdEmail /></a>
      <a href="/settings"><LuSettings /></a>
    </nav>
  );
}
```

#### LINEアイコンが無駄なく使える

LINE アイコンは多くのシーンで必要不可欠ですが、Font Awesome 以外のアイコンフォントには含まれていません。LINE アイコンのためだけに Font Awesome をインストールするのは非効率的です。react-icons を使用すれば、多様なアイコンを含む単一のパッケージで、LINE アイコンを含むあらゆるニーズに対応できます。これにより、プロジェクトの管理が簡素化され、パフォーマンスも向上します。

```jsx
// LINEアイコンの使用例
import { FaLine } from "react-icons/fa6";

<FaLine className="h-5 w-5" />
```

### メモ： Font Awesome は v5 と v6 が含まれている

react-icons は、Font Awesomeの2バージョンが含まれています。リデザインされ、より統一感のあるスタイルの最新のアイコンが含まれている Font Awesome 6 (`fa6`) を使用することを推奨します。

```jsx
// Font Awesome 5（古いバージョン）
import { FaHome } from "react-icons/fa";

// Font Awesome 6（推奨）
import { FaHouse } from "react-icons/fa6";
```

### 2. ツリーシェイキングに対応（バンドルサイズの最適化）

react-icons は完全なツリーシェイキングに対応しており、バンドルに含まれるのは使用するアイコンだけです。多数のアイコンを含む容量の大きなライブラリを使っていても、実際に使用するアイコンだけがバンドルサイズに影響するため、パフォーマンスを損ねません。

```jsx
// この場合、FaHouse と FaUser のみがバンドルに含まれる
import { FaHouse, FaUser } from "react-icons/fa6";
```

### 3. スタイリングの柔軟性(SVGベース)

すべてのアイコンは SVG ベースで、通常の React コンポーネントとして扱えるため、CSS や inline styles でのカスタマイズが可能です。

```jsx
// サイズと色を変更する例
<FaHeart color="red" size="2em" />

// CSSクラスを適用する例
<FaHeart className="icon heart-icon" />

// Tailwind CSSと組み合わせる例
<FaHeart className="h-6 w-6 text-red-500 hover:text-red-700 transition-colors" />
```

### 4. アクセシビリティ対応

SVGベースのアイコンは、スクリーンリーダーなどの支援技術との互換性が高く、適切な `aria` 属性を追加することで、アクセシブルなアプリケーションを構築できます。

```jsx
<FaInfo aria-label="追加情報" role="img" />
```

## shadcn/ui との統合

UIコンポーネントライブラリ shadcn/ui はデフォルトで `lucide-react` アイコンライブラリを使用していますが、react-icons への移行は簡単です。

### 1. 設定ファイル components.json の編集

プロジェクトのルートディレクトリにある `components.json` を編集。

```json
{
  "iconLibrary": "react-icons"  // "lucide"から変更
}
```

### 2. アイコンの置き換え

react-icons では lucide アイコンも利用可能です。shadcn/ui のデフォルトスタイルを維持したい場合は、`react-icons/lu` からインポートします。

```jsx
// 変更前（lucide-react）
import { Moon, Sun, Check } from "lucide-react";

// 変更後。元と同じ lucide を react-icons で使用
import { LuMoon, LuSun, LuCheck } from "react-icons/lu";
// または FontAwesome 6 の同種のアイコンに変更
import { FaMoon, FaSun, FaCheck } from "react-icons/fa6";
```

## 注意点とベストプラクティス

1. **必要なアイコンだけをインポート**: 不要なアイコンをインポートすると、ツリーシェイキングの効果が低減します。
2. **一貫したスタイリング**: プロジェクト全体でアイコンのスタイルに一貫性を持たせましょう。
3. **アクセシビリティ**: 装飾的でないアイコンには適切な `aria` 属性を設定しましょう。

## まとめ

react-icons は Next.js ベースのアプリケーション を含む React プロジェクトにおいて、効率的でメンテナンスしやすいオススメのアイコンフォントファイブラリです。

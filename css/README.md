# CSS coding guide

## General

- idセレクターを使用しない

- keyflamesはケバブケースで記述する

```css
/* bad */
@keyflames {
  fadeIn {
    0% {
      opacity: 0;
    }

    100% {
      opacity: 1;
    }
  }
}

/* good */
@keyflames {
  fade-in {
    0% {
      opacity: 0;
    }

    100% {
      opacity: 1;
    }
  }
}
```

- 短縮プロパティに結合できるロングハンドプロパティを使用しない

```css
/* bad */
a {
  padding-top: 1px;
  padding-right: 2px;
  padding-bottom: 3px;
  padding-left: 4px;
}
/* good */
a {
  padding: 1px 2px 3px 4px;
}
```

- 省略形のプロパティで冗長な記述をしない

```css
/* bad */
a {
  margin: 10px 15px 10px 15px;
}
/* good */
a {
  margin: 10px 15px;
}
```

## BEM

Block: スタイルの論理単位の一意で意味のある名前

```css
ConciergeCard {}
```

Element: ブロックの子にのみ適用されるスタイル。ブロック名に2つのアンダースコアで連結する。

```css
ConciergeCard__title {}
```

Modifier: ブロックまたは要素の基本スタイルをモディファイアスタイルでオーバーライドまたは拡張する。ブロック名に2つのハイフンで連結する。

```css
ConciergeCard__title--large {}
ConciergeCard__title--small {}
```

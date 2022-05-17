# CSS coding guide

## General

- idセレクターを使用しない

使いたい場合はpseudo selector使う

```css
[id="id-selector"] {
  margin: 0;
}
```

- keyframesはケバブケースで記述する

```css
/* bad */
@keyframes {
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
@keyframes {
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

`background` `animation`  `transition`に関してはその限りではない。

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

- コンテンツのfont-sizeはremまたは相対長を使用する

```css
/* bad */
a {
  font-size: 10mm;
}
p {
  font-size: 10px;
}
/* good */
a {
  font-size: 10ex;
}
p {
  font-size: 1.2rem;
}
em {
  font-size: 2em;
}
```

- Type Selectorは非推奨

```css
/* bad */
a {
  color: red;
}
/* good */
.link {
  color: red;
}

```

- !importantの禁止

```css
/* bad */
.important-class {
  color: red!important;
}
/* good */
.important-class.important-class.important-class.important-class {
  color: red;
}

```

## BEM

- Block: スタイルの論理単位の一意で意味のある名前

```css
ConciergeCard {}
```

- Element: ブロックの子にのみ適用されるスタイル。ブロック名に2つのアンダースコアで連結する。

```css
ConciergeCard__title {}
```

- Modifier: ブロックまたは要素の基本スタイルをモディファイアスタイルでオーバーライドまたは拡張する。ブロック名に2つのハイフンで連結する。

```css
ConciergeCard__title--large {}
ConciergeCard__title--small {}
```

## SCSS

- Parent Selectorを使用しない

```scss
/* bad */
.ConciergeCard {
  margin: 10px 15px;
  &__title {
    color: red;
  }
}

/* good */
.ConciergeCard {
  margin: 10px 15px;
}

.ConciergeCard__title {
  color: red;
}
```

- @importの禁止

```scss
/* bad */
@import "./mixins";

/* good */
@use "./mixins" as mixin;
```

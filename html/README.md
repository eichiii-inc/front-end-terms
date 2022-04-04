# HTML マークアップガイド

## Style Rule

### general style

#### インデント
<!-- markdownlint-disable -->
<a name="html-indent">インデント</a>
<!-- markdownlint-enable -->

2スペース

```html
<!-- bad -->
<!-- markdownlint-disable -->
<section>
	<article>記事</article> <!-- タブインデント -->
	<article>記事</article> <!-- タブインデント -->
</section>
<!-- markdownlint-enable -->

<!-- good -->
<section>
  <article>記事</article> <!-- スペースインデント -->
  <article>記事</article> <!-- スペースインデント -->
</section>
```

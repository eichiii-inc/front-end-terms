# Typescriptコーディング規約

## インターフェース

- 名前にパスカルケースを使う

```TypeScript
// bad
interface foo {
  bar: string
}
// good
interface Foo {
  bar: string
}
```

- Enumを使わない

```TypeScript
// bad
enum FOO {
  BAR,
  HOGE,
  FUGA
}

// good
const FOO = {
  BAR: 0,
  HOGE: 0,
  FUGA: 0
} as const
type FOO = typeof FOO[keyof typeof FOO];
```

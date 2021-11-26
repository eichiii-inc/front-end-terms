# Javascript コーディング規約

## String

 プログラムで文字列を生成する場合は、文字列連結ではなくテンプレートリテラルを使用すること。
```JavaScript
// bad
  const sayHello = (name) => {
    return 'Hello' + ' ' + name
  }

 // good
  const sayHello = (name) => {
    return `Hello ${name}`
  }
```

## Equality

抽象的な比較演算子 == や != よりも、厳格な比較演算子 === や !== を使用すること。

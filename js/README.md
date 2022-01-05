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

## Object

メソッドの短縮構文を使用すること。

```JavaScript
// bad
const atom = {
  value: 1,
  addValue: function (value) {
    return atom.value + value
  }
}

// good
const atom = {
  value: 1,
  addValue (value) {
    return atom.value + value
  }
}
```

プロパティの短縮構文を使用すること。

```Javascript
const eichikun = 'eichikun'
// bad
const obj = {
  eichikun: eichikun,
}

//good
const obj = {
  eichikun,
}
```

## Array

配列をコピーする場合は、配列のスプレッド演算子を使用すること。

```Javascript
// bad
const items = [1,2,3,4,5]
const itemsCopy = []

for (let i = 0; i < items.length; i++) {
  itemsCopy[i] = items[i]
}

// good
const itemsCopy = [...items]
```

## Destructuring

複数プロパティから成るオブジェクトにアクセスする際に、分割代入を使用する。

```Javascript
// bad
const getFullName = (user) => {
  const firstName = user.firstName
  const lastName = user.lastName
  return `${firstName} ${lastName}`
}

// good
const getFullName = (obj) => {
  const { firstName, lastName } = obj
  return `${firstName} ${lastName}`
}

// best
const getFullName = ({ firstName, lastName }) => {
  return `${firstName} ${lastName}`
}
```

## Equality

抽象的な比較演算子 == や != よりも、厳格な比較演算子 === や !== を使用すること。

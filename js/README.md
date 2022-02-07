# Javascript コーディング規約

## String

- シングルクォートを使う

```JavaScript
// bad
const foo = "bar"
//good
const foo = 'bar'
```

- プログラムで文字列を生成する場合は、文字列連結ではなくテンプレートリテラルを使用する。

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

- メソッドの短縮構文を使用する。

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

- プロパティの短縮構文を使用する。

```Javascript
const eichikun = 'eichikun'
// bad
const obj = {
  eichikun: eichikun,
  // … some other … //
}

//good
const obj = {
  eichikun,
  // … some other … //
}
```

- オブジェクトのコピーはスプレッド演算子を使用する。

```JavaScript
// bad
const original = { a: 1, b: 2 }
const copy = Object.assign(original, { c: 3 })

// good
const original = { a: 1, b: 2 }
const copy = { ...original, c: 3 }
```

## Array

- 配列をコピーする場合は、配列のスプレッド演算子を使用する。

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

## Functions

- 関数が単一の引数を取り、{ }を使用しない場合は、( )を省略すること。

```JavaScript
// bad
[1, 2, 3].map((x) => x + 1)

// good
[1, 2, 3].map(x => x + 1)
```

## Destructuring

- 複数プロパティから成るオブジェクトにアクセスする際に、分割代入を使用する。

```Javascript
// bad
const getFullName = (user) => {
  const firstName = user.firstName
  const lastName = user.lastName
  return `${firstName} ${lastName}`
}

// good
const getFullName = (user) => {
  const { firstName, lastName } = user
  return `${firstName} ${lastName}`
}

// best
const getFullName = ({ firstName, lastName }) => {
  return `${firstName} ${lastName}`
}
```

## Modules

パスからは一箇所でインポートする。

```Javascript
// bad
import foo from 'foo';
// … some other imports … //
import { eichi1, eichi2 } from 'foo'

// good
import foo, { eichi1, eichi2 } from 'foo'
```

## Equality

- 抽象的な比較演算子 == や != よりも、厳格な比較演算子 === や !== を使用する。

```JavaScript
//bad
if (a == 'bar') {
  console.log('bar')
}

// good
if (a === 'bar') {
  console.log('bar')
}
```

- 三項演算子は入れ子にするべきではなく、単一行に記述すること。

```Javascript
// bad
const foo = maybe1 > maybe2
  ? "bar"
  : value1 > value2 ? "baz" : null

// 分離させる
const maybeNull = value1 > value2 ? 'baz' : null
// good
const foo = maybe1 > maybe2 ? 'bar' : maybeNull
```

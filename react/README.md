# React コーディング規約


## Props

- Propsはキャメルケースで記述する

```jsx
// bad
<Foo
  UserName="eichikun"
  phone_number={12345678} />

// good
<Foo
  userName="eichikun"
  phoneNumber={12345678} />
```

- 属性値が明示的に```true```である場合は省略する

```jsx
// bad
<Foo
  open={true} />

// good
<Foo
  open />

```

## Tags

- 子要素を持たない場合、常に自身でタグを閉じる

```jsx
// bad
<Foo className="stuff"></Foo>

// good
<Foo className="stuff" />
```

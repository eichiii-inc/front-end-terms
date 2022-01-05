# Vue コーディング規約

- 単一ファイルコンポーネント のファイル名は、すべてパスカルケース (PascalCase) にする

```
components/
|- MyComponent.vue
```

- v-for と同じ要素に v-if を使わない

```html
<!-- bad -->
<ul>
  <li
    v-for="user in users"
    v-if="shouldShowUsers"
    :key="user.id">
    {{ user.name }}
  </li>
</ul>

<!-- good -->
<ul v-if="shouldShowUsers">
  <li
    v-for="user in users"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
```

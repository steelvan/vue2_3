
# Условная отрисовка

{% raw %}
<script src="https://cdn.jsdelivr.net/npm/vue@2.6.12"></script>
<style>
.demo{
  border: 1px solid #eee;
  border-radius: 2px;
  padding: 25px 35px;
  margin-top: 1em;
  margin-bottom: 40px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  overflow-x: auto;    
}
</style>
{% endraw %}

## `v-if`

Директива `v-if` используется для отрисовки блока по условию. Блок будет отображаться только в том случае, если выражение директивы возвращает значение, приводимое к true.

```html
<h1 v-if="awesome">Vue восхитителен!</h1>
```

Также можно добавить блок «иначе», используя директиву `v-else`:

```html
<h1 v-if="awesome">Vue восхитителен!</h1>
<h1 v-else>О, нет 😢</h1>
```

### Условные группы с использованием `v-if` и `<template>`

Поскольку `v-if` — директива, она должна быть указана в одном конкретном теге. Но что если мы хотим управлять отображением сразу нескольких элементов? В таком случае можно применить `v-if` к псевдоэлементу `<template>`, который служит невидимой обёрткой и сам в результатах отрисовки не появляется.

```html
<template v-if="ага">
  <h1>Заголовок</h1>
  <p>Абзац 1</p>
  <p>Абзац 2</p>
</template>
```

### `v-else`

Для указания блока «иначе» для `v-if` можно использовать директиву `v-else`:

```html
<div v-if="Math.random() > 0.5">
  Сейчас меня видно
</div>
<div v-else>
  А теперь — нет
</div>
```

Элемент с директивой `v-else` должен следовать сразу за элементом с директивой `v-if` или `v-else-if`, иначе он не будет распознан.

### `v-else-if`

> Добавлено в версии 2.1.0+

Как следует из названия, `v-else-if` служит в качестве блока «else if» для директивы `v-if`. Можно объединять эти директивы в длинные цепочки:

```html
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Не A/B/C
</div>
```

Как и `v-else`, `v-else-if` должен следовать сразу за элементом с `v-if` или `v-else-if`.

### Управление повторным использованием элементов при помощи `key`

Vue старается управлять элементами DOM настолько эффективно, насколько это возможно, зачастую переиспользуя их вместо того чтобы создавать заново. Кроме улучшения производительности, у этого подхода есть и другие преимущества. Например, если вы позволяете пользователю переключаться между несколькими возможными типами входа.

```html
<template v-if="loginType === 'username'">
  <label>Имя пользователя</label>
  <input placeholder="Введите имя пользователя">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Введите адрес email">
</template>
```

Изменение `loginType` в коде выше не сотрёт то, что пользователь ввёл в поле. Оба шаблона используют одни и те же элементы, поэтому `<input>` не заменяется — только его `placeholder`.

{% raw %}
<div id="no-key-example" class="demo">
  <div>
    <template v-if="loginType === 'username'">
      <label>Имя пользователя</label>
      <input placeholder="Введите имя пользователя">
    </template>
    <template v-else>
      <label>Email</label>
      <input placeholder="Введите адрес email">
    </template>
  </div>
  <button v-on:click="toggleLoginType">Переключить тип входа</button>
</div>
<script>
new Vue({
  el: '#no-key-example',
  data: {
    loginType: 'username'
  },
  methods: {
    toggleLoginType: function () {
      return this.loginType = this.loginType === 'username' ? 'email' : 'username'
    }
  }
})
</script>
{% endraw %}

Иногда такое поведение нежелательно, поэтому Vue можно явно указать: «эти элементы должны быть полностью независимы, не надо их переиспользовать». Для этого всего лишь нужно указать уникальное значение ключа `key`:

```html
<template v-if="loginType === 'username'">
  <label>Имя пользователя</label>
  <input placeholder="Введите имя пользователя" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Введите адрес email" key="email-input">
</template>
```
Теперь эти поля ввода будут отрисовываться заново при каждом переключении:

{% raw %}
<div id="key-example" class="demo">
  <div>
    <template v-if="loginType === 'username'">
      <label>Имя пользователя</label>
      <input placeholder="Введите имя пользователя" key="username-input">
    </template>
    <template v-else>
      <label>Email</label>
      <input placeholder="Введите адрес email" key="email-input">
    </template>
  </div>
  <button v-on:click="toggleLoginType">Переключить тип входа</button>
</div>
<script>
new Vue({
  el: '#key-example',
  data: {
    loginType: 'username'
  },
  methods: {
    toggleLoginType: function () {
      return this.loginType = this.loginType === 'username' ? 'email' : 'username'
    }
  }
})
</script>
{% endraw %}

Обратите внимание, элементы `<label>` всё ещё эффективно переиспользуются, поскольку для них `key` не указаны.

## `v-show`

Ещё одну возможность условного отображения предоставляет директива `v-show`. Используется она так же:

```html
<h1 v-show="ага">Привет!</h1>
```

Разница в том, что элемент с `v-show` будет всегда оставаться в DOM, а изменяться будет лишь свойство `display` в его параметрах CSS.

`v-show` не работает на элементе `<template>` и не работает с `v-else`.

## `v-if` в сравнении с `v-show`

`v-if` производит «настоящую» условную отрисовку, удостоверяясь что подписчики событий и дочерние компоненты внутри блока должным образом уничтожаются и воссоздаются при изменении истинности управляющего условия.

`v-if` **ленивый**: если условие ложно на момент первоначальной отрисовки, он не произведёт никаких действий — условный блок не будет отображён, пока условие не станет истинным.

`v-show`, напротив, куда проще: элемент всегда присутствует в DOM, и только CSS-свойство переключается в зависимости от условия.

В целом, у `v-if` выше затраты на переключения, а у `v-show` больше затрат на первичную отрисовку. Так что если вы предполагаете, что переключения будут частыми, используйте `v-show`, если же редкими или маловероятными — `v-if`.

## `v-if` вместе с `v-for`

Совместное использование `v-if` и `v-for` **не рекомендуется**. Подробнее можно изучить в разделе [рекомендаций](/v2/style-guide/#Избегайте-использования-v-if-с-v-for-важно).

При совместном использовании `v-if` и `v-for`, директива `v-for` имеет более высокий приоритет. Подробнее на странице [отрисовки списков](list.md#v-for-и-v-if).

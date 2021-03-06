
# Безопасность


## Сообщение об уязвимости

При поступлении сообщения об уязвимости, её исправление сразу становится первоочередной задачей для нас. Сотрудник работающий на полную ставку, бросает всё, чтобы заняться ею. Чтобы сообщить об уязвимости, пожалуйста, отправьте сообщение на электронную почту [security@vuejs.org](mailto:security@vuejs.org).

Хотя новые уязвимости обнаруживаются редко, рекомендуем также всегда использовать последние версии Vue и его официальных библиотек для обеспечения максимальной безопасности вашего приложения.

## Правило №1: Никогда не использовать ненадёжные шаблоны

Главное правило для безопасности при использовании Vue — **никогда не использовать ненадёжное содержимое в качестве шаблона компонента**. Это аналогично разрешению исполнения JavaScript в приложении — и даже хуже, потому что может привести к нарушению работы сервера, если код выполнится при серверном рендеринге. Пример такого использования:

```js
new Vue({
  el: '#app',
  template: `<div>` + userProvidedString + `</div>` // НИКОГДА ТАК НЕ ДЕЛАЙТЕ
})
```

Шаблоны Vue компилируются в JavaScript, а выражения внутри шаблонов будут выполняться как часть процесса отрисовки. Несмотря на то, что выражения исполняются в определённом контексте отрисовки, из-за комплексности возможных сред выполнения, для такой инфраструктуры как Vue, нецелесообразно полностью защищать от потенциально вредоносного исполнения кода ввиду чрезмерных накладных расходов для производительности. Самый простой способ избежать этой категории проблем — убедиться, что содержимое шаблонов Vue всегда достоверно и полностью контролируется вами.

## Что делает Vue для вашей защиты

### HTML-содержимое

При использовании шаблонов или функций-создателей, содержимое экранируется автоматически. Это значит, что для шаблона:

{% raw %}
```html
<h1>{{ userProvidedString }}</h1>
```
{% endraw %}

если `userProvidedString` содержит:

```js
'<script>alert("hi")</script>'
```

то он будет экранирован в следующий HTML:

```html
&lt;script&gt;alert(&quot;hi&quot;)&lt;/script&gt;
```

таким образом предотвращая внедрение вредоносного скрипта. Экранирование осуществляется с помощью родного API обозревателя, такого как `textContent`, поэтому уязвимость возможна только в случае, если сам обозреватель уязвим.

### Привязка атрибутов

Аналогичным образом, динамические привязки к атрибутам также автоматически экранируются. Это значит, что для шаблона:

```html
<h1 v-bind:title="userProvidedString">
  hello
</h1>
```

если `userProvidedString` содержит:

```js
'" onclick="alert(\'hi\')'
```

то он будет экранирован в следующий HTML:

```html
&quot; onclick=&quot;alert('hi')
```

тем самым предотвращая преждевременное закрытие атрибута `title` для добавления нового, произвольного HTML. Экранирование выполняется с помощью родного API обозревателя, такого как `setAttribute`, поэтому уязвимость возможна только в случае, если сам обозреватель уязвим.

## Потенциальные опасности

Для любого веб-приложении возможность выполнения пользовательского содержимого без дезинфекции в формате HTML, CSS, или JavaScript является потенциально опасным, и этого следует избегать везде где только возможно. Однако бывают моменты, когда некоторый риск может быть приемлемым.

Например, сервисы такие как CodePen и JSFiddle позволяют выполнять пользовательский содержимое, но в таком контексте где это ожидается и изолируется внутри iframe. В тех случаях, когда важная функция по своей природе требует определённого уровня уязвимости, ваша команда должна взвесить необходимость этой функции с учётом наихудших сценариев, которые привнесёт её использование.

### Внедрение HTML

Как сказано ранее, Vue автоматически экранирует HTML-содержимое, предотвращая случайное внедрение HTML в приложение. Однако в тех случаях, когда вы уверены в безопасности HTML, можно отображать HTML-содержимое в сыром виде:

- Используя шаблон:
  ```html
  <div v-html="userProvidedHtml"></div>
  ```

- Используя функцию-создателя:
  ```js
  h('div', {
    domProps: {
      innerHTML: this.userProvidedHtml
    }
  })
  ```

- Используя функцию-создателя с JSX:
  ```jsx
  <div domPropsInnerHTML={this.userProvidedHtml}></div>
  ```

Запомните, что предоставленный пользователем HTML никогда не может считаться безопасным на 100%, если не находится в изолированной ирамке или в той части приложения, где только пользователь, написавший этот HTML, может получить к нему доступ. Кроме того, разрешать пользователям писать свои собственные шаблоны Vue может привнести аналогичные опасности.

### Внедрение URL

В таком URL-адресе:

```html
<a v-bind:href="userProvidedUrl">
  Нажми на меня
</a>
```

Существует потенциальная проблема безопасности, если URL не был "дезинфицирован" для предотвращения выполнения JavaScript через `javascript:`. Есть библиотеки, как [sanitize-url](https://www.npmjs.com/package/@braintree/sanitize-url), которые могут помочь с этим, но несмотря на это:

Если вы когда-нибудь занимались дезинфекцией URL на фронте, то у вас уже есть проблема с безопасностью. URL-адреса, предоставляемые пользователем, всегда должны дезинфицироваться на тыле, перед сохранением в базу данных. Тогда проблема будет решена для _каждого_ клиента, который подключается к вашему API, в том числе родные мобильные приложения. Также запомните, что использование дезинфицированных URL-адресов не гарантирует, что они ведут на безопасные ресурсы

### Внедрение стилей

Посмотрим на этот пример:

```html
<a
  v-bind:href="sanitizedUrl"
  v-bind:style="userProvidedStyles"
>
  Нажми на меня
</a>
```

предположим что `sanitizedUrl` был дезинфицирован и это действительно настоящий URL, а не JavaScript. Но используя `userProvidedStyles`, злоумышленники всё еще могут предоставить CSS для «click jack», т.е. стилизовать ссылку в виде прозрачного блока поверх кнопки «Входа в систему». В таком случае, если `https://user-controlled-website.com/` куда ведёт ссылка, создан таким, чтобы зрительно повторять на страницу авторизации вашего приложения, появляется возможность перехвата например имен входа и паролей пользователей.

Аналогично можете представить себе, как разрешение пользователям определять содержимое элемента `<style>` создаст ещё большую уязвимость, предоставив полный контроль над стилями страницы. Поэтому Vue не стоит отрисовывать теги стилей внутри шаблонов, например так:

{% raw %}
```html
<style>{{ userProvidedStyles }}</style>
```
{% endraw %}

Чтобы полностью обезопасить пользователей от техники click jacking, рекомендуем разрешать полный контроль над CSS только внутри изолированной ирамки. В качестве альтернативы, если всё-таки необходимо предоставить пользователю возможность настройки стилей рекомендуем использовать [объектный синтаксис](class-and-style.md#Использование-объектов-1) и позволять указывать только значения для конкретных свойств, которые безопасно изменять, например так:

```html
<a
  v-bind:href="sanitizedUrl"
  v-bind:style="{
    color: userProvidedColor,
    background: userProvidedBackground
  }"
>
  Нажми на меня
</a>
```

### Внедрение JavaScript

Мы настоятельно не рекомендуем отрисовку элементов `<script>` с помощью Vue, поскольку шаблоны и функции-отрисовщики никогда не должны иметь в себе побочных эффектов (side effects). Но это не единственный способ для включения строк, которые будут расцениваться как JavaScript во время выполнения.

Каждый HTML-элемент может иметь атрибуты, значения которых принимают строки JavaScript, например `onclick`, `onfocus`, и `onmouseenter`. Привязка JavaScript предоставленного пользователем к любому из этих атрибутов является потенциальной угрозой безопасности, поэтому этого следует избегать.

Запомните, что предоставленный пользователем JavaScript никогда не может считаться безопасным на 100%, если он не находится в изолированной ирамке или в части приложения, где только написавший его пользователь может когда-либо получить доступ к нему.

Иногда мы получаем сообщения об уязвимостях, что в шаблонах Vue возможно выполнение межсайтового скриптинга (XSS). В целом, мы не считаем такие случаи реальными уязвимостями, так как нет практического способа защиты разработчиков от двух сценариев, допускающих использование XSS:

1. Разработчик явно указывает Vue отрисовать предоставленный пользователем, не экранированное содержимое в шаблонах Vue. По своей природе это опасно и у Vue нет возможности отслеживать это.

2. Разработчик монтирует Vue на всю HTML-страницу, которая, как оказалось, содержит как содержимое отрисованное на сервере, так и предоставленное гостем. Фундаментально эта проблема аналогична \#1, но иногда разработчики могут сделать так, не осознавая этого. Подобное может привести к возможным уязвимостям, когда атакующий предоставляет HTML, который безопасен как обычный HTML, но опасен в качестве шаблона Vue. Лучше всего никогда не монтировать Vue на узлах, которые могут содержать содержимое, предоставленное как сервером, так и гостем.

## Лучшие практики

Главное правило заключается в том, что если разрешаете выполнение неэкранированного пользовательского содержимого (как например HTML, JavaScript, или даже CSS), то открываетесь для атак. Этот совет остаётся насущным, независимо от того используется ли Vue, другой каркас или никакого вообще.

Кроме вышеизложенных рекомендаций из раздела [потенциальных опасностей](#Потенциальные-опасности), рекомендуем также ознакомиться со следующими ресурсами:

- [HTML5 Security Cheat Sheet](https://html5sec.org/)
- [OWASP's Cross Site Scripting (XSS) Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

После изучения используйте полученные знания для проверки исходного кода зависимостей на наличие потенциально опасных мест, если они содержат сторонние компоненты или каким-либо иным образом влияют на то, что будет отрисовываться в DOM.

## Координация с тылом

Уязвимости безопасности HTTP, такие как подделка межсайтовых запросов (CSRF/XSRF) или внедрение межсайтовых скриптов (XSSI), в основном нацелены на тыл, поэтому Vue тут мало чем может помочь. Тем не менее, лучше скоординировать действия с командой разработчиков тыла, чтобы лучше узнать как следует взаимодействовать с API, например отправляя CSRF-жетоны при отправке форм.

## Отрисовка на стороне сервера (SSR)

При использовании SSR могут возникнуть дополнительные проблемы с безопасностью, поэтому во избежание уязвимостей следуйте рекомендациям изложенным в [документации по SSR](https://ssr.vuejs.org/ru/).

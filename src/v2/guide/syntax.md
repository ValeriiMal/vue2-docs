---
title: Синтаксис шаблону
type: guide
order: 4
---

Vue.js використовує синтаксис на основі HTML-шаблону, що дозволяє декларативно зв'язувати промальовування об'єктної моделі документу (DOM) за даними, закріпленими з екземпляром Vue. Усі шаблони Vue.js є валідним HTML, придатним для парсингу браузерам відповідно до специфікацій, а також парсерами HTML.

Під капотом, Vue компілює шаблони в так звані функції промальовування віртуальної об'єктної моделі документу (Virtual DOM). В комбінації із системою реактивності, Vue здатний розумно визначати мінімальне число компонентів для перемальовування та застосовувати мінімальну кількість маніпуляцій з DOM по мірі змін даних додатку.

Якщо ви знайомі з концепцією Virtual DOM і надаєте перевагу можливостям чистого JavaScript, ви також можете [відразу писати функції промальовування](render-function.html) замість шаблонів, з використанням JSX при бажанні.

## Інтерполяція

### Текст

Найбільш простою формою зв'язування даних — це текстова інтерполяція з використанням синтаксису "Mustache" (подвійні фігурні дужки)

``` html
<span>Повідомлення: {{ msg }}</span>
```

Дужки Mustache будуть замінені значенням властивості `msg` відповідного об'єкту даних. Воно буде автоматично змінюватися по мірі змін цієї властивості.

Ви також можете застосовувати одноразову інтерполяцію, яка не буде оновлюватися наступні рази, використовуючи [директиву v-once](../api/#v-once), але майте на увазі, що це вплине також на будь-які інші зв'язані дані того ж вузла:

``` html
<span v-once>Це ніколи не зміниться: {{ msg }}</span>
```

### Чистий HTML

Подвійні фігурні дужки розуміють дані як звичайний текст, а не HTML. Для того, щоб відображати реальний HTML
, вам потрібно використовувати директиву [`v-html`](../api/#v-html):

``` html
<p>Використання mustache: {{ rawHtml }}</p>
<p>Використання v-html: <span v-html="rawHtml"></span></p>
```

{% raw %}
<div id="example1" class="demo">
  <p>Використання mustache: {{ rawHtml }}</p>
  <p>Використання v-html: <span v-html="rawHtml"></span></p>
</div>
<script>
new Vue({
  el: '#example1',
  data: function () {
    return {
      rawHtml: '<span style="color: red">Це має бути червоним.</span>'
    };
  }
});
</script>
{% endraw %}

Вміст тегу `span` буде замінено значенням властивості `rawHtml`, інтерпретовану як звичайний HTML — зв'язування даних ігнорується. Зауважте, що ви не можете використовувати `v-html` для створення частин шаблонів, тому що Vue не є рядковим шаблонізатором. Замість цього краще використовувати компоненти як базову одиницю UI для повторного використання та композиції.

<p class="tip">Динамічне промальовування довільного HTML на вашому сайті може бути дуже небезпечним, оскільки може легко призвести до [XSS вразливостей](https://uk.wikipedia.org/wiki/%D0%9C%D1%96%D0%B6%D1%81%D0%B0%D0%B9%D1%82%D0%BE%D0%B2%D0%B8%D0%B9_%D1%81%D0%BA%D1%80%D0%B8%D0%BF%D1%82%D0%B8%D0%BD%D0%B3). Використовуйте інтерполяцію HTML лише з довіреним вмістом та **ніколи** не використовуйте на вмісті, що вводиться користувачем.</p>

### Атрибути

Mustache-дужки не можуть використовуватися всередині HTML-атрибутів. Замість цього використовуйте [директиву `v-bind`](../api/#v-bind):

``` html
<div v-bind:id="dynamicId"></div>
```

У випадку з булевими атрибутами, існування яких базується на значенні `true`, `v-bind` працює дещо по-іншому. Для прикладу:

``` html
<button v-bind:disabled="isButtonDisabled">Кнопка</button>
```

Якщо `isButtonDisabled` має значення `null`, `undefined`, або `false`, атрибут `disabled` навіть не буде промальовано в елементі `<button>`.

### Використання JavaScript виразів

До цього часу ми використовували зв'язування даних до звичайних властивостей в наших шаблонах. Але насправді Vue.js підтримує всю силу виразів JavaScript всередині зв'язаних даних:

``` html
{{ number + 1 }}

{{ ok ? 'ТАК' : 'НІ' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

Ці вирази будуть обчистені як JavaScript в контексті даних у Vue екземплярі. Єдиним обмеженням є те, що кожне зв'язування даних повинно містити **один-єдиний вираз**, тому ось цей код **НЕ** працюватиме:

``` html
<!-- це твердження, а не вираз: -->
{{ var a = 1 }}

<!-- управління потоком теж не працюватиме, використовуйте тернарні вирази -->
{{ if (ok) { return message } }}
```

<p class="tip">Вирази в шаблонах ізольовані і можуть мати доступ до [чітко визначеного списку глобальних змінних](https://github.com/vuejs/vue/blob/v2.6.10/src/core/instance/proxy.js#L9), таких як `Math` та `Date`. Не намагайтеся звернутися до глобальних змінних, визначених користувачем з шаблону.</p>

## Директиви

Директиви є спеціальними атрибутами, які мають префікс `v-`. Очікуваним значенням такого атрибуту є **єдиний JavaScript вираз** (виключенням є `v-for`, про який ми поговоримо пізніше). Завданням директиви є реактивне додавання побічних ефектів до DOM, коли значення або вираз атрибуту змінюється. Переглянемо той самий приклад, який ми бачити у вступі:

``` html
<p v-if="seen">Тепер ти мене бачиш</p>
```

Тут директива `v-if` видаляє або додає елемент `<p>` залежно від того, чи вираз `seen` є правдивим.

### Аргументи

Деякі директиви можуть приймати певний аргумент, відділений двокрапкою після імені директиви. Для прикладу, директива `v-bind` використовується для реактивного оновлення HTML атрибутів:

``` html
<a v-bind:href="url"> ... </a>
```

Тут `href` є її аргументом, що вказує директиві `v-bind` зв'язати атрибут елемента `href` із значенням в значенні виразу, в даному випадку `url`.

Іншим зразком є директива `v-on`, яка "слухає" події DOM:

``` html
<a v-on:click="doSomething"> ... </a>
```

Тут аргументом є назва події, яку потрібно прослуховувати. Ми поговоримо детальніше про обробку подій теж.

### Динамічні аргументи

> Нове в 2.6.0+

Починаючи з версії 2.6.0, також стало можливим використовув
Starting in version 2.6.0, it is also possible to use a JavaScript expression in a directive argument by wrapping it with square brackets:

``` html
<!--
Note that there are some constraints to the argument expression, as explained
in the "Dynamic Argument Expression Constraints" section below.
-->
<a v-bind:[attributeName]="url"> ... </a>
```

Here `attributeName` will be dynamically evaluated as a JavaScript expression, and its evaluated value will be used as the final value for the argument. For example, if your Vue instance has a data property, `attributeName`, whose value is `"href"`, then this binding will be equivalent to `v-bind:href`.

Similarly, you can use dynamic arguments to bind a handler to a dynamic event name:

``` html
<a v-on:[eventName]="doSomething"> ... </a>
```

In this example, when `eventName`'s value is `"focus"`, `v-on:[eventName]` will be equivalent to `v-on:focus`.

#### Dynamic Argument Value Constraints

Dynamic arguments are expected to evaluate to a string, with the exception of `null`. The special value `null` can be used to explicitly remove the binding. Any other non-string value will trigger a warning.

#### Dynamic Argument Expression Constraints

Dynamic argument expressions have some syntax constraints because certain characters, such as spaces and quotes, are invalid inside HTML attribute names. For example, the following is invalid:

``` html
<!-- This will trigger a compiler warning. -->
<a v-bind:['foo' + bar]="value"> ... </a>
```

The workaround is to either use expressions without spaces or quotes, or replace the complex expression with a computed property.

When using in-DOM templates (templates directly written in an HTML file), you should also avoid naming keys with uppercase characters, as browsers will coerce attribute names into lowercase:

``` html
<!--
This will be converted to v-bind:[someattr] in in-DOM templates.
Unless you have a "someattr" property in your instance, your code won't work.
-->
<a v-bind:[someAttr]="value"> ... </a>
```

### Modifiers

Modifiers are special postfixes denoted by a dot, which indicate that a directive should be bound in some special way. For example, the `.prevent` modifier tells the `v-on` directive to call `event.preventDefault()` on the triggered event:

``` html
<form v-on:submit.prevent="onSubmit"> ... </form>
```

You'll see other examples of modifiers later, [for `v-on`](events.html#Event-Modifiers) and [for `v-model`](forms.html#Modifiers), when we explore those features.

## Shorthands

The `v-` prefix serves as a visual cue for identifying Vue-specific attributes in your templates. This is useful when you are using Vue.js to apply dynamic behavior to some existing markup, but can feel verbose for some frequently used directives. At the same time, the need for the `v-` prefix becomes less important when you are building a [SPA](https://en.wikipedia.org/wiki/Single-page_application), where Vue manages every template. Therefore, Vue provides special shorthands for two of the most often used directives, `v-bind` and `v-on`:

### `v-bind` Shorthand

``` html
<!-- full syntax -->
<a v-bind:href="url"> ... </a>

<!-- shorthand -->
<a :href="url"> ... </a>

<!-- shorthand with dynamic argument (2.6.0+) -->
<a :[key]="url"> ... </a>
```

### `v-on` Shorthand

``` html
<!-- full syntax -->
<a v-on:click="doSomething"> ... </a>

<!-- shorthand -->
<a @click="doSomething"> ... </a>

<!-- shorthand with dynamic argument (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```

They may look a bit different from normal HTML, but `:` and `@` are valid characters for attribute names and all Vue-supported browsers can parse it correctly. In addition, they do not appear in the final rendered markup. The shorthand syntax is totally optional, but you will likely appreciate it when you learn more about its usage later.

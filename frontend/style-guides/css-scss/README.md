CSS, SCSS style guide
=====================

  - [Общие](#Общие)
  - [Внешний вид](#Внешний-вид)
  - [Наименование классов](#Наименование-классов)
  - [Пробелы](#Пробелы)
  - [Комментарии](#Комментарии)
  - [Селекторы](#Селекторы)
  - [Свойства](#Свойства)
  - [Значения](#Значения)
  - [Особые возможности SCSS](#Особые-возможности-scss)
  - [Иконки](#Иконки)


## Общие

[SCSS Reference](http://sass-lang.com/documentation/file.SASS_REFERENCE.html) (все что можно/нужно знать о sass)

- Разделение логических блоков пустой строкой. При необходимости в начало блока добавляется комментарий.
- Файлы — отдельный логический блок стилей. Все доделки для IE хранятся в отдельных файлах
  и подключаются через условные комментарии.
- Везде, где используются кавычки — только одинарные (`content: 'something'`)
- Не префиксируем сами.
- Максимально исключаем копипаст путем использования миксинов/функций/циклов/плейсхолдеров.
- `@import` используем только для подключения файлов миксинов и переменных. Для сборок используем спрокеты


## Внешний вид
*Селекторы*
- один селектор
```
.selector { ... }
```

- несколько селекторов
```
.selector1,
.selector2,
.selector3 { ... }
```

*Блоки объявлений*

Все свойства в одну строчку, независимо от её длины. После каждого свойства ставим `;`.
```
.selector { instruction1; ...; instructionN; }
```


## Наименование классов
- понятные названия,
- на корректном английском,
- с названием функциональности, а не внешнего вида,
- для логического разделения используется только `-` (дефис),

*родительские классы*
```
.main-menu { ... }
.parameter-box { ... }
```

*дочерние классы (с оговоркой)*
```
.mm-item { ... }
.pb-one { ... }
```

В проектах в наследство передались служебные и часто используемые классы. Их нельзя использовать в гемах
(за редким исключением), в проектах новые подобные тоже лучше не создавать, но удалять их пока тоже не рекомендуется,
так как они могут использоваться в текстовых зонах, цмс-страницах.

*сквозные служебные классы*
```
.inline-icon-16x16- { ... }
.icon-16x16-word- { ... }
.dashed-link- { ... }
```

*часто используемые классы (отступы и другие модификаторы)*
```
.w100p { width: 100%; !important; }
.w150 { width: 150px; !important; }
.p10 { padding: 10px !important; }
.ml-15 { margin-left: -15px !important; }
```

Для служебных классов необходимо использовать первые буквы слов свойства.
Также необходимо отражать в названии единицы измерения, отличные от `px`.


## Пробелы

Вокруг восклицательного знака в свойствах

:white_check_mark: Хорошо:
```
background: #fff !important;
```

:no_entry_sign: Плохо:
```
background: #fff!important;
background: #fff ! important;
```

Между свойствами и фигурными скобками, а также между селектором и фигурными скобками — обязательно ставим пробелы

:white_check_mark: Хорошо:**
```
.selector { margin: 0; }
```

:no_entry_sign: Плохо:
```
.selector {margin: 0;}
.selector{ margin:0; }
```


## Комментарии

В файлах scss мы используем комментарии обозначенные через двойной слеш. Такие комментарии не появляются
в файле css после компиляции. И да, я понимаю, что минификаторы уберут любые, но мы учимся писать сразу правильно.

:white_check_mark: Хорошо:
```
// No one, except developers will ever see me
```

:no_entry_sign: Плохо:
```
/* This code needs refactoring... call us - x-xxx-xxx-xx-xx */
```


## Селекторы

### Не используем идентификаторы для стилей.

Предпочительнее всего использовать только селектор по классу.
В нестандартных случаях допускается использование селекторов по атрибуту, маске, тегу.

:white_check_mark: Хорошо:
```
.selector { ... }
```

:ballot_box_with_check: Допустимо:
```
[type='text'] { ... }
[href] { ... }
[class^='-tree-'] { ... }
ul { ... }
```

:no_entry_sign: Плохо:
```
#header { ... }
[type=password] { ... }
```


### Каждый селектор (группа селекторов) с новой строки

:white_check_mark: Хорошо:
```
.selector-1,
.selector-2 { ... }
.selector-1 a,
.selector-2 a { ... }
```

:no_entry_sign: Плохо:
```
.selector-1, .selector-2 { ... }
.selector-1 a, .selector-2 a { ... }
```


### Максимальная глубина

Максимальная глубина не должна превышать 3 селектора. Псевдоэлементы и состояния считаются находящимися
на одном уровне со своим селектором.

:white_check_mark: Хорошо:
```
.selector-1 {
  .selector-2 {
    .selector-3 {
      color: #fff;
      &:after { content: ''; }
      &:hover { ... }
    }
  }
}

.selector-1 .selector-2 .selector-3 { ... }
```

:no_entry_sign: Плохо:
```
.selector-1 {
  .selector-2 {
    .selector-3 {
      color: #fff;
      .selector-4 { color: #fff; }
    }
  }
}

.selector-1 .selector-2 .selector-3 .selector-4 { ... }
```


### Перенос на новую строку

Если у селектора только свои свойства — пишем в 1 строку. Если внутри есть еще селекторы — переносим.

:white_check_mark: Хорошо:
```
.selector-1 { color: #fff; border: 1px solid #c9c9c9; }
.selector-2 {
  color: #fff; background: #000;
  a { color: #f00; }
}
```

:no_entry_sign: Плохо:
```
.selector-1 {
  color: #fff; border: 1px solid #c9c9c9;
}
```


## Свойства

### Приоритет

Сначала описываем свойства элемента. Затем свойства его состояний и псевдоэлементов. Затем следующие селекторы.

:white_check_mark: Хорошо:
```
.selector {
  color: #fff;
  &:after { ... }
  &:hover { ... }
  .selector-2 { ... }
}
```

:no_entry_sign: Плохо:
```
.selector {
  &:hover { ... }
  color: #fff;
  .selector-2 { ... }
  &:after { ... }
}
```


## Значения

### Алиасы цветов

Алиасы цветов мы не используем. Только код. Исключения — методы sass с заранее определенными переменными.

:white_check_mark: Хорошо:
```
color: #0f0;
color: color-calc(black, 50);
```

:no_entry_sign: Плохо:
```
color: green;
```


### Сокращаем код цвета, если имеется возможность

:white_check_mark: Хорошо:
```
color: #fff;
```

:no_entry_sign: Плохо:
```
color: #ffffff;
```


### Если значения идут через запятую, обязательно разделяем ещё и пробелом

:white_check_mark: Хорошо:
```
color: rgba(255, 255, 255, .3);
transform: translate(50%, 50%);
```

:no_entry_sign: Плохо:
```
color: rgba(255,255,255,.3);
transform: translate(50%,50%);
```


### Все значения строчными буквами

:white_check_mark: Хорошо:
```
color: #c9c9c9;
margin: 13px;
```

:no_entry_sign: Плохо:
```
color: #C9C9C9;
margin: 13PX;
```


### Не используем единицы измерения там, где не нужно

Правило большого пальца: если не знаете где нужно приписать единицу измерения, а где нет — убираете единицу измерения.
Если при этом ничего не поехало, значит тут можно обойтись без единиц измерения.

Некоторые свойства имеют единицы измерения по умолчанию (у `line-height` — `em`).
`0` всегда пишется без единицы измерения.

:white_check_mark: Хорошо:
```
margin: 0;
line-height: 2;
```

:no_entry_sign: Плохо:
```
margin: 0px;
line-height: 2em;
```


### Не пишем 0 там, где он не нужен

:white_check_mark: Хорошо:
```
opacity: .3;
line-height: 1;
```

:no_entry_sign: Плохо:
```
line-height: 1.0;
opacity: 0.3;
```


### Используем сокращения в тех свойствах, где можно

:white_check_mark: Хорошо:
```
margin: 10px;
padding: 10px 5px;
padding: 5px 10px 3px;
```

:no_entry_sign: Плохо:
```
margin: 10px 10px 10px 10px;
padding: 10px 5px 10px 5px;
padding: 5px 10px 3px 10px;
```


## Особые возможности SCSS

### Условия

Условия оформляем таким образом:

:white_check_mark: Хорошо:
```
@if $i == 1 {
  // do smth
} @else if $i == 2 {
  // do smth else
} @else {
  // O_o
}
```

:no_entry_sign: Плохо:
```
@if $i == 1 {
  // do smth
}
@else {
  // O_o
}
```


### Названия функций/миксинов/переменных состоящие из нескольких слов

Все вышеперечисленное называем на грамотном английском, четко обозначая что именно делает та(тот) или иная(-ой)
функция/переменная/миксин. Все слова строчными буквами через тире.

:white_check_mark: Хорошо:
```
$header-color: #000;
@mixin order-button($color) {
  // ...
}
```

:no_entry_sign: Плохо:
```
$blockBackgroundColor: #fff;
@mixin my_mixin() {
  // bad
}
```


### Экстенд

Никогда не экстендим классы. Только плейсхолдеры. Ну и если не понимаете подводные камни
экстенда — лучше вообще обойтись без него.

:white_check_mark: Хорошо:
```
.selector {
  @extend %placeholder
}
```

:no_entry_sign: Плохо:
```
.selector {
  @extend .class
}
```


## Иконки

Для случаев, когда иконка является дополнением к основному элементу и не несет отдельной нагрузки,
необходимо пользоваться псевдоэлементами и не создавать отдельных узлов в DOM-е.

Почему это хорошо:

- события на элементе всегда отрабатывают и на его псевдо
- редко когда обработчик какого-либо события на иконке должен делать что-то отличное обработчика события
на основном элементе, к которому привязана иконка
- иконка занимает отдельное место в доме, а так как чаще всего это ещё и часть handlebars-шаблона,
то в скомпилированном варианте для нее будут прописаны все соответствующие манипуляции с домом
для вставки нового элемента,
- если иконка — это отдельный элемент, обычно ей присваивается отдельный класс,
который мешается во вьюхе, мешается в стилях
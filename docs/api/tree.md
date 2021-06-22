<!-- front-matter
id: tree
title: tree()
hide_title: true
sidebar_label: tree()
-->

# tree()

Получает дерево зависимостей текущей задачи - в редких случаях, когда это необходимо.

Как правило, `tree()` не будет использоваться потребителями gulp, но он доступен, поэтому CLI может отображать граф зависимостей задач, определенных в gulpfile.

## Применение

Пример gulpfile:

```js

const { series, parallel } = require('gulp');

function one(cb) {
  // тело опущено
  cb();
}

function two(cb) {
  // тело опущено
  cb();
}

function three(cb) {
  // тело опущено
  cb();
}

const four = series(one, two);

const five = series(four,
  parallel(three, function(cb) {
    // тело опущено
    cb();
  })
);

module.exports = { one, two, three, four, five };
```

Вывод для `tree()`:

```js
{
  label: 'Tasks',
  nodes: [ 'one', 'two', 'three', 'four', 'five' ]
}
```

Вывод для `tree({ deep: true })`:

```js
{
  label: "Tasks",
  nodes: [
    {
      label: "one",
      type: "task",
      nodes: []
    },
    {
      label: "two",
      type: "task",
      nodes: []
    },
    {
      label: "three",
      type: "task",
      nodes: []
    },
    {
      label: "four",
      type: "task",
      nodes: [
        {
          label: "<series>",
          type: "function",
          branch: true,
          nodes: [
            {
              label: "one",
              type: "function",
              nodes: []
            },
            {
              label: "two",
              type: "function",
              nodes: []
            }
          ]
        }
      ]
    },
    {
      label: "five",
      type: "task",
      nodes: [
        {
          label: "<series>",
          type: "function",
          branch: true,
          nodes: [
            {
              label: "<series>",
              type: "function",
              branch: true,
              nodes: [
                {
                  label: "one",
                  type: "function",
                  nodes: []
                },
                {
                  label: "two",
                  type: "function",
                  nodes: []
                }
              ]
            },
            {
              label: "<parallel>",
              type: "function",
              branch: true,
              nodes: [
                {
                  label: "three",
                  type: "function",
                  nodes: []
                },
                {
                  label: "<anonymous>",
                  type: "function",
                  nodes: []
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

## Подпись

```js
tree([options])
```

### Параметры

| параметр | тип | примечание |
|:--------------:|------:|--------|
| options | object | Подробнее в [Опциях][options-section] ниже. |

### Возвращается

Объект, детализирующий дерево зарегистрированных задач - содержащий вложенные объекты со свойствами `'label'` и `'nodes'` (совместимый с [archy][archy-external] compatible).

Каждый объект может иметь свойство `type` , которое может использоваться для определения того, является ли node `task` или `function`.

Каждый объект может иметь свойство `branch`, которое, когда `true`, , указывает, что node была создана с помощью `series()` или `parallel()`.

### Опции

| наименование | тип | по умолчанию | примечание |
|:-------:|:-------:|------------|--------|
| deep | boolean | false | Если `true`, будет возвращено все дерево. Если установлено значение `false`, будут возвращены только задачи верхнего уровня. |

[options-section]: #options
[archy-external]: https://www.npmjs.com/package/archy

<!-- front-matter
id: task
title: task()
hide_title: true
sidebar_label: task()
-->

# task()

**Напоминание**: Этот API больше не является рекомендуемым шаблоном - [экспорт ваших задач][creating-tasks-docs].

Определяет задачу в системе задач. Затем к задаче можно будет получить доступ из командной строки и API-интерфейсов `series()`, `parallel()` и `lastRun()`.

## Применение

Зарегистрируйте именованную функцию как задачу:

```js
const { task } = require('gulp');

function build(cb) {
  // тело опущено
  cb();
}

task(build);
```

Зарегистрируйте анонимную функцию как задачу:

```js
const { task } = require('gulp');

task('build', function(cb) {
  // тело опущено
  cb();
});
```

Получить ранее зарегистрированную задачу:

```js
const { task } = require('gulp');

task('build', function(cb) {
  // тело опущено
  cb();
});

const build = task('build');
```

## Подпись

```js
task([taskName], taskFunction)
```

### Параметры

Если `taskName` не указано, на задачу будет ссылаться свойство `name` именованной функции или определяемое пользователем свойство `displayName`. Параметр `taskName` должен использоваться для анонимных функций, у которых отсутствует свойство `displayName`.

Поскольку любую зарегистрированную задачу можно запустить из командной строки, избегайте использования пробелов в именах задач.

| параметр | тип | примечание |
|:--------------:|:------:|-------|
| taskName | string | Псевдоним для функции задачи в системе задач. Не требуется при использовании именованных функций для `taskFunction`. |
| taskFunction<br />**(required)** | function | [Функция задачи][task-concepts] или составная задача - генерируется с помощью `series()` и `parallel()`. В идеале именованная функция. [Метаданные задачи][task-metadata-section] могут быть прикреплены для предоставления дополнительной информации в командной строке. |

### Возвращается

При регистрации задачи ничего не возвращается.

При получении задачи будет возвращена обернутая задача (не исходная функция), зарегистрированная как `taskName`. Обернутая задача имеет метод `unwrap()`, который вернет исходную функцию.

### Ошибки

При регистрации задачи, в которой `taskName` отсутствует, а `taskFunction` является анонимным, выдает ошибку с сообщением: "Task name must be specified".

## Метаданные задачи

| свойство | тип | примечание |
|:--------------:|:------:|-------|
| name | string | Особое свойство именованных функций. Используется для регистрации задачи.<br />**Примечание:** [`name`][function-name-external] не доступен для записи; его нельзя установить или изменить. |
| displayName | string | При присоединении к `taskFunction` создает псевдоним для задачи. Если используются символы, недопустимые в именах функций, используйте это свойство. |
| description | string | При присоединении к `taskFunction` предоставляет описание, которое будет напечатано командной строкой при выводе списка задач. |
| flags | object | При присоединении к `taskFunction` функция предоставляет флаги, которые печатаются командной строкой при выводе списка задач. Ключи объекта представляют собой флаги, а значения - их описания. |

```js
const { task } = require('gulp');

const clean = function(cb) {
  // тело опущено
  cb();
};
clean.displayName = 'clean:all';

task(clean);

function build(cb) {
  // тело опущено
  cb();
}
build.description = 'Build the project';
build.flags = { '-e': 'An example flag' };

task(build);
```

[task-metadata-section]: #task-metadata
[task-concepts]: ../api/concepts.md#tasks
[creating-tasks-docs]: ../getting-started/3-creating-tasks.md
[function-name-external]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/name

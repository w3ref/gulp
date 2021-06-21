<!-- front-matter
id: creating-tasks
title: Создание задач
hide_title: true
sidebar_label: Создание задач
-->

# Создание задач

Каждая задача gulp - это асинхронная функция JavaScript - функция, которая принимает обратный вызов сначала с ошибкой или возвращает поток, промис, эмиттер событий, дочерний процесс или наблюдаемый ([подробнее об этом позже][async-completion-docs]). Из-за некоторых ограничений платформы синхронные задачи не поддерживаются, хотя есть довольно изящная [альтернатива][using-async-await-docs].

## Экспортирование

Задачи могут считаться **публичными** или **приватными**.

* **Публичные задачи** экспортируются из вашего gulpfile, что позволяет запускать их с помощью команды `gulp`.
* **Приватные задачи** предназначены для внутреннего использования, обычно как часть композиции `series()` или `parallel()`.

Частная задача выглядит и действует как любая другая задача, но конечный пользователь никогда не сможет выполнить ее самостоятельно. Чтобы зарегистрировать задачу публично, экспортируйте ее из своего файла gulpfile.

```js
const { series } = require('gulp');

// Функция `clean` не экспортируется, поэтому ее можно рассматривать как частную задачу.
// Его все еще можно использовать в составе `series()`.
function clean(cb) {
  // тело опущено
  cb();
}

// Функция `build` экспортируется, поэтому она общедоступна и может быть запущена с помощью команды `gulp`.
// Его также можно использовать в составе `series()`.
function build(cb) {
  // тело опущено
  cb();
}

exports.build = build;
exports.default = series(clean, build);
```

![ALT TEXT MISSING][img-gulp-tasks-command]

<small>В прошлом `task()` использовался для регистрации ваших функций как задач. Хотя этот API все еще доступен, экспорт должен быть основным механизмом регистрации, за исключением крайних случаев, когда экспорт не работает.</small>

## Составление задач

Gulp предоставляет два мощных метода композиции, `series()` и `parallel()`, позволяя объединять отдельные задачи в более крупные операции. Оба метода принимают любое количество функций задач или составных операций.  `series()` и `parallel()` могут быть вложены внутри себя или друг в друга на любую глубину.

Чтобы ваши задачи выполнялись по порядку, используйте метод `series()`.

```js
const { series } = require('gulp');

function transpile(cb) {
  // тело опущено
  cb();
}

function bundle(cb) {
  // тело опущено
  cb();
}

exports.build = series(transpile, bundle);
```

Чтобы задачи выполнялись с максимальным параллелизмом, объедините их с методом `parallel()`.

```js
const { parallel } = require('gulp');

function javascript(cb) {
  // тело опущено
  cb();
}

function css(cb) {
  // тело опущено
  cb();
}

exports.build = parallel(javascript, css);
```

Задачи составляются сразу после вызова `series()` или `parallel()`. Это позволяет варьировать композицию вместо условного поведения внутри отдельных задач.

```js
const { series } = require('gulp');

function minify(cb) {
  // тело опущено
  cb();
}


function transpile(cb) {
  // тело опущено
  cb();
}

function livereload(cb) {
  // тело опущено
  cb();
}

if (process.env.NODE_ENV === 'production') {
  exports.build = series(transpile, minify);
} else {
  exports.build = series(transpile, livereload);
}
```

`series()` и `parallel()` могут быть вложены на любую произвольную глубину.

```js
const { series, parallel } = require('gulp');

function clean(cb) {
  // тело опущено
  cb();
}

function cssTranspile(cb) {
  // тело опущено
  cb();
}

function cssMinify(cb) {
  // тело опущено
  cb();
}

function jsTranspile(cb) {
  // тело опущено
  cb();
}

function jsBundle(cb) {
  // тело опущено
  cb();
}

function jsMinify(cb) {
  // тело опущено
  cb();
}

function publish(cb) {
  // тело опущено
  cb();
}

exports.build = series(
  clean,
  parallel(
    cssTranspile,
    series(jsTranspile, jsBundle)
  ),
  parallel(cssMinify, jsMinify),
  publish
);
```

Когда выполняется составная операция, каждая задача будет выполняться каждый раз, когда на нее ссылаются. Например, задача `clean`, на которую ссылаются перед двумя разными задачами, будет запущена дважды и приведет к нежелательным результатам. Вместо этого выполните рефакторинг `clean` задачи, чтобы она была указана в окончательной композиции.

Если у вас есть такой код:

```js
// Это НЕПРАВИЛЬНО
const { series, parallel } = require('gulp');

const clean = function(cb) {
  // тело опущено
  cb();
};

const css = series(clean, function(cb) {
  // тело опущено
  cb();
});

const javascript = series(clean, function(cb) {
  // тело опущено
  cb();
});

exports.build = parallel(css, javascript);
```

Переходите на это:

```js
const { series, parallel } = require('gulp');

function clean(cb) {
  // тело опущено
  cb();
}

function css(cb) {
  // тело опущено
  cb();
}

function javascript(cb) {
  // тело опущено
  cb();
}

exports.build = series(clean, parallel(css, javascript));
```

[async-completion-docs]: ../getting-started/4-async-completion.md
[using-async-await-docs]: ../getting-started/4-async-completion.md#using-async-await
[img-gulp-tasks-command]: https://gulpjs.com/img/docs-gulp-tasks-command.png
[async-once]: https://github.com/gulpjs/async-once

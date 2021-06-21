<!-- front-matter
id: using-plugins
title: Использование плагинов
hide_title: true
sidebar_label: Использование плагинов
-->

# Использование плагинов

Плагины Gulp - это [Node Transform Streams][through2-docs], которые инкапсулируют общее поведение для преобразования файлов в конвейере - часто помещаются между `src()` и `dest()` с помощью метода `.pipe()`. Они могут изменять имя файла, метаданные или содержимое каждого файла, проходящего через поток.

Плагины от npm, использующие ключевые слова "gulpplugin" и "gulpfriendly", можно просматривать и искать на [странице поиска плагинов][gulp-plugin-site].

Каждый плагин должен выполнять лишь небольшую часть работы, поэтому вы можете соединять их как блоки сборки. Возможно, вам придется объединить их несколько, чтобы получить желаемый результат.

```js
const { src, dest } = require('gulp');
const uglify = require('gulp-uglify');
const rename = require('gulp-rename');

exports.default = function() {
  return src('src/*.js')
    // Плагин gulp-uglify не обновляет имя файла
    .pipe(uglify())
    // Поэтому используйте gulp-rename, чтобы изменить расширение
    .pipe(rename({ extname: '.min.js' }))
    .pipe(dest('output/'));
}
```

## Вам нужен плагин?

Не все в gulp должны использовать плагины. Это быстрый способ начать работу, но многие операции можно улучшить, используя вместо них модуль или библиотеку.

```js
const { rollup } = require('rollup');

// API обещаний Rollup отлично работает в задаче `async`
exports.default = async function() {
  const bundle = await rollup({
    input: 'src/index.js'
  });

  return bundle.write({
    file: 'output/bundle.js',
    format: 'iife'
  });
}
```

Плагины всегда должны преобразовывать файлы. Используйте (не подключаемый) модуль Node или библиотеку для любых других операций.

```js
const del = require('delete');

exports.default = function(cb) {
  // Используйте модуль `delete` напрямую, вместо использования gulp-rimraf
  del(['output/*.js'], cb);
}
```

## Условия плагинов

Поскольку операции плагина не должны учитывать типы файлов, вам может понадобиться плагин, например [gulp-if][gulp-if-package], для преобразования подмножеств файлов.

```js
const { src, dest } = require('gulp');
const gulpif = require('gulp-if');
const uglify = require('gulp-uglify');

function isJavaScript(file) {
  // Проверьте, является ли расширение файла '.js'
  return file.extname === '.js';
}

exports.default = function() {
  // Включите файлы JavaScript и CSS в один конвейер
  return src(['src/*.js', 'src/*.css'])
    // Применяйте плагин gulp-uglify только к файлам JavaScript
    .pipe(gulpif(isJavaScript, uglify()))
    .pipe(dest('output/'));
}
```

## Встроенные плагины

Встроенные плагины - это одноразовые потоки преобразования, которые вы определяете внутри своего gulpfile, записывая желаемое поведение.

Есть две ситуации, когда создание встроенного плагина полезно:

* Вместо создания и поддержки собственного плагина.
* Вместо того, чтобы разрабатывать существующий плагин, чтобы добавить желаемую функцию.

```js
const { src, dest } = require('gulp');
const uglify = require('uglify-js');
const through2 = require('through2');

exports.default = function() {
  return src('src/*.js')
    // Вместо использования gulp-uglify вы можете создать встроенный плагин
    .pipe(through2.obj(function(file, _, cb) {
      if (file.isBuffer()) {
        const code = uglify.minify(file.contents.toString())
        file.contents = Buffer.from(code.code)
      }
      cb(null, file);
    }))
    .pipe(dest('output/'));
}
```

[gulp-plugin-site]: https://gulpjs.com/plugins/
[through2-docs]: https://github.com/rvagg/through2
[gulp-if-package]: https://www.npmjs.com/package/gulp-if

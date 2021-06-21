<!-- front-matter
id: working-with-files
title: Работа с файлами
hide_title: true
sidebar_label: Работа с файлами
-->

# Работа с файлами

Методы `src()` и `dest()` предоставляются gulp для взаимодействия с файлами на вашем компьютере.

`src()` получает [glob][explaining-globs-docs] для чтения из файловой системы и создает [Node stream][node-streams-docs]. Он находит все совпадающие файлы и считывает их в память для прохождения через поток.

Поток, созданный `src()`, должен быть возвращен из задачи, чтобы сигнализировать об асинхронном завершении, как упомянуто в [Создание задач][creating-tasks-docs].

```js
const { src, dest } = require('gulp');

exports.default = function() {
  return src('src/*.js')
    .pipe(dest('output/'));
}
```

Основным API потока является метод `.pipe()` для объединения потоков Transform или Writable.

```js
const { src, dest } = require('gulp');
const babel = require('gulp-babel');

exports.default = function() {
  return src('src/*.js')
    .pipe(babel())
    .pipe(dest('output/'));
}
```

`dest()` получает строку каталога вывода, а также создает [Node stream][node-streams-docs], который обычно используется как поток-ограничитель. Когда он получает файл, прошедший через конвейер, он записывает содержимое и другие детали в файловую систему в заданном каталоге. Метод `symlink()` также доступен и работает как `dest()`, но создает ссылки вместо файлов (подробности смотрите в [`symlink()`][symlink-api-docs]).

Чаще всего плагины помещаются между `src()` и `dest()` с использованием метода `.pipe()` и преобразуют файлы в потоке.

## Добавление файлов в поток

`src()` также можно разместить в середине конвейера для добавления файлов в поток на основе заданных globs. Дополнительные файлы будут доступны только для преобразований позже в потоке. Если [globs overlap][overlapping-globs-docs], файлы будут добавлены снова.

Это может быть полезно для транспиляции некоторых файлов перед добавлением простых файлов JavaScript в конвейер и отменой всего.

```js
const { src, dest } = require('gulp');
const babel = require('gulp-babel');
const uglify = require('gulp-uglify');

exports.default = function() {
  return src('src/*.js')
    .pipe(babel())
    .pipe(src('vendor/*.js'))
    .pipe(uglify())
    .pipe(dest('output/'));
}
```

## Выход в фазах

`dest()` может использоваться в середине конвейера для записи промежуточных состояний в файловую систему. Когда файл получен, текущее состояние записывается в файловую систему, путь обновляется для представления нового местоположения выходного файла, а затем этот файл продолжается по конвейеру.

Эта функция может быть полезна для создания неминифицированных и минифицированных файлов с помощью одного и того же конвейера.

```js
const { src, dest } = require('gulp');
const babel = require('gulp-babel');
const uglify = require('gulp-uglify');
const rename = require('gulp-rename');

exports.default = function() {
  return src('src/*.js')
    .pipe(babel())
    .pipe(src('vendor/*.js'))
    .pipe(dest('output/'))
    .pipe(uglify())
    .pipe(rename({ extname: '.min.js' }))
    .pipe(dest('output/'));
}
```

## Режимы: потоковый, буферизованный и пустой

`src()` может работать в трех режимах: буферизации, потоковой передачи и пустоты. Они настраиваются с помощью [опций][src-options-api-docs] `buffer` и `read` в `src()`.

* Режим буферизации установлен по умолчанию и загружает содержимое файла в память. Плагины обычно работают в режиме буферизации, и многие из них не поддерживают режим потоковой передачи.
* Режим потоковой передачи существует в основном для работы с большими файлами, которые не помещаются в памяти, такими как гигантские изображения или фильмы. Содержимое передается из файловой системы небольшими порциями, а не загружается сразу. Если вам нужно использовать потоковый режим, поищите плагин, который его поддерживает, или напишите свой собственный.
* Пустой режим не содержит содержимого и полезен при работе только с метаданными файла.

[explaining-globs-docs]: ../getting-started/6-explaining-globs.md
[creating-tasks-docs]: ../getting-started/3-creating-tasks.md
[overlapping-globs-docs]: ../getting-started/6-explaining-globs.md#overlapping-globs
[node-streams-docs]: https://nodejs.org/api/stream.html
[symlink-api-docs]: ../api/symlink.md
[src-options-api-docs]: ../api/src.md#options

# Browserify + Uglify2 с исходными картами

[Browserify](https://github.com/browserify/browserify) стал важным и незаменимым инструментом, но для нормальной работы с gulp его необходимо обернуть.
Ниже приведен простой рецепт использования Browserify с полными исходными картами, которые разрешаются в исходные отдельные файлы.

Смотрите также: рецепт [Объединение потоков для обработки ошибок](https://github.com/gulpjs/gulp/blob/master/docs/recipes/combining-streams-to-handle-errors.md) для обработки ошибок с помощью browserify или убрать в свой поток.

Простой файл `gulpfile.js` для Browserify + Uglify2 с исходными картами:

``` javascript
'use strict';

var browserify = require('browserify');
var gulp = require('gulp');
var source = require('vinyl-source-stream');
var buffer = require('vinyl-buffer');
var uglify = require('gulp-uglify');
var sourcemaps = require('gulp-sourcemaps');
var log = require('gulplog');

gulp.task('javascript', function () {
  // настроить экземпляр browserify на основе задачи
  var b = browserify({
    entries: './entry.js',
    debug: true
  });

  return b.bundle()
    .pipe(source('app.js'))
    .pipe(buffer())
    .pipe(sourcemaps.init({loadMaps: true}))
        // Добавьте сюда задачи преобразования в конвейер.
        .pipe(uglify())
        .on('error', log.error)
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest('./dist/js/'));
});
```

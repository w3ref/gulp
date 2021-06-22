# Browserify + Transforms

[Browserify](https://github.com/browserify/browserify) стал важным и незаменимым инструментом, но для нормальной работы с gulp его необходимо обернуть.
Ниже приведен простой рецепт использования Browserify с преобразованиями.

Смотрите также: рецепт [Объединение потоков для обработки ошибок](https://github.com/gulpjs/gulp/blob/master/docs/recipes/combining-streams-to-handle-errors.md) для обработки ошибок с помощью browserify или убрать в свой поток.

``` javascript
'use strict';

var browserify = require('browserify');
var gulp = require('gulp');
var source = require('vinyl-source-stream');
var buffer = require('vinyl-buffer');
var log = require('gulplog');
var uglify = require('gulp-uglify');
var sourcemaps = require('gulp-sourcemaps');
var reactify = require('reactify');

gulp.task('javascript', function () {
  // настроить экземпляр browserify на основе задачи
  var b = browserify({
    entries: './entry.js',
    debug: true,
    // определение преобразований здесь позволит избежать сбоя вашего потока
    transform: [reactify]
  });

  return b.bundle()
    .pipe(source('app.js'))
    .pipe(buffer())
    .pipe(sourcemaps.init({loadMaps: true}))
        // Добавьте задачи преобразования в конвейер здесь.
        .pipe(uglify())
        .on('error', log.error)
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest('./dist/js/'));
});
```

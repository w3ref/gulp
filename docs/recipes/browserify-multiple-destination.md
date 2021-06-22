# Browserify + Globs ((несколько пунктов назначения)

В этом примере показано, как настроить задачу объединения нескольких точек входа в несколько пунктов назначения с помощью browserify.

Приведенная ниже задача `js` объединяет все файлы `.js` в `src/` как точки входа и записывает результаты в `dest/`.

```js
var gulp = require('gulp');
var browserify = require('browserify');
var log = require('gulplog');
var tap = require('gulp-tap');
var buffer = require('gulp-buffer');
var sourcemaps = require('gulp-sourcemaps');
var uglify = require('gulp-uglify');

gulp.task('js', function () {

  return gulp.src('src/**/*.js', {read: false}) // нет необходимости читать файл, потому что браузер это делает.

    // преобразовывать файловые объекты с помощью плагина gulp-tap
    .pipe(tap(function (file) {

      log.info('bundling ' + file.path);

      // заменить содержимое файла потоком пакета browserify
      file.contents = browserify(file.path, {debug: true}).bundle();

    }))

    // преобразовать содержимое потоковой передачи в содержимое буфера (поскольку gulp-sourcemaps не поддерживает содержимое потоковой передачи)
    .pipe(buffer())

    // загрузка и инициализация исходных карт
    .pipe(sourcemaps.init({loadMaps: true}))

    .pipe(uglify())

    // запись исходных карт
    .pipe(sourcemaps.write('./'))

    .pipe(gulp.dest('dest'));

});
```

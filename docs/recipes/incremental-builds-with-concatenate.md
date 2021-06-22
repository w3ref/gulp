# Инкрементное восстановление, включая работу с полными наборами файлов

Проблема с инкрементными перестроениями заключается в том, что вам часто нужно работать со _всеми_ обработанными файлами, а не только с отдельными файлами.
Например, вы можете захотеть выполнить линтинг и обертывание модуля только измененными файлами, а затем объединить его со всеми другими файлами с линзованием и оболочкой из модулей. Это сложно без использования временных файлов.

Для этого используйте [gulp-cached](https://github.com/wearefractal/gulp-cached) и [gulp-remember](https://github.com/ahaurw01/gulp-remember).

```js
var gulp = require('gulp');
var header = require('gulp-header');
var footer = require('gulp-footer');
var concat = require('gulp-concat');
var jshint = require('gulp-jshint');
var cached = require('gulp-cached');
var remember = require('gulp-remember');

var scriptsGlob = 'src/**/*.js';

gulp.task('scripts', function() {
  return gulp.src(scriptsGlob)
      .pipe(cached('scripts'))        // пропускать только измененные файлы,
      .pipe(jshint())                 // делать особые вещи с измененными файлами...
      .pipe(header('(function () {')) // например, jshinting ^^^
      .pipe(footer('})();'))          // и какая-то оболочка модуля
      .pipe(remember('scripts'))      // добавляет обратно все файлы в поток,
      .pipe(concat('app.js'))         // делает то, что требует всех файлов
      .pipe(gulp.dest('public/'));
});

gulp.task('watch', function () {
  var watcher = gulp.watch(scriptsGlob, gulp.series('scripts')); // смотреть те же файлы в нашей задаче скриптов
  watcher.on('change', function (event) {
    if (event.type === 'deleted') {                   // если файл удален, забудьте об этом
      delete cached.caches.scripts[event.path];       // gulp-cached удалит api
      remember.forget('scripts', event.path);         // gulp-remember удалит api
    }
  });
});
```

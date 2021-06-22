# Обработка события удаления на наблюдателе

Вы можете прослушивать события `'unlink'`, чтобы запускать наблюдатель, возвращенный из `gulp.watch`.
Это срабатывает, когда файлы удаляются, поэтому вы можете удалить файл из целевого каталога, используя что-то вроде:

```js
'use strict';

var del = require('del');
var path = require('path');
var gulp = require('gulp');
var header = require('gulp-header');
var footer = require('gulp-footer');

gulp.task('scripts', function() {
  return gulp.src('src/**/*.js', {base: 'src'})
    .pipe(header('(function () {\r\n\t\'use strict\'\r\n'))
    .pipe(footer('\r\n})();'))
    .pipe(gulp.dest('build'));
});

gulp.task('watch', function () {
  var watcher = gulp.watch('src/**/*.js', ['scripts']);

  watcher.on('unlink', function (filepath) {
    var filePathFromSrc = path.relative(path.resolve('src'), filepath);
    // Объединение абсолютного пути 'build', используемого gulp.dest в задаче сценариев
    var destFilePath = path.resolve('build', filePathFromSrc);
    del.sync(destFilePath);
  });
});
```

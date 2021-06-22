# Вывести как минифицированную, так и неминифицированную версию

Вывод как минифицированной, так и неминифицированной версии ваших объединенных файлов JavaScript может быть достигнут с помощью `gulp-rename` и двойного подключения к `dest` дважды (один раз перед минификацией и один раз после минификации):

```js
'use strict';

var gulp = require('gulp');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');

var DEST = 'build/';

gulp.task('default', function() {
  return gulp.src('foo.js')
    // Это выведет неминифицированную версию
    .pipe(gulp.dest(DEST))
    // Это уменьшит размер и переименует его в foo.min.js
    .pipe(uglify())
    .pipe(rename({ extname: '.min.js' }))
    .pipe(gulp.dest(DEST));
});

```

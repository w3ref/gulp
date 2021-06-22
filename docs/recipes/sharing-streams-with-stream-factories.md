# Совместное использование потоков с фабриками потоков

Если вы используете одни и те же плагины в нескольких задачах, вы можете обнаружить, что у вас появился зуд к DRY вещам. Этот метод позволит вам создать фабрики для разделения часто используемых цепочек потоков.

Мы будем использовать [lazypipe](https://github.com/OverZealous/lazypipe)чтобы выполнить эту работу.

Это наш образец файла:

```js
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var coffee = require('gulp-coffee');
var jshint = require('gulp-jshint');
var stylish = require('jshint-stylish');

gulp.task('bootstrap', function() {
  return gulp.src('bootstrap/js/*.js')
    .pipe(jshint())
    .pipe(jshint.reporter(stylish))
    .pipe(uglify())
    .pipe(gulp.dest('public/bootstrap'));
});

gulp.task('coffee', function() {
  return gulp.src('lib/js/*.coffee')
    .pipe(coffee())
    .pipe(jshint())
    .pipe(jshint.reporter(stylish))
    .pipe(uglify())
    .pipe(gulp.dest('public/js'));
});
```

а наш файл после использования lazypipe выглядит так:

```js
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var coffee = require('gulp-coffee');
var jshint = require('gulp-jshint');
var stylish = require('jshint-stylish');
var lazypipe = require('lazypipe');

// отдать lazypipe
var jsTransform = lazypipe()
  .pipe(jshint)
  .pipe(jshint.reporter, stylish)
  .pipe(uglify);

gulp.task('bootstrap', function() {
  return gulp.src('bootstrap/js/*.js')
    .pipe(jsTransform())
    .pipe(gulp.dest('public/bootstrap'));
});

gulp.task('coffee', function() {
  return gulp.src('lib/js/*.coffee')
    .pipe(coffee())
    .pipe(jsTransform())
    .pipe(gulp.dest('public/js'));
});
```

Вы можете видеть, что мы разделили наш конвейер JavaScript (JSHint + Uglify), который повторно использовался в нескольких задачах, в фабрику. Эти фабрики можно повторно использовать в любом количестве задач. Вы также можете объединять фабрики и объединять фабрики в цепочки для большего эффекта. Разделение каждого общего конвейера также дает вам одно центральное место для изменения, если вы решите изменить свой рабочий процесс.

# Передавать аргументы из командной строки

```js
// npm install --save-dev gulp gulp-if gulp-uglify minimist

var gulp = require('gulp');
var gulpif = require('gulp-if');
var uglify = require('gulp-uglify');

var minimist = require('minimist');

var knownOptions = {
  string: 'env',
  default: { env: process.env.NODE_ENV || 'production' }
};

var options = minimist(process.argv.slice(2), knownOptions);

gulp.task('scripts', function() {
  return gulp.src('**/*.js')
    .pipe(gulpif(options.env === 'production', uglify())) // только минифицировать в продакшене
    .pipe(gulp.dest('dist'));
});
```

Затем запустите gulp с помощью:

```sh
$ gulp scripts --env development
```

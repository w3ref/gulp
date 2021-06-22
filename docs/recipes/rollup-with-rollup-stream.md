# Свертывание с помощью rollup-stream

Как и Browserify, [Rollup](https://rollupjs.org/) является сборщиком и поэтому естественно вписывается в gulp, только если он находится в начале конвейера. В отличие от Browserify, Rollup изначально не создает поток в качестве вывода, и его необходимо обернуть, прежде чем он сможет занять эту позицию. [rollup-stream](https://github.com/Permutatrix/rollup-stream) делает это за вас, создавая выходные данные, аналогичные выходным данным Browserify метода&mdash;как `bundle()` в результате большинство рецептов Browserify здесь будут также работают с rollup-stream.

## Базовое применение

```js
// npm install --save-dev gulp @rollup/stream@1 vinyl-source-stream
var gulp = require('gulp');
var rollup = require('rollup-stream');
var source = require('vinyl-source-stream');

gulp.task('rollup', function() {
  return rollup({
      input: './src/main.js'
    })

    // дайте файлу имя, которое вы хотите выводить,
    .pipe(source('app.js'))

    // и вывод его в ./dist/app.js как обычно.
    .pipe(gulp.dest('./dist'));
});
```

## Использование с исходными картами

```js
// npm install --save-dev gulp @rollup/stream@1 gulp-sourcemaps vinyl-source-stream vinyl-buffer
// optional: npm install --save-dev gulp-rename
var gulp = require('gulp');
var rollup = require('rollup-stream');
var sourcemaps = require('gulp-sourcemaps');
//var rename = require('gulp-rename');
var source = require('vinyl-source-stream');
var buffer = require('vinyl-buffer');

gulp.task('rollup', function() {
  return rollup({
      input: './src/main.js',
      sourcemap: true,
      format: 'umd'
    })

    // укажите на входной файл.
    .pipe(source('main.js', './src'))

    // буфер вывода. большинство плагинов gulp, включая gulp-sourcemaps, не поддерживают потоки.
    .pipe(buffer())

    // указать gulp-sourcemaps, чтобы загрузить встроенную исходную карту, созданную с помощью rollup-stream.
    .pipe(sourcemaps.init({loadMaps: true}))

        // преобразуйте код дальше здесь.

    // если вы хотите выводить с другим именем из входного файла, используйте здесь gulp-rename.
    //.pipe(rename('index.js'))

    // запишите исходную карту вместе с выходным файлом.
    .pipe(sourcemaps.write('.'))

    // и вывод в ./dist/main.js как обычно.
    .pipe(gulp.dest('./dist'));
});
```

# Удаление файлов и папок

Возможно, вы захотите удалить некоторые файлы перед запуском сборки. Поскольку удаление файлов не влияет на содержимое файла, нет причин использовать плагин gulp. Отличная возможность использовать модуль vanilla node.

Давайте использовать модуль [`del`](https://github.com/sindresorhus/del) для этого примера, так как он поддерживает несколько файлов и [globbing](https://github.com/sindresorhus/multimatch#globbing-patterns):

```sh
$ npm install --save-dev gulp del
```

Представьте себе следующую файловую структуру:

```
.
├── dist
│   ├── report.csv
│   ├── desktop
│   └── mobile
│       ├── app.js
│       ├── deploy.json
│       └── index.html
└── src
```

В gulpfile мы хотим очистить содержимое папки `mobile` перед запуском нашей сборки:

```js
var gulp = require('gulp');
var del = require('del');

gulp.task('clean:mobile', function () {
  return del([
    'dist/report.csv',
    // здесь мы используем шаблон подстановки, чтобы сопоставить все внутри папки `mobile`
    'dist/mobile/**/*',
    // мы не хотим очищать этот файл, поэтому мы отменяем шаблон
    '!dist/mobile/deploy.json'
  ]);
});

gulp.task('default', gulp.series('clean:mobile'));
```

## Удалить файлы в конвейере

Возможно, вы захотите удалить некоторые файлы после их обработки в конвейере.

Мы будем использовать [vinyl-paths](https://github.com/sindresorhus/vinyl-paths), чтобы легко получить путь к файлам в потоке и передать его методу `del`.

```sh
$ npm install --save-dev gulp del vinyl-paths
```

Представьте себе следующую файловую структуру:

```
.
├── tmp
│   ├── rainbow.js
│   └── unicorn.js
└── dist
```

```js
var gulp = require('gulp');
var stripDebug = require('gulp-strip-debug'); // только как пример
var del = require('del');
var vinylPaths = require('vinyl-paths');

gulp.task('clean:tmp', function () {
  return gulp.src('tmp/*')
    .pipe(vinylPaths(del))
    .pipe(stripDebug())
    .pipe(gulp.dest('dist'));
});

gulp.task('default', gulp.series('clean:tmp'));
```

Это удалит только каталог `tmp`.

Делайте это только в том случае, если вы уже используете другие плагины в конвейере, в противном случае просто используйте модуль напрямую, так как `gulp.src` является дорогостоящим.

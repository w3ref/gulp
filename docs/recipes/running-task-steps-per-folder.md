# Создание файла для каждой папки

Если у вас есть набор папок, и вы хотите выполнить набор задач для каждой, например...

```
/scripts
/scripts/jquery/*.js
/scripts/angularjs/*.js
```

...и хотите получить...

```
/scripts
/scripts/jquery.min.js
/scripts/angularjs.min.js
```

...вам нужно сделать что-то вроде следующего...

``` javascript
var fs = require('fs');
var path = require('path');
var merge = require('merge-stream');
var gulp = require('gulp');
var concat = require('gulp-concat');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');

var scriptsPath = 'src/scripts';

function getFolders(dir) {
    return fs.readdirSync(dir)
      .filter(function(file) {
        return fs.statSync(path.join(dir, file)).isDirectory();
      });
}

gulp.task('scripts', function(done) {
   var folders = getFolders(scriptsPath);
   if (folders.length === 0) return done(); // нечего делать!
   var tasks = folders.map(function(folder) {
      return gulp.src(path.join(scriptsPath, folder, '/**/*.js'))
        // объединить в foldername.js
        .pipe(concat(folder + '.js'))
        // записать в вывод
        .pipe(gulp.dest(scriptsPath))
        // минификация
        .pipe(uglify())
        // переименовать в folder.min.js
        .pipe(rename(folder + '.min.js'))
        // записать в вывод again
        .pipe(gulp.dest(scriptsPath));
   });

   // обработать все оставшиеся файлы в scriptsPath root в файлы main.js и main.min.js
   var root = gulp.src(path.join(scriptsPath, '/*.js'))
        .pipe(concat('main.js'))
        .pipe(gulp.dest(scriptsPath))
        .pipe(uglify())
        .pipe(rename('main.min.js'))
        .pipe(gulp.dest(scriptsPath));

   return merge(tasks, root);
});
```

Несколько примечаний:

- `folders.map` - выполняет функцию один раз для каждой папки и возвращает асинхронный поток
- `merge` - объединяет потоки и завершается только тогда, когда все потоки исходят, заканчиваются

# Объединение потоков для обработки ошибок

По умолчанию, выдача ошибки в потоке будет вызывать его, если у него еще нет слушателя, прикрепленного к событию `error`. Это становится немного сложнее, когда вы работаете с более длинными конвейерами потоков.

Используя [stream-combiner2](https://github.com/substack/stream-combiner2), вы можете превратить серию потоков в один поток, то есть вам нужно только прослушивать событие `error` в одном месте в ваш код.

Вот пример использования его в gulpfile:

```js
var combiner = require('stream-combiner2');
var uglify = require('gulp-uglify');
var gulp = require('gulp');

gulp.task('test', function() {
  return combiner.obj([
      gulp.src('bootstrap/js/*.js'),
      uglify(),
      gulp.dest('public/bootstrap')
    ])
    // любые ошибки в вышеуказанных потоках будут
    // перехвачены этим слушателем, а не выданы:
    .on('error', console.error.bind(console));
});
```

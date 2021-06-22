# Зачем использовать Pump?

При использовании `pipe` из потоков Node.js ошибки не распространяются вперед через конвейерные потоки, и исходные потоки не закрываются, если целевой поток закрыт.
Модуль [`pump`][pump] нормализует эти проблемы и передает вам ошибки в обратном вызове.

## Типичный пример gulpfile

Обычный шаблон в файлах gulp - просто вернуть поток Node.js и ожидать, что инструмент gulp обработает ошибки.

```javascript
// пример обычного gulpfile
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('compress', function () {
  // возвращает поток Node.js, но не обрабатывает сообщения об ошибках
  return gulp.src('lib/*.js')
    .pipe(uglify())
    .pipe(gulp.dest('dist'));
});
```

![pipe error](pipe-error.png)

В одном из файлов JavaScript есть ошибка, но это сообщение об ошибке не имеет смысла. Вы хотите знать, в каком файле и строке содержится ошибка. Так что это за беспорядок?

Когда в потоке возникает ошибка, поток Node.js запускает событие 'error', но если для этого события нет обработчика, оно вместо этого переходит к определенному обработчику [uncaught exception][uncaughtException].
Задокументировано поведение обработчика неперехваченных исключений по умолчанию:

> По умолчанию Node.js обрабатывает такие исключения, выводя трассировку стека в stderr и завершая работу.

## Обработка ошибок

Поскольку разрешать ошибкам попадать в обработчик неперехваченных исключений бесполезно, мы должны обрабатывать исключения должным образом. Давайте быстро попробуем.

```javascript
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('compress', function () {
  return gulp.src('lib/*.js')
    .pipe(uglify())
    .pipe(gulp.dest('dist'))
    .on('error', function(err) {
      console.error('Error in compress task', err.toString());
    });
});
```

К сожалению, функция `pipe` потока Node.js не пересылает ошибки по цепочке, поэтому этот обработчик ошибок обрабатывает только ошибки, заданные `gulp.dest`. Вместо этого нам нужно обрабатывать ошибки для каждого потока.

```javascript
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('compress', function () {
  function createErrorHandler(name) {
    return function (err) {
      console.error('Error from ' + name + ' in compress task', err.toString());
    };
  }

  return gulp.src('lib/*.js')
    .on('error', createErrorHandler('gulp.src'))
    .pipe(uglify())
    .on('error', createErrorHandler('uglify'))
    .pipe(gulp.dest('dist'))
    .on('error', createErrorHandler('gulp.dest'));
});
```

Это очень сложно добавить в каждую из ваших задач gulp, и это легко забыть сделать. Кроме того, он все еще несовершенен, поскольку не сигнализирует системе задач gulp, что задача не выполнена. Мы можем это исправить, и мы можем решить другие неприятные проблемы, связанные с распространением ошибок с потоками, но это еще больше работы!

## Использование pump

Модуль [`pump`][pump] представляет собой своего рода чит-код. Это оболочка для функции `pipe`, которая обрабатывает эти случаи за вас, так что вы можете прекратить взламывать свои файлы gulp и вернуться к взламыванию новых функций в своем приложении.

```javascript
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var pump = require('pump');

gulp.task('compress', function (cb) {
  pump([
      gulp.src('lib/*.js'),
      uglify(),
      gulp.dest('dist')
    ],
    cb
  );
});
```

Система задач gulp предоставляет задачу gulp с обратным вызовом, который может сигнализировать об успешном завершении задачи (вызывается без аргументов) или сбое задачи (вызывается с аргументом Error). К счастью, это тот же самый формат, который использует `pump`!

![pump error](pump-error.png)

Теперь очень ясно, в каком плагине произошла ошибка, в чем действительно была ошибка, в каком файле и в каком номере строки.

[pump]: https://github.com/mafintosh/pump
[uncaughtException]: https://nodejs.org/api/process.html#process_event_uncaughtexception

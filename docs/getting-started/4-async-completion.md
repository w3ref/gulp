<!-- front-matter
id: async-completion
title: Асинхронное выполнение
hide_title: true
sidebar_label: Асинхронное выполнение
-->

# Асинхронное выполнение

Node библиотеки обрабатывают асинхронность различными способами. Наиболее распространенным шаблоном является [обратные вызовы error-first][node-api-error-first-callbacks], но вы также можете встретить [потоки][stream-docs], [промисы][promise-docs], [эмиттеры событий][event-emitter-docs], [дочерние процессы][child-process-docs] или [наблюдаемые][observable-docs]. Задачи Gulp нормализуют все эти типы асинхронности.

## Сигнал о завершении задачи

Когда поток, обещание, эмиттер событий, дочерний процесс или наблюдаемый объект возвращается из задачи, успех или ошибка сообщает gulp, продолжать или завершать. Если в задаче возникает ошибка, gulp немедленно завершится и покажет эту ошибку.

При составлении задач с помощью `series()`, ошибка завершает составление и никакие дальнейшие задачи не будут выполняться. При составлении задач с помощью `parallel()`, ошибка завершит композицию, но другие параллельные задачи могут завершиться, а могут и не завершиться.

### Вернуть поток

```js
const { src, dest } = require('gulp');

function streamTask() {
  return src('*.js')
    .pipe(dest('output'));
}

exports.default = streamTask;
```

### Вернуть промис

```js
function promiseTask() {
  return Promise.resolve('the value is ignored');
}

exports.default = promiseTask;
```

### Вернуть эмиттер события

```js
const { EventEmitter } = require('events');

function eventEmitterTask() {
  const emitter = new EventEmitter();
  // Emit должен происходить асинхронно, иначе gulp еще не слушает.
  setTimeout(() => emitter.emit('finish'), 250);
  return emitter;
}

exports.default = eventEmitterTask;
```

### Вернуть дочерний процесс

```js
const { exec } = require('child_process');

function childProcessTask() {
  return exec('date');
}

exports.default = childProcessTask;
```

### Вернуть наблюдаемый

```js
const { Observable } = require('rxjs');

function observableTask() {
  return Observable.of(1, 2, 3);
}

exports.default = observableTask;
```

### Использование обратного вызова при ошибке

Если ваша задача ничего не вернула, вы должны использовать обратный вызов error-first, чтобы сигнализировать о завершении. Обратный вызов будет передан вашей задаче как единственный аргумент - в примерах ниже он называется `cb()`.

```js
function callbackTask(cb) {
  // `cb()` должен вызываться некоторой асинхронной работой
  cb();
}

exports.default = callbackTask;
```

To indicate to gulp that an error occurred in a task using an error-first callback, call it with an `Error` as the only argument.

```js
function callbackError(cb) {
  // `cb()` должен вызываться некоторой асинхронной работой
  cb(new Error('kaboom'));
}

exports.default = callbackError;
```

Однако вы часто будете передавать этот обратный вызов другому API вместо того, чтобы вызывать его самостоятельно.

```js
const fs = require('fs');

function passingCallback(cb) {
  fs.access('gulpfile.js', cb);
}

exports.default = passingCallback;
```

## Отсутствие синхронных задач

Синхронные задачи больше не поддерживаются. Они часто приводили к незаметным ошибкам, которые было трудно отладить, например, к забыванию вернуть потоки из задачи.

Когда вы видите предупреждение _«Вы забыли сигнализировать об асинхронном завершении?»_, значит, ни один из упомянутых выше методов не использовался. Вам нужно будет использовать обратный вызов error-first или вернуть поток, промис, эмиттер событий, дочерний процесс или наблюдаемое, чтобы решить проблему.

## Использование async/await

Если вы не используете какие-либо из предыдущих параметров, вы можете определить свою задачу как [`async` функцию][async-await-docs], которая заключает вашу задачу в обещание. Это позволяет вам работать с обещаниями синхронно с помощью `await` и использовать другой синхронный код.

```js
const fs = require('fs');

async function asyncAwaitTask() {
  const { version } = JSON.parse(fs.readFileSync('package.json', 'utf8'));
  console.log(version);
  await Promise.resolve('some result');
}

exports.default = asyncAwaitTask;
```

[node-api-error-first-callbacks]: https://nodejs.org/api/errors.html#errors_error_first_callbacks
[stream-docs]: https://nodejs.org/api/stream.html#stream_stream
[promise-docs]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises
[event-emitter-docs]: https://nodejs.org/api/events.html#events_events
[child-process-docs]: https://nodejs.org/api/child_process.html#child_process_child_process
[observable-docs]: https://github.com/tc39/proposal-observable/blob/master/README.md
[async-await-docs]: https://developers.google.com/web/fundamentals/primers/async-functions

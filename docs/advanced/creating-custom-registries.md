<!-- front-matter
id: creating-custom-registries
title: Создание пользовательских реестров
hide_title: true
sidebar_label: Создание пользовательских реестров
-->

# Создание пользовательских реестров

Позволяет подключать настраиваемые реестры к системе задач, которая может предоставлять общие задачи или расширенную функциональность. Реестры регистрируются с помощью [`registry()`][registry-api-docs].

## Структура

Чтобы быть принятыми gulp, пользовательские реестры должны соответствовать определенному формату.

```js
// как функция
function TestRegistry() {}

TestRegistry.prototype.init = function (gulpInst) {}
TestRegistry.prototype.get = function (name) {}
TestRegistry.prototype.set = function (name, fn) {}
TestRegistry.prototype.tasks = function () {}

// как класс
class TestRegistry {
  init(gulpInst) {}

  get(name) {}

  set(name, fn) {}

  tasks() {}
}
```

Если экземпляр реестра, переданный в `registry()`, не имеет всех четырех методов, будет выдана ошибка.

## Регистрация

Если мы хотим зарегистрировать наш пример реестра сверху, нам нужно будет передать его экземпляр в `registry()`.

```js
const { registry } = require('gulp');

// ... Код установки TestRegistry

// хорошо!
registry(new TestRegistry())

// плохо!
registry(TestRegistry())
// Это вызовет ошибку: «Необходимо создать экземпляры пользовательских реестров, но похоже, что вы передали конструктор».
```

## Методы

### `init(gulpInst)`

Метод реестра `init()` вызывается в самом конце функции `registry()`. Экземпляр gulp, переданный как единственный аргумент (`gulpInst`), может использоваться для предварительного определения задач с помощью `gulpInst.task(taskName, fn)`.

#### Параметры

| параметр  | тип  | примечание |
|:---------:|:----:|------------|
| gulpInst | object | Экземпляр gulp. |

### `get(name)`

Метод `get()` получает имя задачи `name`, которую пользовательский реестр должен разрешить и вернуть, или `undefined`, если задачи с таким именем не существует.

#### Параметры

| параметр  | тип  | примечание |
|:---------:|:----:|------------|
| name | string | Название задачи, которую нужно получить. |

### `set(name, fn)`

Метод `set()` получает имя задачи `name` и `fn`. Это вызывается внутри `task()` для предоставления задач, зарегистрированных пользователями, настраиваемым реестрам.

#### Параметры

| параметр  | тип  | примечание |
|:---------:|:----:|------------|
| name | string | Название устанавливаемой задачи. |
| fn | function | Задача, которую необходимо установить. |

### `tasks()`

Должен возвращать объект со списком всех задач в реестре.

## Сценарии использования

### Совместное использование задач

Чтобы совместно использовать общие задачи со всеми вашими проектами, вы можете выставить метод `init` в реестре, и он получит экземпляр gulp в качестве единственного аргумента. Затем вы можете использовать `gulpInst.task(name, fn)` для регистрации предопределенных задач.

Например, вы можете захотеть поделиться задачей `clean`:

```js
const fs = require('fs');
const util = require('util');

const DefaultRegistry = require('undertaker-registry');
const del = require('del');

function CommonRegistry(opts){
  DefaultRegistry.call(this);

  opts = opts || {};

  this.buildDir = opts.buildDir || './build';
}

util.inherits(CommonRegistry, DefaultRegistry);

CommonRegistry.prototype.init = function(gulpInst) {
  const buildDir = this.buildDir;
  const exists = fs.existsSync(buildDir);

  if(exists){
    throw new Error('Невозможно инициализировать общие задачи. Каталог ' + buildDir + ' существует.');
  }

  gulpInst.task('clean', function(){
    return del([buildDir]);
  });
}

module.exports = CommonRegistry;
```

Затем, чтобы использовать его в проекте:

```js
const { registry, series, task } = require('gulp');
const CommonRegistry = require('myorg-common-tasks');

registry(new CommonRegistry({ buildDir: '/dist' }));

task('build', series('clean', function build(cb) {
  // делать что-то
  cb();
}));
```

### Совместное использование функций

Контролируя, как задачи добавляются в реестр, вы можете их декорировать.

Например, если вы хотите, чтобы все задачи разделяли некоторые данные, вы можете использовать настраиваемый реестр, чтобы привязать их к этим данным. Обязательно верните измененную задачу в соответствии с описанием методов реестра выше:

```js
const { registry, series, task } = require('gulp');
const util = require('util');
const DefaultRegistry = require('undertaker-registry');

// Некоторая задача определена где-то еще
const BuildRegistry = require('./build.js');
const ServeRegistry = require('./serve.js');

function ConfigRegistry(config){
  DefaultRegistry.call(this);
  this.config = config;
}

util.inherits(ConfigRegistry, DefaultRegistry);

ConfigRegistry.prototype.set = function set(name, fn) {
  var bound = fn.bind(this.config);
  // Сохраните внутренние свойства и метаданные задачи.
  var task = Object.assign(bound, fn);
  // `DefaultRegistry` использует `this._tasks` для хранения.
  this._tasks[name] = task;
  return task;
};

registry(new BuildRegistry());
registry(new ServeRegistry());

// `registry` сбросит каждую задачу в реестре с
// `ConfigRegistry.prototype.set`, который привяжет их к объекту конфигурации.
registry(new ConfigRegistry({
  src: './src',
  build: './build',
  bindTo: '0.0.0.0:8888'
}));

task('default', series('clean', 'build', 'serve', function(cb) {
  console.log('Server bind to ' + this.bindTo);
  console.log('Serving' + this.build);
  cb();
}));
```

## Примеры

* [undertaker-registry][undertaker-registry-example]: Реестр Gulp 4 по умолчанию.
* [undertaker-common-tasks][undertaker-common-tasks-example]: Проверочный настраиваемый реестр, который предварительно определяет задачи.
* [undertaker-task-metadata][undertaker-task-metadata-example]: Проверочный настраиваемый реестр, который прикрепляет метаданные к каждой задаче.

[registry-api-docs]: ../api/registry.md
[undertaker-registry-example]: https://github.com/gulpjs/undertaker-registry
[undertaker-common-tasks-example]: https://github.com/gulpjs/undertaker-common-tasks
[undertaker-task-metadata-example]: https://github.com/gulpjs/undertaker-task-metadata

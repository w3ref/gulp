<!-- front-matter
id: registry
title: registry()
hide_title: true
sidebar_label: registry()
-->

# registry()

Позволяет подключать настраиваемые реестры к системе задач, которая может предоставлять общие задачи или расширенную функциональность.

**Примечание:** В настраиваемый реестр будут добавлены только задачи, зарегистрированные с помощью `task()`. Функции задач, переданные непосредственно в `series()` или `parallel()`, не будут предоставлены - если вам нужно настроить поведение реестра, составьте задачи со строковыми ссылками.

При назначении нового реестра каждая задача из текущего реестра будет перенесена, а текущий реестр будет заменен новым. Это позволяет добавлять несколько настраиваемых реестров в последовательном порядке.

Смотрите [Создание настраиваемых реестров][creating-custom-registries] для получения дополнительной информации.

## Применение

```js
const { registry, task, series } = require('gulp');
const FwdRef = require('undertaker-forward-reference');

registry(FwdRef());

task('default', series('forward-ref'));

task('forward-ref', function(cb) {
  // тело опущено
  cb();
});
```

## Подпись

```js
registry([registryInstance])
```

### Параметры

| параметр | тип | примечание |
|:--------------:|:-----:|--------|
| registryInstance | object | Экземпляр, а не класс настраиваемого реестра. |

### Возвращается

Если будет передан объект `registryInstance`, ничего не будет возвращено. Если аргументы не переданы, возвращает текущий экземпляр реестра.

### Ошибки

#### Неверный параметр

Когда конструктор (вместо экземпляра) передается как `registryInstance`, выдает ошибку с сообщением:

> Custom registries must be instantiated, but it looks like you passed a constructor.

#### Missing `get` method

Когда реестр без метода `get` передается как `registryInstance`, выдает ошибку с сообщением:

> Custom registry must have `get` function.

#### Missing `set` method

Когда реестр без метода `set` передается как `registryInstance`, выдает ошибку с сообщением:

> Custom registry must have `set` function.

#### Missing `init` method

Когда реестр без метода `init` передается как `registryInstance`, выдает ошибку с сообщением:

> Custom registry must have `init` function"

#### Missing `tasks` method

Когда реестр без метода `tasks` передается как `registryInstance`, выдает ошибку с сообщением:

> Custom registry must have `tasks` function.

[creating-custom-registries]: ../advanced/creating-custom-registries.md

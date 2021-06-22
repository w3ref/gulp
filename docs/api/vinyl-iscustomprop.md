<!-- front-matter
id: vinyl-iscustomprop
title: Vinyl.isCustomProp()
hide_title: true
sidebar_label: Vinyl.isCustomProp()
-->

# Vinyl.isCustomProp()

Определяет, находится ли свойство под внутренним управлением Vinyl. Используется Vinyl при установке значений внутри конструктора или при копировании свойств в методе экземпляра `clone()`.

Этот метод полезен при расширении класса Vinyl. Подробно в [Расширение винила][extending-vinyl-section] ниже.

## Применение

```js
const Vinyl = require('vinyl');

Vinyl.isCustomProp('sourceMap') === true;
Vinyl.isCustomProp('path') === false;
```

## Подпись

```js
Vinyl.isCustomProp(property)
```

### Параметры

| параметр | тип | примечание |
|:--------------:|:------:|-------|
| property | string | Имя свойства для проверки. |

### Возвращается

Истинно, если свойство не управляется изнутри.

## Расширение Vinyl

При внутреннем управлении настраиваемыми свойствами статический метод `isCustomProp` должен быть расширен и возвращать `false` при запросе одного из настраиваемых свойств.

```js
const Vinyl = require('vinyl');

const builtInProps = ['foo', '_foo'];

class SuperFile extends Vinyl {
  constructor(options) {
    super(options);
    this._foo = 'example internal read-only value';
  }

  get foo() {
    return this._foo;
  }

  static isCustomProp(name) {
    return super.isCustomProp(name) && builtInProps.indexOf(name) === -1;
  }
}
```

В приведенном выше примере `foo` и `_foo` не будут присвоены новому объекту при клонировании или переданы в `options` в `new SuperFile(options)`.

Если ваши настраиваемые свойства или логика требуют особой обработки во время клонирования, переопределите метод `clone` при расширении Vinyl.

[extending-vinyl-section]: #extending-vinyl

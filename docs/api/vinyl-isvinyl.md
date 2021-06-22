<!-- front-matter
id: vinyl-isvinyl
title: Vinyl.isVinyl()
hide_title: true
sidebar_label: Vinyl.isVinyl()
-->

# Vinyl.isVinyl()

Определяет, является ли объект экземпляром Vinyl. Используйте этот метод вместо `instanceof`.

**Примечание**: В этом методе используется внутреннее свойство, которое не раскрывается в некоторых более старых версиях Vinyl, что приводит к ложноотрицательному результату при использовании устаревшей версии.

## Применение

```js
const Vinyl = require('vinyl');

const file = new Vinyl();
const notAFile = {};

Vinyl.isVinyl(file) === true;
Vinyl.isVinyl(notAFile) === false;
```

## Подпись

```js
Vinyl.isVinyl(file);
```

### Параметры

| параметр | тип | примечание |
|:--------------:|:------:|-------|
| file | object | Объект проверки. |

### Возвращается

Истинно, если объект `file` является экземпляром Vinyl.

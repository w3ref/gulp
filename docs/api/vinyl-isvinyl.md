<!-- front-matter
id: vinyl-isvinyl
title: Vinyl.isVinyl()
hide_title: true
sidebar_label: Vinyl.isVinyl()
-->

# Vinyl.isVinyl()

Determines if an object is a Vinyl instance. Use this method instead of `instanceof`.

**Note**: This method uses an internal property that some older versions of Vinyl didn't expose resulting in a false negative if using an outdated version.

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
| file | object | The object to check. |

### Возвращается

True if the `file` object is a Vinyl instance.

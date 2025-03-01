<!-- front-matter
id: symlink
title: symlink()
hide_title: true
sidebar_label: symlink()
-->

# symlink()

Создает поток для связывания объектов [Vinyl][vinyl-concepts] с файловой системой.

## Применение

```js
const { src, symlink } = require('gulp');

function link() {
  return src('input/*.js')
    .pipe(symlink('output/'));
}

exports.link = link;
```

## Подпись

```js
symlink(directory, [options])
```

### Параметры

| параметр | тип | примечание |
|:--------------:|:-----:|--------|
| directory<br />**(обязательный)** | string<br />function | Путь к выходному каталогу, в котором будут созданы символические ссылки. Если функция используется, она будет вызываться с каждым объектом Vinyl и должна возвращать строковый путь к каталогу. |
| options | object | Подробнее в [Опциях][options-section] ниже. |

### Возвращается

Поток, который можно использовать в середине или в конце конвейера для создания символических ссылок в файловой системе.
Всякий раз, когда объект Vinyl проходит через поток, он создает символическую ссылку на исходный файл в файловой системе в данном каталоге.

Всякий раз, когда в файловой системе создается символическая ссылка, объект Vinyl будет изменен.

* Свойства `cwd`, `base` и `path` будут обновлены, чтобы соответствовать созданной символической ссылке.
* Свойство `stat` будет обновлено, чтобы соответствовать символической ссылке в файловой системе.
* Свойство `contents` будет установлено в `null`.
* Свойство `symlink` будет добавлено или заменено исходным путем..

**Примечание:** В Windows ссылки каталогов по умолчанию создаются с использованием переходов. Опция `useJunctions` отключает это поведение.

### Ошибки

Когда `directory` является пустой строкой, выдает ошибку с сообщением: "Invalid symlink() folder argument. Please specify a non-empty string or a function."

Когда `directory` не является строкой или функцией, выдает ошибку с сообщением: "Invalid symlink() folder argument. Please specify a non-empty string or a function."

Когда `directory` - это функция, которая возвращает пустую строку или `undefined`, выдает ошибку с сообщением: "Invalid output folder".

### Опции

**Для параметров, которые принимают функцию, переданная функция будет вызываться с каждым объектом Vinyl и должна возвращать значение другого перечисленного типа.**

| наименование | тип | по умолчанию | примечание |
|:-------:|:------:|-----------|-------|
| cwd | string<br />function | `process.cwd()` |Каталог, который будет объединен с любым относительным путем для формирования абсолютного пути. Игнорируется для абсолютных путей. Используйте, чтобы избежать объединения `directory` с `path.join()`. |
| dirMode | number<br />function | | Режим, используемый при создании каталогов. Если не установлен, будет использоваться режим процесса. |
| overwrite | boolean<br />function | true | Если установлено значение `true`, существующие файлы перезаписываются с тем же путем. |
| relativeSymlinks | boolean<br />function | false | Если установлено значение `false`, любые созданные символические ссылки будут абсолютными.<br />**Примечание**: Игнорируется, если создается соединение, поскольку они должны быть абсолютными. |
| useJunctions | boolean<br />function | true | Этот параметр актуален только в Windows и в других местах игнорируется. Если установлено значение `true`, создает символическую ссылку на каталог в виде соединения. Подробно в [Символические ссылки в Windows][symbolic-links-section] ниже. |

## Символические ссылки в Windows

При создании символических ссылок в Windows, аргумент `type` передается методу Node `fs.symlink()`, который определяет тип связываемого целевого объекта. Тип ссылки установлен на:

* `'file'`, если целью является обычный файл
* `'junction'`, если целью является каталог
* `'dir'`, если целью является каталог, и пользователь отключает опцию `useJunctions`

Если вы попытаетесь создать висящую (указывающую на несуществующую цель) ссылку, тип ссылки не может быть определен автоматически. В этих случаях поведение будет варьироваться в зависимости от того, создается ли висячая ссылка через `symlink()` или через `dest()`.

Для висящих ссылок, созданных с помощью `symlink()`, входящий объект Vinyl представляет цель, поэтому его статистика будет определять желаемый тип ссылки. Если `isDirectory()` возвращает false, создается ссылка на файл `'file'`, в противном случае создается ссылка `'junction'` или `'dir'` в зависимости от значения параметра `useJunctions`.

Для висящих ссылок, созданных с помощью `dest()`, входящий объект Vinyl представляет ссылку - обычно загружается с диска через `src(..., { resolveSymlinks: false })`. В этом случае тип ссылки не может быть обоснованно определен и по умолчанию используется `'file'`. Это может вызвать непредвиденное поведение при создании висячей ссылки на каталог. **Избегайте этого сценария.**

[options-section]: #options
[symbolic-links-section]: #symbolic-links-on-windows
[vinyl-concepts]: ../api/concepts.md#vinyl

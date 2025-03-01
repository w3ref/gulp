<!-- front-matter
id: dest
title: dest()
hide_title: true
sidebar_label: dest()
-->

# dest()

Создает поток для записи объектов [Винил][vinyl-concepts] в файловую систему.

## Применение

```js
const { src, dest } = require('gulp');

function copy() {
  return src('input/*.js')
    .pipe(dest('output/'));
}

exports.copy = copy;
```

## Подпись

```js
dest(directory, [options])
```

### Параметры

| параметр | тип | примечание |
|:--------------:|:-----:|--------|
| directory<br />**(required)** | string<br />function | Путь к выходному каталогу, в который будут записаны файлы. Если функция используется, она будет вызываться с каждым объектом Vinyl и должна возвращать строковый путь к каталогу. |
| options | object | Подробнее в  [Опциях][options-section] ниже. |

### Возвращается

Поток, который можно использовать в середине или в конце конвейера для создания файлов в файловой системе.
Всякий раз, когда объект Vinyl проходит через поток, он записывает содержимое и другие детали в файловую систему в данном каталоге. Если объект Vinyl имеет свойство `symlink`, символическая ссылка будет создана вместо записи содержимого. После создания файла его [метаданные будут обновлены][metadata-updates-section] для соответствия объекту Vinyl.

Каждый раз, когда файл создается в файловой системе, объект Vinyl будет изменен.

* Свойства `cwd`, `base` и `path` будут обновлены, чтобы соответствовать созданному файлу.
* Свойство `stat` будет обновлено, чтобы соответствовать файлу в файловой системе.
* Если свойство `contents` является потоком, оно будет сброшено, чтобы его можно было прочитать снова.

### Ошибки

Когда `directory` является пустой строкой, выдает ошибку с сообщением: "Invalid dest() folder argument. Please specify a non-empty string or a function."

Когда `directory` не является строкой или функцией, выдает ошибку с сообщением: "Invalid dest() folder argument. Please specify a non-empty string or a function."

Когда `directory` - это функция, которая возвращает пустую строку или `undefined`, выдает ошибку с сообщением: "Invalid output folder".

### Опции

**Для параметров, которые принимают функцию, переданная функция будет вызываться с каждым объектом Vinyl и должна возвращать значение другого перечисленного типа.**

| наименование | тип | по умолчанию | примечание |
|:-------:|:------:|-----------|-------|
| cwd | string<br />function | `process.cwd()` | Каталог, который будет объединен с любым относительным путем, чтобы сформировать абсолютный путь. Игнорируется для абсолютных путей. Используйте, чтобы избежать объединения `directory` с `path.join()`. |
| mode | number<br />function | `stat.mode` объекта Vinyl | Режим, используемый при создании файлов. Если не установлен и параметр `stat.mode` отсутствует, вместо него будет использоваться режим процесса. |
| dirMode | number<br />function | | Режим, используемый при создании каталогов. Если не установлен, будет использоваться режим процесса. |
| overwrite | boolean<br />function | true | Если установлено значение `true`, существующие файлы перезаписываются с тем же путем. |
| append | boolean<br />function | false | Если `true`, добавляет содержимое в конец файла вместо замены существующего содержимого. |
| sourcemaps | boolean<br />string<br />function | false | Если `true`, записывает встроенные исходные карты в выходной файл. Указание строкового пути `string` приведет к записи внешних [sourcemaps][sourcemaps-section] по заданному пути. |
| relativeSymlinks | boolean<br />function | false | Если установлено значение `false`, любые созданные символические ссылки будут абсолютными.<br />**Примечание**: Игнорируется, если создается соединение, поскольку они должны быть абсолютными. |
| useJunctions | boolean<br />function | true | Этот параметр актуален только в Windows и в других местах игнорируется. Если задано значение `true`, создает символическую ссылку каталога в виде соединения. Подробно в [Символические ссылки в Windows][symbolic-links-section] ниже. |

## Обновления метаданных

Каждый раз, когда поток `dest()` создает файл, объекты Vinyl `mode`, `mtime` и `atime` сравниваются с созданным файлом. Если они отличаются, созданный файл будет обновлен, чтобы отразить метаданные объекта Vinyl. Если эти свойства совпадают или gulp не имеет разрешений на внесение изменений, попытка автоматически пропускается.

Эта функция отключена в Windows или других операционных системах, которые не поддерживают методы Node `process.getuid()` или `process.geteuid()`. Это происходит из-за того, что Windows дает неожиданные результаты при использовании `fs.fchmod()` и `fs.futimes()`.

**Примечание**: Метод `fs.futimes()` внутренне конвертирует временные метки `mtime` и `atime` в секунды. Это деление на 1000 может вызвать некоторую потерю точности в 32-битных операционных системах.

## Исходные карты

Поддержка Sourcemap встроена непосредственно в `src()` и `dest()`, но по умолчанию отключена. Включите его для создания встроенных или внешних исходных карт.

Встроенные исходные карты:

```js
const { src, dest } = require('gulp');
const uglify = require('gulp-uglify');

src('input/**/*.js', { sourcemaps: true })
  .pipe(uglify())
  .pipe(dest('output/', { sourcemaps: true }));
```

Внешние исходные карты:

```js
const { src, dest } = require('gulp');
const uglify = require('gulp-uglify');

src('input/**/*.js', { sourcemaps: true })
  .pipe(uglify())
  .pipe(dest('output/', { sourcemaps: '.' }));
```

## Символические ссылки в Windows

При создании символьных ссылок в Windows, аргумент `type` передается методу Node `fs.symlink()`, который указывает тип связываемой цели. Тип ссылки установлен на:

* `'file'`, когда целью является обычный файл
* `'junction'`, когда целью является каталог
* `'dir'`, когда целью является каталог и пользователь отключает параметр `useJunctions`

Если вы попытаетесь создать висящую (указывающую на несуществующую цель) ссылку, тип ссылки не может быть определен автоматически. В этих случаях поведение будет варьироваться в зависимости от того, создается ли висячая ссылка через `symlink()` или через `dest()`.

Для висящих ссылок, созданных с помощью `symlink()`, входящий объект Vinyl представляет цель, поэтому его статистика будет определять желаемый тип ссылки. Если `isDirectory()` возвращает `false`, создается ссылка на `'file'`, в противном случае создается ссылка `'junction'` или `'dir'` в зависимости от значения параметра `useJunctions`.

Для висящих ссылок, созданных с помощью `dest()`, входящий объект Vinyl представляет ссылку - обычно загружается с диска через `src(..., { resolveSymlinks: false })`. В этом случае тип ссылки не может быть обоснованно определен и по умолчанию используется `'file'`. Это может вызвать непредвиденное поведение, если вы создаете висящую ссылку на каталог. **Избегайте этого сценария.**

[sourcemaps-section]: #sourcemaps
[symbolic-links-section]: #symbolic-links-on-windows
[options-section]: #options
[metadata-updates-section]: #metadata-updates
[vinyl-concepts]: ../api/concepts.md#vinyl

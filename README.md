<p align="center">
  <a href="https://gulpjs.com">
    <img height="257" width="114" src="https://raw.githubusercontent.com/gulpjs/artwork/master/gulp-2x.png">
  </a>
  <p align="center">Система потоковой сборки</p>
</p>

[![NPM version][npm-image]][npm-url] [![Downloads][downloads-image]][npm-url] [![Azure Pipelines Build Status][azure-pipelines-image]][azure-pipelines-url] [![Build Status][travis-image]][travis-url] [![AppVeyor Build Status][appveyor-image]][appveyor-url] [![Coveralls Status][coveralls-image]][coveralls-url] [![OpenCollective Backers][backer-badge]][backer-url] [![OpenCollective Sponsors][sponsor-badge]][sponsor-url] [![Gitter chat][gitter-image]][gitter-url]

## Что такое gulp?

- **Автоматизация** - gulp - это набор инструментов, который помогает автоматизировать болезненные или трудоемкие задачи в рабочем процессе разработки.
- **Независимость от платформы** - Интеграции встроены во все основные IDE, и люди используют gulp с PHP, .NET, Node.js, Java и другими платформами.
- **Сильная экосистема** - Используйте модули npm, чтобы делать все, что захотите + более 3000 тщательно подобранных плагинов для потокового преобразования файлов.
- **Простота** - Благодаря минимальной площади API-интерфейса gulp легко изучить и использовать.

## Что нового в 4.0?!

* Система задач была переписана с нуля, позволяя составлять задачи с использованием методов `series()` и `parallel()`.
* Наблюдатель был обновлен, теперь он использует chokidar (больше нет необходимости в gulp-watch!), с паритетом функций с нашей системой задач.
* Первоклассная поддержка была добавлена для инкрементных сборок с использованием `lastRun()`.
* Был предоставлен метод `symlink()` для создания символических ссылок вместо копирования файлов.
* Добавлена встроенная поддержка исходных карт - плагин gulp-sourcemaps больше не нужен!
* Теперь рекомендуется регистрация задачи для экспортируемых функций - с использованием экспорта узла или ES.
* Были разработаны специальные реестры, позволяющие выполнять общие задачи или расширять функциональность.
* Реализации потоков были улучшены, что позволило улучшить условную и поэтапную сборку.

## gulp для энтерпрайза

Доступно как часть подписки Tidelift

Сопровождающие gulp и тысячи других пакетов работают с Tidelift, чтобы обеспечить коммерческую поддержку и обслуживание зависимостей с открытым исходным кодом, которые вы используете для создания своих приложений. Экономьте время, снижайте риски и улучшайте работоспособность кода, оплачивая при этом те, кто поддерживает именно те зависимости, которые вы используете. [Подробнее.](https://tidelift.com/subscription/pkg/npm-gulp?utm_source=npm-gulp&utm_medium=referral&utm_campaign=enterprise&utm_term=repo)

## Установка

Следуйте нашему [Краткому руководству][quick-start].

## Дорожная карта

Узнайте обо всех наших незавершенных работах и нерешенных проблемах на https://github.com/orgs/gulpjs/projects.

## Документация

Ознакомьтесь с [Руководством по началу работы][getting-started-guide] и [API документацией][api-docs] на нашем веб-сайте!

__Простите нашу пыль! Все остальные документы будут отложены, пока мы не обновим все. Пожалуйста, откройте вопрос, если что-то не работает.__

## Пример `gulpfile.js`

Этот файл даст вам представление о том, что делает gulp.

```js
var gulp = require('gulp');
var less = require('gulp-less');
var babel = require('gulp-babel');
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var rename = require('gulp-rename');
var cleanCSS = require('gulp-clean-css');
var del = require('del');

var paths = {
  styles: {
    src: 'src/styles/**/*.less',
    dest: 'assets/styles/'
  },
  scripts: {
    src: 'src/scripts/**/*.js',
    dest: 'assets/scripts/'
  }
};

/* Не все задачи должны использовать потоки,
 * файл gulpfile - это просто еще одна программа узла,
 * и вы можете использовать все пакеты, доступные в npm,
 * но он должен возвращать либо Promise, либо Stream,
 * либо принимать обратный вызов и вызывать его.
 */
function clean() {
  // Вы можете использовать несколько шаблонов подстановки, как и с `gulp.src`,
  // например, если вы используете del 2.0 или выше, верните его промис
  return del([ 'assets' ]);
}

/*
 * Определите наши задачи, используя простые функции
 */
function styles() {
  return gulp.src(paths.styles.src)
    .pipe(less())
    .pipe(cleanCSS())
    // передать параметры в поток
    .pipe(rename({
      basename: 'main',
      suffix: '.min'
    }))
    .pipe(gulp.dest(paths.styles.dest));
}

function scripts() {
  return gulp.src(paths.scripts.src, { sourcemaps: true })
    .pipe(babel())
    .pipe(uglify())
    .pipe(concat('main.min.js'))
    .pipe(gulp.dest(paths.scripts.dest));
}

function watch() {
  gulp.watch(paths.scripts.src, scripts);
  gulp.watch(paths.styles.src, styles);
}

/*
 * Укажите, будут ли задачи выполняться последовательно или параллельно, используя `gulp.series` и `gulp.parallel`
 */
var build = gulp.series(clean, gulp.parallel(styles, scripts));

/*
 * Вы можете использовать нотацию модуля CommonJS `exports` для объявления задач
 */
exports.clean = clean;
exports.styles = styles;
exports.scripts = scripts;
exports.watch = watch;
exports.build = build;
/*
 * Определите задачу по умолчанию, которую можно вызвать, просто запустив `gulp` из cli
 */
exports.default = build;
```

## Используйте последнюю версию JavaScript в вашем gulpfile

__Большинство новых версий node поддерживают большинство функций, которые предоставляет Babel, за исключением синтаксиса `import`/`export`. Если требуется только этот синтаксис, переименуйте его в `gulpfile.esm.js`, установите модуль [esm][esm-module], и пропустите часть Babel ниже.__

Node уже поддерживает множество функций __ES2015+__, но, чтобы избежать проблем с совместимостью, мы предлагаем установить Babel и переименовать ваш `gulpfile.js` в `gulpfile.babel.js`.

```sh
npm install --save-dev @babel/register @babel/core @babel/preset-env
```

Затем создайте файл **.babelrc** с предустановленной конфигурацией.

```js
{
  "presets": [ "@babel/preset-env" ]
}
```

А вот тот же образец сверху, написанный в **ES2015+**.

```js
import gulp from 'gulp';
import less from 'gulp-less';
import babel from 'gulp-babel';
import concat from 'gulp-concat';
import uglify from 'gulp-uglify';
import rename from 'gulp-rename';
import cleanCSS from 'gulp-clean-css';
import del from 'del';

const paths = {
  styles: {
    src: 'src/styles/**/*.less',
    dest: 'assets/styles/'
  },
  scripts: {
    src: 'src/scripts/**/*.js',
    dest: 'assets/scripts/'
  }
};

/*
 * Для небольших задач вы можете экспортировать стрелочные функции
 */
export const clean = () => del([ 'assets' ]);

/*
 * Вы также можете объявить именованные функции и экспортировать их как задачи
 */
export function styles() {
  return gulp.src(paths.styles.src)
    .pipe(less())
    .pipe(cleanCSS())
    // передать параметры в поток
    .pipe(rename({
      basename: 'main',
      suffix: '.min'
    }))
    .pipe(gulp.dest(paths.styles.dest));
}

export function scripts() {
  return gulp.src(paths.scripts.src, { sourcemaps: true })
    .pipe(babel())
    .pipe(uglify())
    .pipe(concat('main.min.js'))
    .pipe(gulp.dest(paths.scripts.dest));
}

 /*
  * Вы даже можете использовать `export as` для переименования экспортируемых задач
  */
function watchFiles() {
  gulp.watch(paths.scripts.src, scripts);
  gulp.watch(paths.styles.src, styles);
}
export { watchFiles as watch };

const build = gulp.series(clean, gulp.parallel(styles, scripts));
/*
 * Экспорт задачи по умолчанию
 */
export default build;
```

## Дополнительные сборки

Вы можете отфильтровать неизмененные файлы между запусками задачи, используя параметр `since` функции `gulp.src` и `gulp.lastRun`:

```js
const paths = {
  ...
  images: {
    src: 'src/images/**/*.{jpg,jpeg,png}',
    dest: 'build/img/'
  }
}

function images() {
  return gulp.src(paths.images.src, {since: gulp.lastRun(images)})
    .pipe(imagemin())
    .pipe(gulp.dest(paths.images.dest));
}

function watch() {
  gulp.watch(paths.images.src, images);
}
```

Время выполнения задачи сохраняется в памяти и теряется при выходе из gulp.
Это только сэкономит время во время выполнения задачи `watch` при повторном запуске задачи `images` .

## Хотите внести свой вклад?

Кто угодно может помочь сделать этот проект лучше - ознакомьтесь с нашим [Руководством по участию](/CONTRIBUTING.md)!

## Сторонники

Поддержите нас ежемесячным пожертвованием и помогите нам продолжить нашу деятельность.

[![Backers][backers-image]][support-url]

## Спонсоры

Станьте спонсором, чтобы разместить свой логотип в нашем README на Github.

[![Sponsors][sponsors-image]][support-url]

[downloads-image]: https://img.shields.io/npm/dm/gulp.svg
[npm-url]: https://www.npmjs.com/package/gulp
[npm-image]: https://img.shields.io/npm/v/gulp.svg

[azure-pipelines-url]: https://dev.azure.com/gulpjs/gulp/_build/latest?definitionId=1&branchName=master
[azure-pipelines-image]: https://dev.azure.com/gulpjs/gulp/_apis/build/status/gulp?branchName=master

[travis-url]: https://travis-ci.org/gulpjs/gulp
[travis-image]: https://img.shields.io/travis/gulpjs/gulp.svg?label=travis-ci

[appveyor-url]: https://ci.appveyor.com/project/gulpjs/gulp
[appveyor-image]: https://img.shields.io/appveyor/ci/gulpjs/gulp.svg?label=appveyor

[coveralls-url]: https://coveralls.io/r/gulpjs/gulp
[coveralls-image]: https://img.shields.io/coveralls/gulpjs/gulp/master.svg

[gitter-url]: https://gitter.im/gulpjs/gulp
[gitter-image]: https://badges.gitter.im/gulpjs/gulp.svg

[backer-url]: #backers
[backer-badge]: https://opencollective.com/gulpjs/backers/badge.svg?color=blue
[sponsor-url]: #sponsors
[sponsor-badge]: https://opencollective.com/gulpjs/sponsors/badge.svg?color=blue

[support-url]: https://opencollective.com/gulpjs#support

[backers-image]: https://opencollective.com/gulpjs/backers.svg
[sponsors-image]: https://opencollective.com/gulpjs/sponsors.svg

[quick-start]: https://gulpjs.com/docs/en/getting-started/quick-start
[getting-started-guide]: https://gulpjs.com/docs/en/getting-started/quick-start
[api-docs]: https://gulpjs.com/docs/en/api/concepts
[esm-module]: https://github.com/standard-things/esm

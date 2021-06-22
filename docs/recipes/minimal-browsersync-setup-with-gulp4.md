# Минимальная настройка BrowserSync с Gulp 4

[BrowserSync](https://www.browsersync.io/) - отличный инструмент для оптимизации процесса разработки с возможностью мгновенного отражения изменений кода в браузере посредством перезагрузки в реальном времени.
Настройка сервера BrowserSync с перезагрузкой в реальном времени с помощью Gulp 4 очень проста и понятна.

## Шаг 1. Установите зависимости

```
npm install --save-dev browser-sync
```

## Шаг 2: Настройте структуру проекта

```
src/
  scripts/
    |__ index.js
dist/
  scripts/
index.html
gulpfile.babel.js
```

Цель здесь - уметь:

- Создайте файл исходного сценария в `src/scripts/`, например. компиляция с помощью babel, минификация и т. д.
- Поместите скомпилированную версию в `dist/scripts` для использования в `index.html`
- Следите за изменениями в исходном файле и пересоберите пакет `dist`
- При каждой перестройке пакета `dist`, перезагружайте браузер, чтобы сразу отразить изменения

## Шаг 3: напишите gulpfile

Gulpfile можно разбить на 3 части.

### 1. Напишите задачу подготовить пакет dist как обычно

Обратитесь к основному [README](https://github.com/gulpjs/gulp/blob/master/docs/README.md) для получения дополнительной информации.

```javascript
import babel from 'gulp-babel';
import concat from 'gulp-concat';
import del from 'del';
import gulp from 'gulp';
import uglify from 'gulp-uglify';

const paths = {
  scripts: {
    src: 'src/scripts/*.js',
    dest: 'dist/scripts/'
  }
};

const clean = () => del(['dist']);

function scripts() {
  return gulp.src(paths.scripts.src, { sourcemaps: true })
    .pipe(babel())
    .pipe(uglify())
    .pipe(concat('index.min.js'))
    .pipe(gulp.dest(paths.scripts.dest));
}
```

### 2. Настройте сервер BrowserSync

И напишите задачи для обслуживания и перезагрузите сервер соответственно.

```javascript
import browserSync from 'browser-sync';
const server = browserSync.create();

function reload(done) {
  server.reload();
  done();
}

function serve(done) {
  server.init({
    server: {
      baseDir: './'
    }
  });
  done();
}
```

### 3. Следите за изменением источника, перестройте скрипты и перезагрузите сервер

Это тривиально выполняется с помощью `gulp.series`

```javascript
const watch = () => gulp.watch(paths.scripts.src, gulp.series(scripts, reload));
```

## Шаг 4. Соберите все вместе

Последний шаг - выставить задачу по умолчанию

```javascript
const dev = gulp.series(clean, scripts, serve, watch);
export default dev;
```

И профит

```bash
$ gulp
```

Теперь, если вы перейдете на [http://localhost:3000](http://localhost:3000), который является адресом по умолчанию для сервера BrowserSync, вы увидите, что конечный результат в браузере обновляется каждый раз, когда вы меняете содержимое исходного файла.
Вот весь gulpfile:

```javascript
import babel from 'gulp-babel';
import concat from 'gulp-concat';
import del from 'del';
import gulp from 'gulp';
import uglify from 'gulp-uglify';
import browserSync from 'browser-sync';

const server = browserSync.create();

const paths = {
  scripts: {
    src: 'src/scripts/*.js',
    dest: 'dist/scripts/'
  }
};

const clean = () => del(['dist']);

function scripts() {
  return gulp.src(paths.scripts.src, { sourcemaps: true })
    .pipe(babel())
    .pipe(uglify())
    .pipe(concat('index.min.js'))
    .pipe(gulp.dest(paths.scripts.dest));
}

function reload(done) {
  server.reload();
  done();
}

function serve(done) {
  server.init({
    server: {
      baseDir: './'
    }
  });
  done();
}

const watch = () => gulp.watch(paths.scripts.src, gulp.series(scripts, reload));

const dev = gulp.series(clean, scripts, serve, watch);
export default dev;
```

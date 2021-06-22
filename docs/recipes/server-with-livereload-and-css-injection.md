# Сервер с живой перезагрузкой и CSS-инъекцией

С помощью [BrowserSync](https://browsersync.io) и gulp вы можете легко создать сервер разработки, доступный для любого устройства в той же сети Wi-Fi. BrowserSync также имеет встроенную перезагрузку в реальном времени, поэтому настраивать больше нечего.

Сначала установите модули:

```sh
$ npm install --save-dev gulp browser-sync
```

Затем, учитывая следующую файловую структуру...

```
gulpfile.js
app/
  styles/
    main.css
  scripts/
    main.js
  index.html
```

...вы можете легко обслуживать файлы из каталога `app` и перезагружать все браузеры при изменении любого из них, используя следующую команду в `gulpfile.js`:

```js
var gulp = require('gulp');
var browserSync = require('browser-sync');
var reload = browserSync.reload;

// смотреть файлы на предмет изменений и перезагружать
gulp.task('serve', function() {
  browserSync({
    server: {
      baseDir: 'app'
    }
  });

  gulp.watch(['*.html', 'styles/**/*.css', 'scripts/**/*.js'], {cwd: 'app'}, reload);
});

```

и включение CSS в `index.html`:

```html
<html>
  <head>
    ...
    <link rel="stylesheet" href="styles/main.css">
    ...

```

для обслуживания ваших файлов и запуска окна браузера, указывающего на URL-адрес по умолчанию (http://localhost:3000), выполните:

```bash
gulp serve
```

## + Препроцессоры CSS

Распространенный вариант использования - перезагрузка файлов CSS после их предварительной обработки. Используя Sass в качестве примера, вы можете указать браузерам перезагрузить CSS, не выполняя обновление всей страницы.

Учитывая эту обновленную файловую структуру...

```
gulpfile.js
app/
  scss/
    main.scss
  scripts/
    main.js
  index.html
```

...вы можете легко просматривать файлы Sass из каталога `scss` и перезагружать все браузеры при изменении любого из них, используя следующую команду в `gulpfile.js`:

```js
var gulp = require('gulp');
var sass = require('gulp-ruby-sass');
var browserSync = require('browser-sync');
var reload = browserSync.reload;

gulp.task('sass', function() {
  return sass('scss/styles.scss')
    .pipe(gulp.dest('app/css'))
    .pipe(reload({ stream:true }));
});

// наблюдать за файлами Sass на предмет изменений, запускать препроцессор Sass с задачей 'sass' и перезагружать
gulp.task('serve', gulp.series('sass', function() {
  browserSync({
    server: {
      baseDir: 'app'
    }
  });

  gulp.watch('scss/*.scss', gulp.series('sass'));
}));
```

и включение предварительно обработанного CSS в `index.html`:

```html
<html>
  <head>
    ...
    <link rel="stylesheet" href="css/main.css">
    ...

```

для обслуживания ваших файлов и запуска окна браузера, указывающего на URL-адрес по умолчанию (http://localhost:3000), выполните:

```bash
gulp serve
```

## Дополнительно

- Живая перезагрузка, внедрение CSS и синхронизация прокрутки/формы без проблем работают внутри виртуальных машин [BrowserStack](https://www.browserstack.com/).
- Установите `tunnel: true` для просмотра вашего локального сайта по общедоступному URL-адресу (со всеми функциями BrowserSync).

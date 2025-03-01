# FAQ

## Почему gulp? Почему нет ____?

Смотрите [Слайд-шоу введения gulp][gulp introduction slideshow] для краткого описания того, как появился gulp.

## Это "gulp" bkb "Gulp"?

gulp всегда в нижнем регистре. Единственное исключение - логотип gulp, где gulp пишется с заглавной буквы.

## Где я могу найти список плагинов gulp?

Плагины gulp всегда включают ключевое слово `gulpplugin`. [Искать gulp плагины][search-gulp-plugins] или [просмотреть все плагины][npm plugin search].

## Я хочу написать плагин gulp, как мне начать?

Смотрите вики-страницу [Написание подключаемого модуля gulp][Writing a gulp plugin] для получения рекомендаций и примеров для начала.

## Мой плагин делает ____, он слишком много делает?

Наверное. Спроси себя:

1. Делает ли мой плагин то, что может понадобиться другим плагинам?
    * Если это так, эта функциональность должна быть отдельным плагином. [Проверить, существует ли он уже в npm][npm plugin search].
1. Делает ли мой плагин две совершенно разные вещи в зависимости от варианта конфигурации?
    * Если да, то для сообщества может быть лучше выпустить его как два отдельных плагина.
    * Если две задачи разные, но очень тесно связаны, вероятно, все в порядке.

## Как должны быть представлены новые строки в выводе плагина?

Всегда используйте `\n` для предотвращения проблем с различиями между операционными системами.

## Я установил gulp как зависимость от файла package.json, запустив `npm install`, но я продолжаю получать `command not found` всякий раз, когда пытаюсь запустить команду gulp, почему это не работает?

После установки gulp в качестве зависимости проекта вам необходимо добавить это в переменную среды PATH, чтобы при запуске команды система могла ее найти. Простое решение - установить gulp глобально, чтобы его двоичные файлы попадали в переменную среды PATH. Чтобы установить gulp глобально, используйте команду `npm install gulp-cli -g`

## Где я могу получить обновления по gulp?

Обновления gulp можно найти в следующих твиттерах:

* [@wearefractal](https://twitter.com/wearefractal)
* [@eschoff](https://twitter.com/eschoff)
* [@gulpjs](https://twitter.com/gulpjs)

## Есть ли у gulp канал чата?

Да, поговорите с нами на [Gitter](https://gitter.im/gulpjs/gulp).

[Writing a gulp plugin]: writing-a-plugin/README.md
[gulp introduction slideshow]: https://slid.es/contra/gulp
[Freenode]: https://freenode.net/
[search-gulp-plugins]: https://gulpjs.com/plugins/
[npm plugin search]: https://npmjs.org/browse/keyword/gulpplugin

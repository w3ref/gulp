<!-- front-matter
id: quick-start
title: Быстрый старт
hide_title: true
sidebar_label: Быстрый старт
-->

# Быстрый старт

Если вы ранее устанавливали gulp глобально, запустите `npm rm --global gulp` перед тем, как следовать этим инструкциям. Для получения дополнительной информации прочтите это [Sip][sip-article].

## Проверьте node, npm и npx

```sh
node --version
```

![Output: v8.11.1][img-node-version-command]

```sh
npm --version
```

![Output: 5.6.0][img-npm-version-command]

```sh
npx --version
```

![Output: 9.7.1][img-npx-version-command]

Если они не установлены, следуйте инструкциям [здесь][node-install].

## Установите утилиту командной строки gulp

```sh
npm install --global gulp-cli
```

## Создайте каталог проекта и перейдите в него

```sh
npx mkdirp my-project
```

```sh
cd my-project
```

## Создайте файл package.json в каталоге вашего проекта

```sh
npm init
```

Это поможет вам дать вашему проекту имя, версию, описание и т. д.

## Установите пакет gulp в свои devDependencies

```sh
npm install --save-dev gulp
```

## Проверьте свои версии gulp

```sh
gulp --version
```

Убедитесь, что результат соответствует снимку экрана ниже, в противном случае вам может потребоваться перезапустить шаги, описанные в этом руководстве.

![Output: CLI version 2.0.1 & Local version 4.0.0][img-gulp-version-command]

## Создайте a gulpfile

Используя текстовый редактор, создайте файл с именем gulpfile.js в корне вашего проекта со следующим содержимым:

```js
function defaultTask(cb) {
  // поместите здесь код для вашей задачи по умолчанию
  cb();
}

exports.default = defaultTask
```

## Протестируйте это

Запустите команду gulp в каталоге вашего проекта:

```sh
gulp
```

Чтобы запустить несколько задач, вы можете использовать `gulp <task> <othertask>`.

## Результат

Задача по умолчанию будет запущена и ничего не сделает.
![Output: Starting default & Finished default][img-gulp-command]

[sip-article]: https://medium.com/gulpjs/gulp-sips-command-line-interface-e53411d4467
[node-install]: https://nodejs.org/en/
[img-node-version-command]: https://gulpjs.com/img/docs-node-version-command.png
[img-npm-version-command]: https://gulpjs.com/img/docs-npm-version-command.png
[img-npx-version-command]: https://gulpjs.com/img/docs-npx-version-command.png
[img-gulp-version-command]: https://gulpjs.com/img/docs-gulp-version-command.png
[img-gulp-command]: https://gulpjs.com/img/docs-gulp-command.png

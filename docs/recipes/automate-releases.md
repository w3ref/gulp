<!-- front-matter
id: automate-releases
title: Автоматизация релизов
hide_title: true
sidebar_label: Автоматизация релизов
-->

# Автоматизация релизов

Если ваш проект следует семантическому управлению версиями, может быть хорошей идеей автоматизировать шаги, необходимые для выпуска.
Приведенный ниже рецепт увеличивает версию проекта, фиксирует изменения в git и создает новый GitHub-релиз.

Для публикации выпуска GitHub вам необходимо [создать личный токен доступа](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) и добавьте его в свой проект. Однако мы не хотим фиксировать его, поэтому мы будем использовать [`dotenv`](https://www.npmjs.com/package/dotenv), чтобы загрузить его из игнорируемого git файла `.env`:

```
GH_TOKEN=ff34885...
```

Не забудьте добавить `.env` в ваш `.gitignore`.

Далее устанавливаем все необходимые зависимости для этого рецепта:

```sh
npm install --save-dev conventional-recommended-bump conventional-changelog-cli conventional-github-releaser dotenv execa
```

В зависимости от вашей среды, настроек и предпочтений рабочий процесс выпуска может выглядеть примерно так:

``` js
const gulp = require('gulp');
const conventionalRecommendedBump = require('conventional-recommended-bump');
const conventionalGithubReleaser = require('conventional-github-releaser');
const execa = require('execa');
const fs = require('fs');
const { promisify } = require('util');
const dotenv = require('dotenv');

// загрузить переменные среды
const result = dotenv.config();

if (result.error) {
  throw result.error;
}

// Стандартная предустановка журнала изменений
const preset = 'angular';
// вывод команд в терминал
const stdio = 'inherit';

async function bumpVersion() {
  // получить рекомендованную версию на основе коммитов
  const { releaseType } = await promisify(conventionalRecommendedBump)({ preset });
  // версия bump без коммитов и тегов
  await execa('npm', ['version', releaseType, '--no-git-tag-version'], {
    stdio,
  });
}

async function changelog() {
  await execa(
    'npx',
    [
      'conventional-changelog',
      '--preset',
      preset,
      '--infile',
      'CHANGELOG.md',
      '--same-file',
    ],
    { stdio }
  );
}

async function commitTagPush() {
  // даже несмотря на то, что в этом случае мы могли бы обойтись без "require",
  // мы выбираем безопасный путь, потому что "require" кэширует значение,
  // поэтому, если мы снова используем "require" где-то еще, мы не получим текущее значение,
  // но значение в последний раз, которое мы назвали "require"
  const { version } = JSON.parse(await promisify(fs.readFile)('package.json'));
  const commitMsg = `chore: release ${version}`;
  await execa('git', ['add', '.'], { stdio });
  await execa('git', ['commit', '--message', commitMsg], { stdio });
  await execa('git', ['tag', `v${version}`], { stdio });
  await execa('git', ['push', '--follow-tags'], { stdio });
}

function githubRelease(done) {
  conventionalGithubReleaser(
    { type: 'oauth', token: process.env.GH_TOKEN },
    { preset },
    done
  );
}

exports.release = gulp.series(
  bumpVersion,
  changelog,
  commitTagPush,
  githubRelease
);
```

---
title: "File nesting"
date: 2024-02-08T21:44:24+03:00
draft: false
tags: [ "IntelliJ IDEA", "VS Code", ]
author: "Artur Rafikov"
showToc: true
TocOpen: true
UseHugoToc: true
cover:
  image: "/articles/file_nesting/cover.png" 
  alt: "File nesting article cover"
  relative: true
---

Порой, в проекте накапливается много файлов, имеющих одно имя, но разное расширение. И они очень мешают навигации по
файловой структуре. Например, во Flutter проектах часто используют
связку [freezed](https://pub.dev/packages/freezed) + [json_serializable](https://pub.dev/packages/json_serializable),
которая генерирует дополнительно два файла к основному, и получается три файла. А если в папке несколько таких выхлопов,
то вообще ужас:

```
.
└── data
    └── models
        └── user
            ├── password_change_dto.dart
            ├── password_change_dto.freezed.dart
            ├── password_change_dto.g.dart
            ├── password_reset_dto.dart
            ├── password_reset_dto.frezed.dart
            ├── password_reset_dto.g.dart
```

Чтобы не напрягаться в попытках найти нужный файл, можно настроить File nesting — превращение файла в раскладываемый
файл.

Результат будет выглядеть так:

![Untitled](/articles/file_nesting/images/Untitled.png)

## Настройка в IntelliJ IDEA

Для начала, необходимо открыть любой проект. Нажмите дважды на Shift и напишите “File nesting”.

![Untitled image](/articles/file_nesting/images/Untitled%201.png)

Первый столбик — главный файл, в который будут складываться другие файлы, совпадающие по имени и расширению из второго
столбика.

По этой логике, нас интересует колонка “.dart”. Необходимо во вторую колонку напротив прописать в конце, соблюдая
разделители: “.freezed.dart; .g.dart”

![Untitled](/articles/file_nesting/images/Untitled%202.png)

После сохранения изменений, файлы будут аккуратно сгруппированы:

![Untitled](/articles/file_nesting/images/Untitled%203.png)

Точно также можно настроить, например для pubspec.yaml, pubspec.lock и остальных файлов конфигурации.

![Untitled](/articles/file_nesting/images/Untitled%204.png)

## Настройка в VS Code

Открыть настройки (settings.json) и добавить в конец следующие строчки:

```json
{
  "explorer.fileNesting.enabled": true,
  "explorer.fileNesting.expand": false,
  "explorer.fileNesting.patterns": {
    "pubspec.yaml": ".flutter-plugins, .packages, .dart_tool, .flutter-plugins-dependencies, .metadata, .packages, pubspec.lock, build.yaml, analysis_options.yaml, all_lint_rules.yaml, flutter_*.yaml, icons_launcher.yaml",
    ".gitignore": ".gitattributes, .gitmodules, .gitmessage, .mailmap, .git-blame*",
    "readme.*": "authors, backers.md, changelog*, citation*, code_of_conduct.md, codeowners, contributing.md, contributors, copying, credits, governance.md, history.md, license*, maintainers, readme*, security.md, sponsors.md",
    "*.dart": "$(capture).g.dart, $(capture).freezed.dart, $(capture).config.dart",
    "*.graphql": "$(capture).gql.dart, $(capture).*.gql.dart",
    "schema.graphql": "schema.schema.gql.dart",
    "firebase.json": ".firebaserc"
  }
}
```
(код настроек взят у [plugfox](https://github.com/plugfox/))

Ключ это паттерн главного файла, а значение — файлы, которые будут дочерними для него.

Результат выглядит вот так:

![Untitled](/articles/file_nesting/images/Untitled%205.png)

Не очень красиво, файлы располагаются друг за другом сплошняком — проблема, от которой мы хотели избавиться. Чтобы эту
ситуацию улучшить — можно установить
тему [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme). Результат:

![Untitled](/articles/file_nesting/images/Untitled%206.png)
---
title: "Dart 3.3 & Flutter 3.19"
date: 2024-02-16T23:27:24+03:00
draft: false
tags: [ "Flutter", "Dart", ]
author: "Artur Rafikov"
showToc: true
TocOpen: true
UseHugoToc: true
cover:
  image: "/articles/flutter_3_19/cover.png" 
  alt: "Flutter 3.3 update article cover"
  relative: true
---


# Новое в Dart

В основном, вектор обновления направлен на развитие в сторону взаимодействия с JS и компиляцией в WebAssembly. Представили новый пакет для взаимодействия с Web API — [package:web](https://pub.dev/packages/web). Является новым способом для интеграции с js для Dart проектов. Миграция на этот пакет привнесет очень вкусный бонус — компиляция в WebAssembly. А секретом нового пакета является новая фича языка — extension type.

## Extension type

Обёртка над указанным типом, которая имеет доступ к его методам и полям. Позволяет добавлять свои методы (переопределить методы можно тоже). Является compile-time объектом, который не имеет затраты на выделение типа, как написание обычного враппера.

Выглядит примерно так:

```dart
/// Название extension - новый тип. В скобках указывается тип, который мы хотим расширить.
extension type NewTypeName(int typeThatWeWantToExtend) {

  // Мы можем добавлять поля и методы, которые будут доступны через новый тип.
  bool isPrime() {
    if (typeThatWeWantToExtend < 2) {
      return false;
    }
    for (int i = 2; i < typeThatWeWantToExtend; i++) {
      if (typeThatWeWantToExtend % i == 0) {
        return false;
      }
    }
    return true;
  }
}
```

Пример инициализации:

```dart
void main() {
  final newInt = NewTypeName(5); // используем наш новый тип, в который передаем тип, который мы расширяем
  print(newInt.isPrime()); // true; вызов метода из нашего нового типа
  print(newInt.typeThatWeWantToExtend); // обращение к типу, над котором делали обёртку
}
```

Повторять документацию я не вижу смысла, поэтому вот краткий список фич, которые умеют extension type:

1. [Конструкторы](https://dart.dev/language/extension-types#constructors)
2. [Поддерживаемые поля](https://dart.dev/language/extension-types#members) (функции, геттеры, сеттеры, операторы)
3. [Имплементация типа](https://dart.dev/language/extension-types#implements)
4. [Переопределение поля](https://dart.dev/language/extension-types#redeclare)

Но остановлюсь более подробно на 3 и 4 пункте.

Имплементация типа позволяет сделать так, что наш тип будет равен тому же типу, над которым мы пишем обёртку. Это удобно, например, для создания объекта Money:

```dart
extension type Money(int amount) implements int {
  /// Сколько центов в одном экземпляре класса Money.
  ///
  /// Например, в 1 долларе или евро 100 центов.
  static const _base = 100;

  /// Создает экземпляр класса Money из текстового значения.
  /// 
  /// Допустимые форматы:
  /// - '100' - 100 долларов
  /// - '100.50' - 100 долларов и 50 центов
  /// - '100.' - 100 долларов
  /// - '.50' - 0 долларов и 50 центов
  factory Money.fromString(String amount) {
    final moneyValues = amount.split('.').map((e) => e.isEmpty ? '0' : e).toList();
    assert(moneyValues.length <= 2, 'Invalid money format');

    final (int dollars, int cents) = switch (moneyValues) {
      [String dollarsString] => (int.parse(dollarsString), 0),
      [String dollarsString, String centsString] => (
          int.parse(dollarsString),
          int.parse(centsString.padRight(2, '0')),
        ),
      [] => (0, 0),
      _ => throw FormatException('Invalid money format'),
    };

    return Money(dollars * _base + cents);
  }

  /// Возвращает отформатированное значение в формате долларов и центов.
  String get formatted {
    final dollars = amount ~/ 100;
    final cents = amount % 100;

    return '\$$dollars.${cents.toString().padLeft(2, '0')}';
  }
}
```

Как вы заметили, здесь нет математических операторов. Вся логика для этого класса будет равна логике при работе с обычным числом, поэтому её можно опустить:

```dart
void main() {
  final moneyFromInt = Money(1000); // создаем Money из 1000 центов (10 долларов)
  final moneyFromString = Money.fromString('10'); // создаем Money из 10 долларов
  print(moneyFromInt.formatted); // $10.00
  print(moneyFromString.formatted); // $10.00

  print(moneyFromInt == moneyFromString); // true
  print(moneyFromInt == 1000); // true
  print(moneyFromInt + moneyFromString); // 2000 центов
  print(moneyFromInt - moneyFromString); // 0 центов
  print(moneyFromInt ~/ 3); // 333 цента
  print(Money(moneyFromInt ~/ 3).formatted); // $3.33
}
```

А теперь перейдем к грязи — `@redeclare` аннотация ([package:meta](https://pub.dev/packages/meta)). В extension type нельзя переопределить (`@override`) какое-то поле, поддерживая связь с родителем через `super.method`. Нам дали новую аннотацию, с помощью которой поведение функции меняется на новую:

```dart
void main() {
  final someString = 'Hello World!';
  final myString = MyString(someString);

  print(someString[0]); // H
  print(myString[0]); // 72
  print(isSame(someString, myString)); // true
}

// метод, который принимает два String
bool isSame(String value1, String value2) {
  print(value1[0]); // H
  print(value2[0]); // H
  return value1 == value2;
}

extension type MyString(String _) implements String {
  // Заменяет 'String.operator[]'
  @redeclare
  int operator [](int index) => codeUnitAt(index);
}
```

То есть, мы можем переопределить для себя какие-то поля, и они будут отрабатывать согласно нашему новому типу. Но если какой-то метод будет запрашивать тип, над которым мы делали обёртку — то использовать он будет напрямую тип, а не обёртку. Это защищает от кейсов, когда автор библиотеки сделал `final class` , и ожидает одно поведение от метода класса, а мы написали обёртку, в которой метод будет возвращать совершенно другое значение. Dart умный, и всё взаимодействие у автора библиотеки будет происходить только с его классом.

Но если вы у себя в проекте сделаете что-то подобное, то главное не утоните в слоях пост-иронии и самообмана.

```dart
extension type int(String _) implements String {
  // ...
}
```

В заключение этой фичи, хочу сказать, что она предназначена для достаточно узких кейсов, либо когда функционал нового типа во многом повторяет какой-либо существующий. Но в то же время, это очень полезный инструмент для работы с JS. Возможно, в будущем комьюнити найдет более интересные способы применения этой конструкции.

### Полезные ссылки:

1. [js_interop](https://dart.dev/interop/js-interop)
2. [Gemini in Dart & Flutter apps](https://medium.com/flutter/harness-the-gemini-api-in-your-dart-and-flutter-apps-00573e560381)

# Новое во Flutter

## Улучшения

- [Несжатый Flutter Engine теперь весит на 350KB меньше](https://flutter-flutter-perf.skia.org/e/?begin=1708023887&end=1708110287&queries=test%3Dhello_world_ios__compile&requestType=0&selected=commit%3D37892%26name%3D%252Carch%253Darm%252Cbranch%253Dmaster%252Cconfig%253Ddefault%252Cdevice_type%253DiPhone_11%252Cdevice_version%253Dnone%252Chost_type%253Dmac%252Csub_result%253Dflutter_framework_uncompressed_bytes%252Ctest%253Dhello_world_ios__compile%252C).
- [Оптимизация блюра до 70%](https://github.com/flutter/engine/pull/47808), [в зависимости от сложности UI](https://github.com/flutter/engine/pull/47397)
- [Google Maps](https://github.com/flutter/packages/pull/5408) и лупа в TextField теперь работают в TLHC режиме, обеспечивая лучшую производительность

## Android

- В DevTools добавлена секция “Deep Link”, которая поможет проверить правильность настройки deeplinks. От себя добавлю, что в Android Studio тоже можно проверять запуск ссылок с помощью инструмента “App Links Assistant”.
- Контекстное меню, которое появляется после выбора текста, теперь поддерживает другие установленные приложения в системе. Например, при выделении номера, будет предложено открыть его в приложении “Телефон”.  [Также добавлена кнопка “Поделиться”](https://github.com/flutter/flutter/issues/107578), которая в Android-native отображается по умолчанию.
- Добавлена поддержка Android OpenGL в Impeller (до этого был только Vulkan). Это значит, что почти все устройства на Android, теперь поддерживаются Impeller. Чтобы потыкать новый движок, [используйте эту статью](https://docs.flutter.dev/perf/impeller#android).

## iOS

- Системный шрифт теперь следует Apple Human Design: чем меньше размер шрифта, тем больше глифы удалены друг от друга; и наоборот — чем больше шрифт, тем ближе глифы находятся друг к другу.
- Flutter теперь содержит Privacy Manifest, согласно [новым требованиям Apple](https://developer.apple.com/support/third-party-SDK-requirements/).
- Прекращена поддержка iOS 11

## Windows

- Прекращена поддержка Windows 7, Windows 8
- Добавлена поддержка Windows arm64

## Важное

- Флаг `Paint.enableDithering` удалён. Он был включен глобально уже много версий назад, а также проблема, которую он решал, была устранена.

# Roadmap

Разработчики поделились с нами планами на 2024 год, и там очень много вкусного:

1. Skia на iOS будет полностью убрана. На Android Impeller станет основным, но Skia можно будет включить по необходимости.
2. Фреймворк перейдет полностью на Material 3, Material 2 со временем будет удалён. Я настоятельно рекомендую переводить ваши проекты на Material 3 и правильно настраивать тему. Используя Material 2, вы ограничиваете себя от новых виджетов, например [IconButton.filled](https://api.flutter.dev/flutter/material/IconButton/IconButton.filled.html).
3. Команда также ожидает вернуться к работе над [Blackcanvas](https://flutter.dev/go/blankcanvas) — проект, который позволит разработчикам создавать свои отдельные UI наборы с нуля. Проблема в том, что все существующие UI пакеты наследуются от Material, и редко от Cupertino. Разработчики хотят внести улучшения, чтобы подобные пакеты не зависели от уже существующих UI наборов.
4. В Web продолжат пилить оптимизацию, хотят сделать CanvasKit рендером по умолчанию. Но самое интересное — поддержка platform view и компиляция в WebAssembly. Также, возобновляется работа над [Hot Reload в вебе](https://github.com/flutter/flutter/issues/53041) (сейчас приложение перезапускается при вызове Hot Reload).
5. Поддержка platform views на macOS и Windows, которая позволит встраивать, например, WebView. На Linux приоритет добавить GTK4. И на всех платформах ведется работа по мультиоконному режиму в рамках одного Dart изолята и одного дерева виджетов. Первые превью этой фичи уже были на канале Flutter!
6. В этом году в Dart ожидают решить, насколько поддержка метапрограммирования целесообразна, и либо выпустить первые этапы поддержки этой фичи, либо же отказаться вовсе. [Метапрограммирование (или же макросы)](https://github.com/dart-lang/language/blob/main/working/macros/feature-specification.md) — это что-то типа генератора кода внутри самого кода. В основном, комьюнити ждет эту фичу, чтобы писать сериализацию/десериализацию дата классов. Для такого случая рассматривают возможность добавления [ValueClasses](https://github.com/dart-lang/language/blob/main/working/value-classes/feature-specification.md).
7. Обсуждают интересные фичи для сокращения написания кода, например: [структы](https://github.com/dart-lang/language/issues/2364) или [сокращенные импорты](https://github.com/dart-lang/language/issues/649)
8. В этом году будет 4 стабильных релиза и 12 беток.
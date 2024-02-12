---
title: "Font Icon Pack"
date: 2024-02-01T23:44:12+03:00
draft: false

# Meta of the article
tags: [ "Flutter", "Figma", "Design"]
author: [ "Artur Rafikov" ]
cover:
  image: "/articles/font_icon_pack/cover.png" # Relative path to the cover image
  alt: "Article cover"
  relative: true

# Table of contents settings
showToc: true
TocOpen: true
UseHugoToc: true
---

В этой статье я расскажу про правильное использование иконок, их стандартизацию со стороны дизайна, а также как превращать иконки из Figma в шрифтовой символ.

## **Способы отображения SVG иконок**

В основном их два: прямо использовать SVG файл, либо использовать шрифтовой пак. Но перед тем, как решить, какой способ использовать, надо узнать побольше про SVG. SVG был создан Консорциумом Всемирной паутины, и он использует XML для описания графики. Из этой информации можно сделать вывод: для показа SVG необходим рендер-движок, поддерживающий XML. С этим прекрасно справляются веб браузеры, но если речь заходит про другие движки для рендера, то их поддержка, скорее всего, не представлена ([Flutter](https://flutter.dev/go/vector-graphics)), либо ограничена ([Android Native](https://developer.android.com/studio/write/vector-asset-studio#svg)).

Для решения этой проблемы, придумали иконки превращать в глиф шрифта. Любой рендер-движок точно умеет рисовать шрифты, поэтому и с иконками проблем быть не должно. Такие решения используют [FontAwesome](https://fontawesome.com/icons), [Google Material Symbols](https://fonts.google.com/icons) и многие другие популярные наборы иконок. И, так как IconPack является шрифтом, то ничего не мешает добавлять иконки, например, в [терминал](https://starship.rs/). Единственной проблемой может возникнуть сложность преобразования SVG иконки из Figma в шрифт, но об этом я расскажу подробнее ниже.

## **Как правильно во Flutter?**

Как я упомянул выше, SVG не является нативным для Flutter. Существуют пакеты для рендера SVG файлов ([flutter_svg](https://pub.dev/packages/flutter_svg), [jovial_svg](https://pub.dev/packages/jovial_svg)), но в их работе есть очень много багов отображения изображения (особенно с выходом Impeller), плюс они очень медленные. Также сам формат SVG позволяет творить неописуемые ужасы.

Flutter уже имеет нативный способ использования иконок - [IconData](https://api.flutter.dev/flutter/widgets/IconData-class.html) класс, и material набор иконок [Google Material Symbols](https://fonts.google.com/icons). А приятной плюшкой является tree-shaking шрифта - выброс неиспользуемых символов для экономии памяти.

## **Делаем IconPack**

Перед началом работы, нам понадобится одна из двух программ: Adobe Illustrator или [Inkscape](https://inkscape.org/). Figma не предназначена для работы со шрифтами, и довольно часто экспортирует SVG не так, как нам нужно, поэтому мы будем править за ней самостоятельно. Я буду описывать шаги для Inkscape, но для Illustrator действия будут аналогичными.

## **Работа в Figma**

Очень важно перед началом конвертации стандартизировать иконки. Все ваши иконки должны иметь строгий шаблон, по которому они создаются, иначе у вас будет очень много проблем с подбором корректных размеров иконки на всех этапах (и дизайнеру, и разработчику).

В качестве референсов дизайнеру можно показать [Figma Material Design Icons](https://www.figma.com/community/file/1014241558898418245). Также довольно полезной является статья [Designing Icons](https://m3.material.io/styles/icons/designing-icons) от команды Material Design.

Теперь к стандартизации иконок. Все иконки дизайнер изначально должен рисовать на фиксированном холсте, итоговый размер который выбирает сам. Я рекомендую выбирать холст размером в 24. Теперь необходимо обозначить зону, в которой иконка будет рисоваться. Иконка не должна выходить за пределы этой зоны. Обычно внутренний отступ на несколько значений (2-3), нет необходимости делать его сильно большим, потому что этот отступ нельзя будет изменить разработчику. Вы добавляете "safe zones" к иконке.

Примеры:

![Иконка 24, область для рисования 18, отступ 3](/articles/font_icon_pack/images/icon_24.png)

Иконка 24, область для рисования 18, отступ 3

![Иконка 16, область рисования 12, отступ 2](/articles/font_icon_pack/images/icon_16.png)

Иконка 16, область рисования 12, отступ 2

Остальные тонкости дизайна расписаны в статье выше, я выделил необходимый минимум для работы. Такой формат иконки должен являться базой для экспорта разработчику. Когда дизайнер применяет его в других макетах, и ему необходимо изменить размер иконки, то корректнее это будет делать с помощью инструмента Scale (K), чтобы сохранить все пропорции.

На этом начинается работа по превращению иконки в шрифт. Экспортировать мы должны базовый объект, который имеет установленный формат. В моем случае это формат 24х24. В качестве примера я создам иконку "пауза", состоящей из двух разных объектов и остаточными слоями от базового макета иконок:

![icon_24_example_1.png](/articles/font_icon_pack/images/icon_24_example_1.png)

Экспортируем эту иконку 1x в SVG формате и открываем с помощью Inkscape.

## **Работа в Inkscape**

Вот так выглядит интерфейс программы с открытой иконкой:

![inkscape_workspace.png](/articles/font_icon_pack/images/inkscape_workspace.png)

Необходимо потыкать на эти варианты по порядку:

1. Выделить все объекты (Ctrl + A)
2. Выбрать Object -> Ungroup
3. Выбрать Path -> Stroke to path (превращаем объекты в path)
4. Выбрать Path -> Union (если пропадет иконки - не беспокойтесь)
5. Выбрать Path -> Combine
6. Если пропала иконка, то необходимо выбрать цвет для новой фигуры. Снизу есть селектор цветов, достаточно выбрать черный. Выбранный цвет не важен, он нужен лишь для того, чтобы мы могли увидеть иконку. Конечный цвет можно будет задавать через код.
7. Выбрать File -> Clean Up Document
8. На этом этапе справа в колонке "Layers and Objects" у вас должен остаться один path объект
9. Сохраните файл, обязательно выбрав тип файла "Plain SVG"

На данном этапе мы закончили основную работу, осталось запаковать иконку в шрифт. Для Flutter можно использовать сайт [fluttericon](https://www.fluttericon.com/). Достаточно перенести наш новый SVG файл на сайт, и он будет добавлен в шрифтовой пак. Нажмите на иконку, чтобы добавить её в пак (она должна загореться красным), пак после этого можно скачать. В скачанном архиве будет шрифт формата Open Font Format, dart класс с IconData всех иконок и json файл, с помощью которого можно восстановить все добавленные иконки на сайте fluttericon.

Использование в коде будет выглядеть примерно так:

```dart
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text('IconPack Example'),
      leading: Icon(MyIcons.pause, color: Colors.black),
    ),
  );
}
```

Что мы получили от этого способа:

1. Всегда корректные и одинаковые размеры иконок в дизайне Figma и в коде
2. Нативный рендер иконок
3. Tree-shaking
4. Возможность кросс-платформенного использования (полученный шрифт можно использовать и в вебе)

> *Но мне нужно добавить SVG логотип нашего приложения, что мне делать?*
>

Используйте CustomPaint для этого случая. Есть сайты, которые напрямую конвертируют SVG в код для CustomPaint. Буквально Ctrl + C и Ctrl + V. Этим самым вы ограждаетесь от проблем в рендере библиотек, которые фиксить будут полгода, плюс вам не надо будет решать проблемы с совместимостью зависимостей.